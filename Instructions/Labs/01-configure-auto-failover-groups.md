---
lab:
  title: Habilita la resistencia de las aplicaciones con grupos de conmutación por error automática para Azure SQL Database
  module: Get started with Azure SQL Database for cloud-native application development
---

# Habilita la resistencia de las aplicaciones con grupos de conmutación por error automática para Azure SQL Database

En este ejercicio, creará dos bases de datos de Azure SQL que actuarán como principales y secundarias. Configurarás grupos de conmutación por error automática para garantizar la alta disponibilidad y la recuperación ante desastres de las bases de datos de tu aplicación, y validarás el estado de replicación en tu aplicación.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de comenzar

Antes de empezar este ejercicio, necesitas:

- Una suscripción a Azure con los permisos adecuados para crear y administrar recursos.
- [**Visual Studio Code**](https://code.visualstudio.com/download?azure-portal=true) instalado en tu equipo con la siguiente extensión instalada:
    - [Kit de desarrollo de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit?azure-portal=true).

## Creación de los servidores principal y secundario de Azure SQL

En primer lugar, configuraremos los servidores principal y secundario, y usaremos la base de datos de ejemplo **AdventureWorksLT**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com?azure-portal=true).

1. Selecciona el icono Cloud Shell en la esquina superior derecha de Azure Portal. Parece un símbolo `>_`. Si se te solicita, elige **Bash** como tipo de shell.

1. Ejecuta los siguientes comandos en el terminal de Cloud Shell. Reemplaza los valores `<your_resource_group>`, `<your_primary_server>`, `<your_location>`, `<your_secondary_server>` y `<your_admin_password>` por tus valores reales:

    * Crear un grupo de recursos
    ```azurecli
    az group create --name <your_resource_group> --location <your_location>
    ```

    * Creación del servidor SQL principal
    ```azurecli        
    az sql server create --name <your_primary_server> --resource-group <your_resource_group> --location <your_location> --admin-user "sqladmin" --admin-password <your_admin_password>
    ```

    * Creación del servidor SQL secundario. El mismo script solo cambia el nombre y la ubicación del servidor.
    ```azurecli
    az sql server create --name <your_secondary_server> --resource-group <your_resource_group> --location <your_location> --admin-user "sqladmin" --admin-password <your_admin_password>    
    ```

    * Creación de una base de datos de ejemplo en el servidor principal con el plan de tarifa especificado
    ```azurecli
    az sql db create --resource-group <your_resource_group> --server <your_primary_server> --name AdventureWorksLT --sample-name AdventureWorksLT --service-objective "S0"    
    ```
    
1. Una vez completadas las implementaciones, ve al servidor principal de Azure SQL Server que has creado.
1. Selecciona **Redes** en **Seguridad** en el panel izquierdo. Agrega tu dirección IP a las reglas de firewall.
1. Selecciona la opción **Permitir que los servicios y recursos de Azure accedan a este servidor**.
1. Seleccione **Guardar**.
1. Repite los pasos anteriores para el servidor secundario.

    Estos pasos garantizan que tengas un entorno estructurado y redundante de Azure SQL Database listo para su uso.

## Configuración de los grupos de conmutación por error automática

Después, crearás un grupo de conmutación por error automática para la Azure SQL Database que has configurado anteriormente. Esto implica establecer un grupo de conmutación por error entre dos servidores y comprobar la configuración para asegurarse de que funciona correctamente.

1. Ejecuta los siguientes comandos en el terminal de Cloud Shell. Reemplaza los valores `<your_failover_group>`, `<your_resource_group>`, `<your_primary_server>` y `<your_secondary_server>` por tus valores reales:

    * Creación del grupo de conmutación por error
    ```azurecli
    az sql failover-group create -n <your_failover_group> -g <your_resource_group> -s <your_primary_server> --partner-server <your_secondary_server> --failover-policy Automatic --grace-period 1 --add-db AdventureWorksLT
    ```

    * Comprueba el grupo de conmutación por error
    ```azurecli    
    az sql failover-group show -n <your_failover_group> -g <your_resource_group> -s <your_primary_server>
    ```

    > Dedica un momento a revisar los resultados y los valores de `partnerServers`. ¿Por qué esto es importante?

    > Al comprobar el atributo `role` dentro de cada servidor asociado, puedes determinar si un servidor está actuando en ese momento como principal o secundario. Esta información es fundamental para comprender la configuración actual y la preparación del grupo de conmutación por error. Te ayuda a evaluar el posible impacto en la aplicación durante los escenarios de conmutación por error y garantiza que la configuración esté configurada correctamente para la alta disponibilidad y la recuperación ante desastres.
    
