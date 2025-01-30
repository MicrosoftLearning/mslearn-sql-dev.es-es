---
lab:
  title: Desarrollo de una API de datos para Azure SQL Database
  module: Develop a Data API for Azure SQL Database
---

# Desarrollo de una API de datos para Azure SQL Database

En este ejercicio, vas a desarrollar e implementar una API de datos para una Azure SQL Database con Azure Static Web Apps. Esto proporcionará experiencia práctica en la configuración del generador de API de datos y su implementación en un entorno de Azure Static Web App.

## Requisitos previos

Antes de empezar este ejercicio, asegúrate de que tienes lo siguiente:

- Una suscripción a Azure activa.
- Conocimientos básicos de Azure SQL Database, Azure Static Web Apps y GitHub.
- Visual Studio Code instalado con las extensiones necesarias.
- Una cuenta de GitHub para administrar el repositorio.

## Configuración del entorno

Hay algunos pasos que debes seguir para configurar el entorno de este ejercicio.

### Instalación de extensiones de Visual Studio Code

Antes de comenzar este ejercicio, debes instalar las extensiones de Visual Studio Code.

1. Abra Visual Studio Code.
1. Abre una ventana de terminal en Visual Studio Code.
1. Instala la CLI de Static Web Apps mediante la ejecución del siguiente comando:

    ```bash
    npm install -g @azure/static-web-apps-cli
    ```

1. Instala la CLI del generador de API de datos mediante la ejecución del siguiente comando:

    ```bash
    dotnet tool install --global Microsoft.DataApiBuilder
    ```

Visual Studio Code ahora está configurado con las extensiones necesarias.

### Crear una instancia de Azure SQL Database

Si aún no lo has hecho, debes crear una instancia de Azure SQL Database.

