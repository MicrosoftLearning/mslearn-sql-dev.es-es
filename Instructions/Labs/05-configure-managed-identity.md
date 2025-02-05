---
lab:
  title: Configuración de la identidad administrada para Azure SQL Database
  module: Explore Azure SQL Database safety practices for development
---

# Configuración de la identidad administrada para Azure SQL Database

En este ejercicio, agregarás una identidad administrada a la aplicación web de ejemplo sin almacenar credenciales en el código.

Azure App Service ofrece una solución de hospedaje web altamente escalable y de mantenimiento automático. Una de sus características clave es la provisión de una identidad administrada para su aplicación, lo que simplifica la seguridad del acceso a Azure SQL Database y otros servicios de Azure. Al usar identidades administradas, puedes mejorar la seguridad de tu aplicación al eliminar la necesidad de almacenar información confidencial como credenciales en cadenas de conexión. 

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de comenzar

Antes de empezar este ejercicio, necesitas:

- Una suscripción a Azure con los permisos adecuados para crear y administrar recursos.
- [**Visual Studio Code**](https://code.visualstudio.com/download?azure-portal=true) instalado en tu equipo con la siguiente extensión instalada:
    - [Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice?azure-portal=true).

## Creación de una aplicación web y una Azure SQL Database

En primer lugar, crearemos una aplicación web y una Azure SQL Database.

1. Inicie sesión en [Azure Portal](https://portal.azure.com?azure-portal=true).
1. Busca **Suscripciones** y selecciónalo.
1. Ve a **Proveedores de recursos** en **Configuración**, busca el proveedor **Microsoft.Sql** y selecciona **Registrar**.
1. Vuelve a la página principal de Azure Portal, selecciona **Crear un recurso**.
1. Busca **Aplicación web + base de datos** y selecciónala.
1. Selecciona **Crear** y rellena los detalles requeridos:

    | Grupo | Configuración | Valor |
    | --- | --- | --- |
    | **Detalles del proyecto** | **Suscripción** | Seleccione la suscripción a Azure. |
    | **Detalles del proyecto** | **Grupo de recursos** | Seleccione o cree un grupo de recursos nuevo. |
    | **Detalles del proyecto** | **Región** | Selecciona la región donde quieres hospedar tu aplicación web |
    | **Detalles de la aplicación web** | **Nombre** | Escriba un nombre único para la aplicación web |
    | **Detalles de la aplicación web** | **Pila en tiempo de ejecución** | .NET 8 (LTS) |
    | **Base de datos** | **Engine** | SQLAzure |
    | **Base de datos** | **Nombre del servidor** | Escribe un nombre único para tu SQL Server |
    | **Base de datos** | **Nombre de la base de datos** | Escribe un nombre único para tu base de datos |
    | **Hospedar aplicaciones de WPF** | **Plan de hospedaje** | Básico |

    > **Nota:** en el caso de las cargas de trabajo de producción, selecciona **Estándar: aplicaciones de producción de uso general**. El nombre de usuario y la contraseña de la nueva base de datos se generan automáticamente. Para recuperar estos valores después de la implementación, ve a las **Cadenas de conexión** ubicadas en la página **Variables de entorno** de tu aplicación. 

1. Seleccione **Revisar y crear** y, a continuación, **Crear**. La implementación puede tardar unos minutos en completarse.
1. Conéctate a tu base de datos en Azure Data Studio y ejecuta el siguiente código.

    ```sql
    CREATE TABLE Products (
        ProductID INT PRIMARY KEY,
        ProductName NVARCHAR(100),
        Category NVARCHAR(50),
        Price DECIMAL(10, 2),
        Stock INT
    );
    
    INSERT INTO Products (ProductID, ProductName, Category, Price, Stock) VALUES
    (1, 'Laptop', 'Electronics', 999.99, 50),
    (2, 'Smartphone', 'Electronics', 699.99, 150),
    (3, 'Desk Chair', 'Furniture', 89.99, 200),
    (4, 'Coffee Maker', 'Appliances', 49.99, 100),
    (5, 'Book', 'Books', 19.99, 300);
    ```

## Agrega la cuenta como administrador de SQL

Después, agregarás el acceso de tu cuenta a la base de datos. Esto es necesario porque solo las cuentas autenticadas a través de Microsoft Entra pueden crear otros usuarios de Microsoft Entra ID, que son un requisito previo para los pasos posteriores de este ejercicio.

1. Ve al servidor Azure SQL que has creado anteriormente.
1. En el menú de la izquierda **Configuración**, selecciona **Microsoft Entra ID**.
1. Seleccione **Establecer administrador**.
1. Busca tu cuenta y selecciónala.
1. Seleccione **Guardar**.

## Habilitación de una entidad administrada

Después, habilitarás la identidad administrada asignada por el sistema para tu aplicación web de Azure, que es un procedimiento recomendado de seguridad que permite la administración automatizada de credenciales.

1. Vaya a la aplicación web en Azure Portal.
1. En el menú **Configuración** de la izquierda, selecciona **Identidad**.
1. En la pestaña **Sistema asignado**, cambia el **Estado** a **Activado** y selecciona **Guardar**. En caso de que recibas un mensaje preguntándote si deseas habilitar la identidad administrada asignada por el sistema para tu aplicación web, selecciona **Sí**.

## Concesión de acceso a Azure SQL Database

1. Conexión a Azure SQL Database mediante Azure Data Studio. Selecciona **Microsoft Entra ID: universal con compatibilidad MFA** y proporciona tu nombre de usuario.
1. Selecciona la base de datos y abre un nuevo editor de consultas.
1. Ejecuta los siguientes comandos SQL para crear un usuario para la identidad administrada y asignar los permisos necesarios. Edita el script al proporcionar el nombre de la aplicación web.

    ```sql
    CREATE USER [your-web-app-name] FROM EXTERNAL PROVIDER;
    ALTER ROLE db_datareader ADD MEMBER [your-web-app-name];
    ALTER ROLE db_datawriter ADD MEMBER [your-web-app-name];
    ```

## Crear una aplicación web 

Después, crearás una aplicación ASP.NET que use Entity Framework Core con Azure SQL Database para mostrar una lista de productos de la tabla de productos.

### Creación del proyecto

1. En VS Code, crea una carpeta. Asigna un nombre a la carpeta del proyecto.
1. Abre el terminal y ejecuta el siguiente comando para crear tu nuevo proyecto de MVC.
    
    ```dos
        dotnet new mvc
    ```
    Esto creará un nuevo proyecto ASP.NET MVC en la carpeta que has elegido y lo cargará en Visual Studio Code.

1. Ejecuta el siguiente comando para ejecutar tu aplicación. 

    ```dos
    dotnet run
    ```
1. El terminal genera *Escuchando: http://localhost:<port>*. Ve a la dirección URL en tu explorador para acceder a la aplicación. 

1. Cierra el explorador web y detén la aplicación. Como alternativa, puedes detener la aplicación al presionar `Ctrl+C` en el terminal de VS Code.

### Actualización del proyecto para conectar a Azure SQL Database

Después, actualizarás algunas configuraciones que te permitirán conectarte correctamente a Azure SQL Database con una identidad administrada.

1. En el proyecto, agrega los paquetes NuGet necesarios para SQL Server.
    ```dos
    dotnet add package Microsoft.EntityFrameworkCore.SqlServer
    ```
1. En la carpeta raíz de tu proyecto, abre el archivo **appsettings.json** e inserta la sección `ConnectionStrings`. Aquí, reemplazarás `<server-name>` y `<db-name>` por los nombres reales del servidor y la base de datos. El constructor predeterminado del archivo `Models/MyDbContext.cs` usa esta cadena de conexión para establecer una conexión con tu base de datos.

    ```json
    {
      "Logging": {
        "LogLevel": {
          "Default": "Information",
          "Microsoft.AspNetCore": "Warning"
        }
      },
      "AllowedHosts": "*",
      "ConnectionStrings": {
        "DefaultConnection": "Server=<server-name>.database.windows.net,1433;Initial Catalog=<db-name>;Authentication=Active Directory Default;"
      }
    }
    ```
1. Guarde y cierre el archivo.

### Adición del código

1. En la carpeta **Modelos** de tu proyecto, crea un archivo **Product.cs** para tu entidad de producto con el siguiente código. Reemplaza `<app name>` por el nombre real de tu aplicación.

    ```csharp
    namespace <app name>.Models;
    
    public class Product
    {
        public int ProductId { get; set; }
        public string ProductName { get; set; }
        public string Category { get; set; }
        public decimal Price { get; set; }
        public int Stock { get; set; }
    }
    ```
1. Crea la carpeta **Base de datos** en la carpeta raíz de tu proyecto.
1. En la carpeta **Base de datos** de tu proyecto, crea un archivo **MyDbContext.cs** para tu entidad de producto con el siguiente código. Reemplaza `<app name>` por el nombre real de tu aplicación.

    ```csharp
    using <app name>.Models;
    
    namespace <app name>.Database;
    
    using Microsoft.EntityFrameworkCore;
    
    public class MyDbContext : DbContext
    {
        public MyDbContext(DbContextOptions<MyDbContext> options) : base(options)
        {
        }
    
        public DbSet<Product> Products { get; set; }
    }    
    ```
1. En la carpeta **Controladores** de tu proyecto, edita las clases `HomeController` y `IActionResult` del archivo **HomeController.cs**, y agrega la variable `_context` con el siguiente código.

    ```csharp
    private MyDbContext _context;

    public HomeController(ILogger<HomeController> logger, MyDbContext context)
    {
        _logger = logger;
        _context = context;
    }

    public IActionResult Index()
    {
        var data = _context.Products.ToList();
        return View(data);
    }
    ```
1. En la carpeta **Vistas -> Inicio** de tu proyecto, actualiza el archivo **Index.cshtml**, y agrega el siguiente código.

    ```html
    <table class="table">
        <thead>
            <tr>
                <th>Product Id</th>
                <th>Product Name</th>
                <th>Category</th>
                <th>Price</th>
                <th>Stock</th>
            </tr>
        </thead>
        <tbody>
            @foreach(var item in Model)
            {
                <tr>
                    <td>@item.ProductId</td>
                    <td>@item.ProductName</td>
                    <td>@item.Category</td>
                    <td>@item.Price</td>
                    <td>@item.Stock</td>
                </tr>
            }
        </tbody>
    </table>
    ```

1. Edita el archivo **Program.cs** e inserta el fragmento de código proporcionado justo encima de la línea `var app = builder.Build();`. Este cambio garantiza que el código se ejecute durante la secuencia de inicio de la aplicación. Reemplaza `<app name>` por el nombre real de tu aplicación.

    ```csharp
    using Microsoft.EntityFrameworkCore;
    using <app name>.Database;

    var builder = WebApplication.CreateBuilder(args);

    // Add services to the container.
    builder.Services.AddControllersWithViews();
    builder.Services.AddDbContext<MyDbContext>(options =>
        options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
    ```

    > **Nota:** si deseas ejecutar la aplicación antes de la implementación, actualiza la cadena de conexión con las credenciales de usuario de SQL. El nombre de usuario y la contraseña de la nueva base de datos se generan automáticamente. Para recuperar estos valores después de la implementación, ve a las **Cadenas de conexión** ubicadas en la página **Variables de entorno** de tu aplicación. Una vez que hayas confirmado que la aplicación se está ejecutando según lo previsto, vuelve a usar la identidad administrada para obtener un proceso de implementación seguro.

### Implementación del código

1. Presiona `Ctrl+Shift+P` para abrir la **paleta de comandos**.
1. Escribe **Azure App Service: implementar en aplicación web...** y selecciónalo.
1. Selecciona la carpeta que contiene el código de tu aplicación web.
1. Elige la aplicación web que has creado en el paso anterior.
    > Nota: puedes recibir el mensaje: "Falta la configuración necesaria para implementarla en la aplicación". Selecciona **Agregar configuración**. Después, sigue las instrucciones y selecciona tu suscripción y el recurso de App Service.
1. Confirma la implementación cuando se te solicite.

## Prueba de la aplicación

Ejecuta tu aplicación web y comprueba que puede conectarse a Azure SQL Database sin ninguna credencial almacenada.

1. Abre un explorador y ve a la dirección URL de la aplicación web de Azure (por ejemplo, https://your-web-app-name.azurewebsites.net).
1. Comprueba que la aplicación web se está ejecutando y es accesible.
1. Deberías ver una página web como la siguiente.

    ![Una captura de pantalla de la aplicación web después de implementarla.](./Media/01-app-page.png)

## Configurar la implementación continua (opcional)

1. Presiona `Ctrl+Shift+P` para abrir la **paleta de comandos**.
1. Escribe **App de Azure Servicio: configurar entrega continua...** y selecciónalo.
1. Sigue las indicaciones para configurar la implementación continua desde el repositorio de GitHub o Azure DevOps.

Ten en cuenta los escenarios en los que sería beneficioso usar **Identidad administrada asignada por el usuario** en lugar de **Identidad administrada asignada por el sistema**.

## Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. 

Dejar que los recursos se ejecuten innecesariamente puede dar lugar a costes adicionales. Puede eliminar recursos de forma individual o bien eliminar el grupo de recursos completo desde [Azure Portal](https://portal.azure.com?azure-portal=true).

## Más información

Para obtener más información sobre los grupos de conmutación por error automática para Azure SQL Database, consulta [Identidades administradas en Microsoft Entra para Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).