## Integración con el código de la aplicación

Para conectar la aplicación .NET al punto de conexión de Azure SQL Database, deberás seguir estos pasos.

1. En Visual Studio Code, abre el terminal y ejecuta los siguientes comandos para instalar el paquete `Microsoft.Data.SqlClient` y crear una nueva aplicación de consola de .NET.

    ```bash
    dotnet new console -n AdventureWorksLTApp
    cd AdventureWorksLTApp 
    dotnet add package Microsoft.Data.SqlClient --version 5.2.1
    ```

1. Abre la carpeta `AdventureWorksLTApp`que se ha creado con el paso anterior en **Visual Studio Code**.

1. Creación de un archivo `appsettings.json` en la raíz del directorio del proyecto. Este archivo de configuración almacenará la cadena de conexión a la base de datos. Asegúrate de reemplazar los valores `<your_failover_group>` y `<your_password>` de la cadena de conexión por tus detalles reales.

    ```json
    {
      "ConnectionStrings": {
        "FailoverGroupConnection": "Server=tcp:<your_failover_group>.database.windows.net,1433;Initial Catalog=AdventureWorksLT;Persist Security Info=False;User ID=sqladmin;Password=<your_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
      }
    }
    ```

1. Abre el archivo `.csproj` en **Código Visual Studio** y agrega el siguiente contenido justo debajo de la etiqueta `</PropertyGroup>`.

    ```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.Data.SqlClient" Version="5.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration" Version="6.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="6.0.0" />
    </ItemGroup>
    ```

    Tu archivo `.csproj` completo debería tener un aspecto similar al siguiente.

    ```xml
    <Project Sdk="Microsoft.NET.Sdk">

      <PropertyGroup>
        <OutputType>Exe</OutputType>
        <TargetFramework>net8.0</TargetFramework>
        <ImplicitUsings>enable</ImplicitUsings>
        <Nullable>enable</Nullable>
      </PropertyGroup>
    
      <ItemGroup>
        <PackageReference Include="Microsoft.Data.SqlClient" Version="5.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration" Version="6.0.0" />
        <PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="6.0.0" />
      </ItemGroup>
    
    </Project>
    ```

1. Abre el archivo `Program.cs`en **Visual Studio Code**. En el editor, reemplaza todo el código existente por el código proporcionado a continuación.

    > **Nota:** dedica un momento a revisar el código y observa cómo imprime información sobre los servidores principales y secundarios en el grupo de conmutación por error automática.

    ```csharp
    using System;
    using Microsoft.Data.SqlClient;
    using Microsoft.Extensions.Configuration;
    using System.IO;
    
    namespace AdventureWorksLTApp
    {
        class Program
        {
            static void Main(string[] args)
            {
                var configuration = new ConfigurationBuilder()
                    .SetBasePath(Directory.GetCurrentDirectory())
                    .AddJsonFile("appsettings.json")
                    .Build();
    
                string connectionString = configuration.GetConnectionString("FailoverGroupConnection");
    
                ExecuteQuery(connectionString);
            }
    
            static void ExecuteQuery(string connectionString)
            {
                using (SqlConnection connection = new SqlConnection(connectionString))
                {
                    try
                    {
                        connection.Open();
                        string query = @"
                            SELECT 
                                @@SERVERNAME AS [Primary_Server],
                                partner_server AS [Secondary_Server],
                                partner_database AS [Database],
                                replication_state_desc
                            FROM 
                                sys.dm_geo_replication_link_status";
    
                        using (SqlCommand command = new SqlCommand(query, connection))
                        {
                            using (SqlDataReader reader = command.ExecuteReader())
                            {
                                while (reader.Read())
                                {
                                    Console.WriteLine($"Primary Server: {reader["Primary_Server"]}");
                                    Console.WriteLine($"Secondary Server: {reader["Secondary_Server"]}");
                                    Console.WriteLine($"Database: {reader["Database"]}");
                                    Console.WriteLine($"Replication State: {reader["replication_state_desc"]}");
                                }
                            }
                        }
                    }
                    catch (Exception ex)
                    {
                        Console.WriteLine($"Error executing query: {ex.Message}");
                    }
                    finally
                    {
                        connection.Close();
                    }
                }
            }
        }
    }
    ```