1. Inicie sesión en [Azure Portal](https://portal.azure.com?azure-portal=true). 
1. Vaya a la página **Azure SQL** y seleccione **+ Crear**.
1. Seleccione **SQL Database**, *Base de datos* única y el botón **Crear**.
1. Rellena la información necesaria en el cuadro de diálogo **Crear SQL Database** y selecciona **OK** (deja el resto de opciones con sus valores predeterminados).

    | Configuración | Valor |
    | --- | --- |
    | Oferta gratuita sin servidor | *Aplicación de la oferta* |
    | Subscription | Su suscripción |
    | Resource group | *Selecciona o crea un nuevo grupo de recursos* |
    | Nombre de la base de datos | *MyDB* |
    | Server | *Selecciona el vínculo **Crear nuevo*** |
    | Nombre del servidor | *Elija un nombre único* |
    | Location | *Seleccionar una ubicación* |
    | Método de autenticación | *Use la autenticación de SQL.* |
    | Inicio de sesión del administrador del servidor | *sqladmin* |
    | Contraseña | *Escriba una contraseña.* |
    | Confirmar contraseña | *Confirma la contraseña* |

1. Seleccione **Revisar y crear** y, después, **Crear**.
1. Una vez completada la implementación, ve al *servidor* de Azure SQL Database que has creado.
1. Selecciona **Redes** en **Seguridad** en el panel izquierdo. Agrega tu dirección IP a las reglas de firewall.
1. Seleccione **Guardar**.

### Adición de datos de ejemplo a la Database

Ahora que tienes una instancia de Azure SQL Database, debes agregar algunos datos de ejemplo. Esto te ayudará a probar la API una vez que esté en funcionamiento.

1. Ve a tu Azure SQL Database recién creada.
1. Usa el **Editor de consultas** de Azure Portal para ejecutar el siguiente script SQL:

    ```sql
    CREATE TABLE [dbo].[Employees]
    (
        EmployeeID INT PRIMARY KEY,
        FirstName NVARCHAR(50),
        LastName NVARCHAR(50),
        Department NVARCHAR(50)
    );
    
    INSERT INTO [dbo].[Employees] VALUES (1,'John', 'Smith', 'Accounting');
    INSERT INTO [dbo].[Employees] VALUES (2,'Michelle', 'Sanchez', 'IT');
    
    SELECT * FROM [dbo].[Employees];
    ```

### Creación de una aplicación web básica en GitHub

Antes de poder crear una aplicación web estática de Azure, tenemos que crear una aplicación web básica en GitHub.

1. Para crear una aplicación web básica en GitHub, ve a [Generar un sitio web de Vanilla](https://github.com/staticwebdev/vanilla-basic/generate).
1. Asegúrate de que la plantilla Repositorio está establecida en **staticwebdev/vanilla-basic**.
1. En ***Propietario***, selecciona tu cuenta.
1. En ***Nombre del repositorio***, escribe el nombre **my-sql-repo**.
1. Haz que el repositorio sea **Privado**.
1. Selecciona el botón **Crear repositorio**.

## Creación de un recurso de aplicación web estática de Azure

Primero crearemos nuestra aplicación web estática y luego le agregaremos la configuración de generador de API de datos.

1. En Azure Portal, ve a la página **Aplicaciones web estáticas**.
1. Selecciona **+ Crear**.
1. Rellena la siguiente información en el cuadro de diálogo **Crear aplicación web estática** (deja el resto de opciones con sus valores predeterminados):

    | Configuración | Valor |
    | --- | --- |
    | Suscripción | Su suscripción |
    | Resource group | *Selecciona o crea un nuevo grupo de recursos* |
    | Nombre | *un nombre único* |
    | Origen del plan de hospedaje | *GitHub* |
    | Cuenta de GitHub | *Selecciona tu cuenta* |
    | Organización | *Lo más probable es que sea tu nombre de usuario de GitHub* |
    | Repositorio | *Selecciona el repositorio que has creado en el paso anterior* |
    | Sucursal | *principal* |

1. Seleccione **Revisar y crear** y, a continuación, **Crear**.
1. Una vez implementado, ve al recurso.
1. Selecciona el botón **Ver aplicación en el explorador**. Deberías ver una página web sencilla con un mensaje de bienvenida. Puedes cerrar esta pestaña.

## Adicción del archivo de configuración del generador de API de datos

Es el momento de agregar la configuración del generador de API de datos a la aplicación web estática de Azure. Es necesario crear un nuevo archivo en el repositorio de GitHub para agregar la configuración de generador de API de datos.

1. En Visual Studio Code, clona el repositorio de GitHub que has creado antes.
1. Abra una ventana de terminal en Visual Studio Code.
1. Ejecuta el siguiente comando para crear un nuevo archivo de configuración del generador de API de datos:

    ```bash
    swa db init --database-type "mssql"
    ```

    Esto creará una nueva carpeta denominada *swa-db-connections* y un archivo denominado *staticwebapp.database.config.json* dentro de esa carpeta.

1. Ejecuta el siguiente comando para agregar las entidades de la base de datos al archivo de configuración:

    ```bash
    dab add "Employees" --source "dbo.Employees" --permissions "anonymous:*" --config "swa-db-connections/staticwebapp.database.config.json"
    ```

1. Revisa el contenido del archivo *staticwebapp.database.config.json*. 
1. Confirma e inserta los cambios en el repositorio de GitHub.

## Configuración de la conexión de base de datos

1. En Azure Portal, ve a la aplicación web estática de Azure que has creado.
1. En **Configuración**, selecciona **Conexión a base de datos**.
1. Selecciona **Vincular base de datos existente**.
1. En el cuadro de diálogo **Vincular base de datos*, selecciona la Azure SQL Database que has creado anteriormente con la siguiente configuración adicional.

    | Configuración | Valor |
    | --- | --- |
    | Tipo de base de datos | *Azure SQL Database* |
    | Tipo de autenticación | *Cadena de conexión* |
    | Nombre de usuario | *tu nombre de usuario administrador* |
    | Contraseña | *la contraseña que le diste al usuario administrador* |
    | Casilla de confirmación | *Activada* |

   > **Nota:** en un entorno de producción, restringe el acceso solo a las direcciones IP necesarias. Además, considera la posibilidad de usar identidades administradas para que la aplicación web estática acceda a la base de datos en lugar de la autenticación de SQL. Para obtener más información, vea las [Identidades administradas en Microsoft Entra para Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).
1. Seleccione **Vínculo**.

## Prueba del punto de conexión de la API de datos

Ahora solo es necesario probar el punto de conexión de la API de datos.

1. En Azure Portal, ve a la aplicación web estática de Azure que has creado.
1. En la página Información general, copia la dirección URL de la aplicación web.
1. Abra una nueva pestaña del explorador y pegue la dirección URL. Deberías seguir viendo la página web sencilla con el mensaje **Aplicación JavaScript de Vanilla**.
1. Agrega **/data-api** al final de la dirección URL y presiona **Entrar**. Debería mostrar **Correcto** para indicar que la API de datos está funcionando.
1. Agrega **/data-api/rest/Empleados** al final de la dirección URL y presiona **Entrar**. Debería ver los datos de ejemplo que has agregado anteriormente a Azure SQL Database.

Has desarrollado e implementado correctamente una API de datos para una Azure SQL Database con aplicaciones web estáticas de Azure.

## Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. 

Dejar que los recursos se ejecuten innecesariamente puede dar lugar a costes adicionales. Puede eliminar recursos de forma individual o bien eliminar el grupo de recursos completo desde [Azure Portal](https://portal.azure.com?azure-portal=true).

## Más información

Para obtener más información sobre el generador de API de datos para bases de datos de Azure SQL, consulta [¿Qué es el generador de API de datos para bases de datos de Azure?](https://learn.microsoft.com/azure/data-api-builder/overview?azure-portal=true).