1. Ejecuta el código al seleccionar **Ejecutar** > **Iniciar depuración** en el menú, o simplemente presiona **F5**. También puedes seleccionar el botón Reproducir de la barra de herramientas superior para iniciar la aplicación.

    > **Importante:** si recibes el mensaje *"No tiene una extensión para depurar C#. ¿Deberíamos encontrar una extensión de C# en Marketplace?"*, asegúrate de que la extensión **Kit de desarrollo de C#** está instalada.

1. Después de ejecutar el código, deberías ver la salida en la pestaña **Consola de depuración** de Visual Studio Code.

    ```
    Primary Server: <your_server_name>
    Secondary Server: <your_server_name>
    Database: AdventureWorksLT
    Replication State: CATCH_UP
    ```
    
    El estado de replicación `CATCH_UP` significa que la base de datos está totalmente sincronizada con su pareja y está lista para la conmutación por error. La supervisión del estado de replicación puede ayudar a identificar cuellos de botella de rendimiento y asegurarse de que la replicación de datos está ocurriendo de forma eficaz.

## Realice una conmutación por error a una región secundaria

Imagine un escenario en el que la base de datos principal de Azure SQL Database está experimentando problemas debido a una interrupción regional. Para mantener la continuidad del servicio y minimizar el tiempo de inactividad, debes cambiar la aplicación a la réplica secundaria mediante la realización de una conmutación por error forzada.

Durante una conmutación por error forzada, todas las nuevas sesiones de TDS se vuelven a enrutar automáticamente al servidor secundario, que luego se convierte en el servidor principal. Lo mejor es que no es necesario cambiar la cadena de conexión de la aplicación, ya que el punto de conexión sigue siendo el mismo.

Vamos a iniciar una conmutación por error y ejecutar nuestra aplicación para comprobar el estado de nuestros servidores principales y secundarios.

1. Vuelve a Azure Portal, abre una nueva instancia del terminal de Cloud Shell. Ejecute el código siguiente: Reemplaza los valores `<your_failover_group>`, `<your_resource_group>` y `<your_primary_server>` por tus valores reales. El valor del parámetro `--server` debe ser el secundario actual.

    ```azurecli
    az sql failover-group set-primary --name <your_failover_group> --resource-group <your_resource_group> --server <your_server_name>
    ```

    > **Nota**: Esta operación puede tardar unos minutos.

1. Una vez completada la conmutación por error, vuelve a ejecutar la aplicación para comprobar el estado de replicación. Deberías ver que el servidor secundario ha tomado el control como principal y que el servidor principal original se ha convertido en el secundario.

Ten en cuenta por qué querrías colocar tus bases de datos de aplicaciones primaria y secundaria en la misma región, y cuándo podría resultar beneficioso elegir regiones diferentes.

## Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. 

Dejar que los recursos se ejecuten innecesariamente puede dar lugar a costes adicionales. Puede eliminar recursos de forma individual o bien eliminar el grupo de recursos completo desde [Azure Portal](https://portal.azure.com?azure-portal=true).

## Más información

Para obtener más información sobre los grupos de conmutación por error automática para Azure SQL Database, consulta [Información general y procedimientos recomendados sobre los grupos de conmutación por error (Azure SQL Database)](https://learn.microsoft.com/azure/azure-sql/database/failover-group-sql-db?azure-portal=true).
