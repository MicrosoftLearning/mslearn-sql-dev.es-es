---
lab:
  title: Importación y exportación de datos para el desarrollo en Azure SQL Database
  module: Import and export data for development in Azure SQL Database
---

# Importación y exportación de datos para el desarrollo en Azure SQL Database

En este ejercicio, importarás datos desde un punto de conexión de REST externo (simulado mediante Azure Static Web App) y exportarás datos mediante una función de Azure. El laboratorio proporcionará experiencia práctica en el trabajo con Azure SQL Database con fines de desarrollo, al centrarse en la integración de las API REST y Azure Functions para controlar las operaciones de importación y exportación de datos.

## Requisitos previos

Antes de empezar este laboratorio, asegúrate de que tienes lo siguiente:

- Una suscripción de Azure activa con permiso para crear y administrar recursos.
- Conocimientos básicos de Azure SQL Database, las API de REST y Azure Functions.
- Visual Studio Code instalado con las siguientes extensiones:
      - La extensión de Azure Functions.
- Git instalado para clonar el repositorio.
- SQL Server Management Studio (SSMS) o Azure Data Studio para administrar la base de datos.

## Configuración del entorno

Comencemos configurando los recursos necesarios para este laboratorio, incluyendo una instancia de Azure SQL Database y las herramientas necesarias para importar y exportar datos.

### Crear una instancia de Azure SQL Database

Este paso requiere la creación de una base de datos en Azure:

1. En Azure Portal, ve a la página **Bases de datos SQL**.
1. Seleccione **Crear**.
1. Rellene los campos obligatorios:

    | Configuración | Valor |
    |---|---|
    | Oferta gratuita sin servidor | Aplicación de la oferta |
    | Subscription | Su suscripción |
    | Resource group | Seleccione o cree un grupo de recursos nuevo. |
    | Nombre de la base de datos | **MyDB** |
    | Server | Selecciona o crea un nuevo servidor |
    | Método de autenticación | Autenticación de SQL |
    | Inicio de sesión de administrador de servidor | **sqladmin** |
    | Contraseña | Escribe una contraseña segura |
    | Confirmar contraseña | Confirmación de la contraseña |

1. Seleccione **Revisar y crear** y, a continuación, **Crear**.
1. Una vez finalizada la implementación, ve a la sección **Redes** de tu ***Azure SQL Server*** (no a la Azure SQL Database) y:
    1. Agrega tu dirección IP a las reglas de firewall. Esto te permitirá usar SQL Server Management Studio (SSMS) o Azure Data Studio para administrar la base de datos.
    1. Active la casilla **Permitir que los servicios y recursos de Azure accedan a este servidor**. Esto permitirá que la aplicación de Azure Function acceda al servidor de bases de datos.
    1. Guarda los cambios.
1. Ve a la sección **Microsoft Entra ID** de tu **Azure SQL Server** y asegúrate de *anular la selección* de **Admitir solo la autenticación de Microsoft Entra para este servidor** y **Guardar** los cambios si se han seleccionado. En este ejemplo se usa la autenticación de SQL, por lo que es necesario deshabilitar la compatibilidad solo con Entra.

> [!NOTE]
> En un entorno de producción, deberás determinar a qué tipo de acceso quieres y desde dónde deseas conceder acceso. Aunque la función tenga un ligero cambio si eliges solo la autenticación Entra, ten en cuenta que aún necesitarás habilitar *Permitir que los servicios y recursos de Azure accedan a este servidor* para permitir que la aplicación de Azure Function acceda al servidor.

### Clonación del repositorio de GitHub

1. Abra **Visual Studio Code**.

1. Clona el repositorio de GitHub y prepara tu proyecto:

    1. En **Visual Studio Code**, abre la **paleta de comandos** al presionar **Ctrl+Mayús+P** (Windows) o **Cmd+Mayús+P** (Mac).
    1. Escribe **Git: clonar** y selecciona **Git: clonar**.
    1. En el símbolo del sistema, escribe la siguiente dirección URL para clonar el repositorio:
        ```bash
        https://github.com/MicrosoftLearning/mslearn-sql-dev.git
        ```

    1. Elige la carpeta de destino en la que deseas clonar el repositorio.

### Configuración de Azure Blob Storage para datos JSON

Ahora configuraremos **Azure Blob Storage** para hospedar el archivo **employees.json**. Sige estos pasos en Azure Portal y **Visual Studio Code**.

Para comenzar, crearemos una cuenta de Azure Storage.

1. En **Azure Portal**, ve a la página **Cuentas de almacenamiento**.
1. Seleccione **Crear**.
1. Rellene los campos obligatorios:

    | Configuración | Valor |
    |---|---|
    | Suscripción | Su suscripción |
    | Resource group | Seleccione o cree un grupo de recursos nuevo. |
    | Nombre de la cuenta de almacenamiento | Elige un nombre único global |
    | Region | Elija la región más cercana a la suya. |
    | Servicio principal | **Azure Blob Storage o Azure Data Lake Storage Gen 2** |
    | Rendimiento | Estándar |
    | Redundancia | Almacenamiento con redundancia local (LRS) |

1. Seleccione **Revisar y crear** y, a continuación, **Crear**.
1. Espere a que se cree la cuenta de almacenamiento.

Ahora que tenemos una cuenta, vamos a cargar **employees.json** en Blob Storage.

1. Ve a la página **Cuentas de almacenamiento** en Azure Portal.
1. Seleccione la cuenta de almacenamiento.
1. Ve a la sección **Contenedores**.
1. Crea un nuevo contenedor denominado **jsonfiles**.
1. Dentro del contenedor, haz clic en **Cargar** y carga el archivo **employees.json** ubicado en **/Allfiles/Labs/04/blob-storage** en el directorio clonado.

Aunque podríamos permitir el acceso anónimo al archivo, en nuestro caso, vamos a generar una *firma de acceso compartido (SAS)* para este archivo para garantizar el acceso seguro.

1. En el contenedor **jsonfiles**, selecciona el archivo **employees.json**.
1. Selecciona **Generar SAS** en el menú contextual del archivo.
1. Revisa la configuración y selecciona **Generar SAS y dirección URL**.
1. Se generará un token de SAS del blob y una dirección URL de SAS del blob. Copia el **token de SAS de blob** y la **dirección URL de SAS de blob** para usarlos en los siguientes pasos. No podrás acceder de nuevo al valor del token una vez que cierres esa ventana.

Ahora deberíamos tener una dirección URL segura para acceder al archivo **employees.json**, vamos a probarlo.

1. Abre una nueva pestaña del explorador y pega la **dirección URL de SAS de blob**.
1. Deberías ver el contenido del archivo **employees.json** mostrado en el explorador, que debería tener este aspecto:

    ```json
    {
        "employees": [
            {
                "EmployeeID": 1,
                "FirstName": "John",
                "LastName": "Doe",
                "Department": "HR"
            },
            {
                "EmployeeID": 2,
                "FirstName": "Jane",
                "LastName": "Smith",
                "Department": "Engineering"
            }
        ]
    }
    ```

### Importación de datos de Blob Storage a Azure SQL Database

Ya estamos listos para importar los datos del archivo **employees.json** hospedado en Azure Blob Storage a nuestra Azure SQL Database.

Es necesario empezar creando una **clave maestra** y una **credencial con ámbito de base de datos** en Azure SQL Database.

1. Conéctate a tu base Azure SQL Database con **SQL Server Management Studio** (SSMS) o **Azure Data Studio**.
1. Ejecuta el siguiente comando SQL para crear una clave maestra *si aún no tienes una*:

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'YourStrongPassword!';
    ```

1. Después, crea una **credencial de ámbito de base de datos** para acceder al Azure Blob Storage mediante la ejecución del siguiente comando SQL:

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL MyBlobCredential
    WITH IDENTITY = 'SHARED ACCESS SIGNATURE', 
    SECRET = '<your-sas-token>';
    ```

    Reemplaza ***<your-sas-token>*** con el **token de SAS de blob** generado anteriormente.

1. Por último, necesitas un **origen de datos** para acceder al Azure Blob Storage. Ejecuta el siguiente comando SQL para crear un **origen de datos**:

    ```sql
    CREATE EXTERNAL DATA SOURCE MyBlobDataSource
    WITH (
        TYPE = BLOB_STORAGE,
        LOCATION = 'https://<your-storage-account-name>.blob.core.windows.net',
        CREDENTIAL = MyBlobCredential
    );
    ```

    Reemplaza ***<your-storage-account-name>*** por el nombre de tu cuenta de Azure Storage.

Ya está todo listo para importar los datos del archivo **employees.json** a la *Azure SQL Database*.

Usa el siguiente comando SQL para importar datos del archivo **employees.json** hospedado en *Azure Blob Storage*:

```sql
SELECT EmployeeID
    , FirstName
    , LastName
    , Department
INTO dbo.employee_data
FROM OPENROWSET(
    BULK 'jsonfiles/employees.json',
    DATA_SOURCE = 'MyBlobDataSource',
    SINGLE_CLOB
) AS JSONData
CROSS APPLY OPENJSON(JSONData.BulkColumn, '$.employees')
WITH (
    EmployeeID INT '$.EmployeeID',
    FirstName NVARCHAR(50) '$.FirstName',
    LastName NVARCHAR(50) '$.LastName',
    Department NVARCHAR(50) '$.Department'
) AS EmployeeData;
```

Este comando lee el archivo **employees.json** del contenedor **jsonfiles** en *Azure Blob Storage* e importa los datos a la tabla **employee_data** de Azure SQL Database.

Ahora puedes ejecutar el siguiente comando SQL para comprobar la importación de datos: 

```sql
SELECT * FROM dbo.employee_data;
```

Deberías ver los datos del archivo **employees.json** importados en la tabla **employee_data**.

---

## Exportación de datos con una aplicación de Azure Function

En esta parte del laboratorio, crearás una aplicación de Azure Function en C# para exportar datos desde Azure SQL Database. Esta función recuperará los datos y los devolverá como una respuesta JSON.

### Creación de una aplicación de Azure Function en Visual Studio Code

Comencemos por crear una aplicación de Azure Function en Visual Studio Code:

1. Abra **Visual Studio Code**.
1. En el panel del explorador, ve a la carpeta **/Allfiles/Labs/04/azure-functions**.
1. Haz clic con el botón derecho del ratón en la carpeta **azure-functions** y selecciona **Abrir en terminal integrado**.
1. En el terminal de VS Code, inicia sesión en Azure con el siguiente comando:

    ```bash
    az login
    ```

1. (Opcional) Si tienes varias suscripciones, establece la suscripción activa:

    ```bash
    az account set --subscription <your-subscription-id>
    ```

1. Ejecuta el siguiente comando para crear una aplicación de Azure Function:

    ```bash
    $functionappname = "YourUniqueFunctionAppName"
    $resourcegroup = "YourResourceGroupName"
    $location = "YourLocation"
    # NOTE - The following should be a new storage account name where your Azure function will resided.
    # It should not be the same Storage Account name used to store the JSON file
    $storageaccount = "YourStorageAccountName"

    az storage account create --name $storageaccount --location $location --resource-group $resourcegroup --sku Standard_LRS
    
    az functionapp create --resource-group $resourcegroup --consumption-plan-location $location --runtime dotnet --name  $functionappname --os-type Linux --storage-account $storageaccount --functions-version 4
    
    ```

    ***Reemplaza los marcadores de posición por tus propios valores. No utilices el nombre de cuenta de almacenamiento usado para el archivo json; este script necesita crear una nueva cuenta de almacenamiento para almacenar la aplicación de Azure Function***.


### Creación de una nueva aplicación de funciones en Visual Studio Code

Vamos a crear una nueva función en Visual Studio Code para exportar datos desde Azure SQL Database:

Es posible que tengas que agregar la extensión de Azure Functions a Visual Studio Code si aún no lo has hecho. Para ello, busca **Azure Functions** en el panel de extensiones e instálalo.

1. En Visual Studio Code, presiona **Ctrl+Mayús+P** (Windows) o **Cmd+Mayús+P** (Mac) para abrir la paleta de comandos.
1. Escriba y seleccione **Azure Functions: Crear nuevo proyecto**.
1. Elige tu directorio **Aplicación de funciones**. Elige la carpeta **/Allfiles/Labs/04/azure-functions** del repositorio clonado de GitHub.
1. Elige **C#** como lenguaje.
1. Elige **.Net 8.0 LTS** como tiempo de ejecución.
1. Elige **Desencadenador HTTP** como plantilla.
1. Llama a la función **ExportDataFunction**.
1. Haz que el espacio de nombres sea **Contoso.ExportFunction**.
1. Asigna a la función el nivel de acceso **anónimo**.

### Escritura del código de C# para exportar datos

1. Es posible que la aplicación de funciones de Azure necesite que primero se instalen algunos paquetes. Puedes instalarlos con la ejecución de los siguientes comandos:

    ```bash
    dotnet add package Microsoft.Data.SqlClient
    dotnet add package Newtonsoft.Json
    dotnet restore
    
    npm install -g azure-functions-core-tools@4 --unsafe-perm true

    ```

1. Reemplaza el código de función de marcador de posición por el código de C# siguiente para consultar tu Azure SQL Database y devolver los resultados como JSON:

    ```csharp
    using System;
    using System.IO;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Azure.WebJobs;
    using Microsoft.Azure.WebJobs.Extensions.Http;
    using Microsoft.AspNetCore.Http;
    using Microsoft.Extensions.Logging;
    using Microsoft.Data.SqlClient;
    using Newtonsoft.Json;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    
    public static class ExportDataFunction
    {
        [FunctionName("ExportDataFunction")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
            ILogger log)
        {
            // Connection string to the database
            // NOTE: REPLACE THIS CONNECTION STRING WITH THE CONNECTION STRING OF YOUR AZURE SQL DATABASE
            string connectionString = "Server=tcp:yourserver.database.windows.net;Database=DataLabDatabase;User ID=youruserid;Password=yourpassword;Encrypt=True;";
            
            // List to hold employee data
            List<Employee> employees = new List<Employee>();
            
            try
            {
                // Establishing connection to the database
                using (SqlConnection conn = new SqlConnection(connectionString))
                {
                    await conn.OpenAsync();
                    var query = "SELECT EmployeeID, FirstName, LastName, Department FROM employee_data";
                    
                    // Executing the query
                    using (SqlCommand cmd = new SqlCommand(query, conn))
                    {
                        // Adding parameters to the query (if needed)
                        // cmd.Parameters.AddWithValue("@ParameterName", parameterValue);

                        using (SqlDataReader reader = await cmd.ExecuteReaderAsync())
                        {
                            // Reading data from the database
                            while (await reader.ReadAsync())
                            {
                                employees.Add(new Employee
                                {
                                    EmployeeID = (int)reader["EmployeeID"],
                                    FirstName = reader["FirstName"].ToString(),
                                    LastName = reader["LastName"].ToString(),
                                    Department = reader["Department"].ToString()
                                });
                            }
                        }
                    }
                }
            }
            catch (SqlException ex)
            {
                // Logging SQL errors
                log.LogError($"SQL Error: {ex.Message}");
                return new StatusCodeResult(500);
            }
            catch (System.Exception ex)
            {
                // Logging unexpected errors
                log.LogError($"Unexpected Error: {ex.Message}");
                return new StatusCodeResult(500);
            }
    
            // Returning the list of employees as a JSON response
            return new OkObjectResult(JsonConvert.SerializeObject(employees, Formatting.Indented));
        }
    
        // Employee class to hold employee data
        public class Employee
        {
            public int EmployeeID { get; set; }
            public string FirstName { get; set; }
            public string LastName { get; set; }
            public string Department { get; set; }
        }
    }
    ```

    *Recuerda reemplazar **connectionString** por la cadena de conexión a tu Azure SQL Database e introduce también tu contraseña sqladmin en la cadena de conexión.*

    > **Nota:** en un entorno de producción, restringe el acceso solo a las direcciones IP necesarias. Además, considera la posibilidad de usar identidades administradas para que la aplicación de funciones de Azure acceda a la base de datos en lugar de la autenticación de SQL. Para obtener más información, vea las [Identidades administradas en Microsoft Entra para Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).

1. Guarda el código de la función y asegúrate de que tu archivo **.csproj** incluye el paquete **Newtonsoft.Json** para serializar objetos en JSON. Si no está incluido, agrégalo:

    ```xml
    <PackageReference Include="Newtonsoft.Json" Version="13.X.X" />
    ```

Es hora de implementar la aplicación de funciones Azure en Azure.

### Implementación de la aplicación de funciones de Azure en Azure

1. En el terminal integrado de **Visual Studio Code**, ejecuta el siguiente comando para implementar la función de Azure en Azure:

    ```bash
    func azure functionapp publish <your-function-app-name>
    ```

    Reemplaza ***<your-function-app-name>*** por el nombre de tu aplicación de funciones de Azure.

1. Espere a que la implementación se complete.

### Obtención de la dirección URL de la aplicación de funciones de Azure

1. Abre el Azure Portal y ve a tu aplicación de funciones de Azure.
1. En la sección *Información general*, en la pestaña *Función*, verás tu nueva función, selecciónala.
1. En la pestaña **Código + prueba**, selecciona **Obtener dirección URL de la función**.
1. Copia el **valor predeterminado (clave de función)**, lo necesitaremos en breve. La URL debería tener este aspecto:
   
   ```url
   https://YourFunctionAppName.azurewebsites.net/api/ExportDataFunction?code=2pjO0HqRyz_13DHQg8ga-ysdDWbDU_eHdtlixbAHLVEGAzFuplomUg%3D%3D
   ```

### Prueba de la aplicación de funciones de Azure

1. Una vez completada la implementación, puedes probar la función enviando una solicitud HTTP a la dirección URL de la clave de función que has copiado anteriormente del terminal de Visual Studio Code:

    ```bash
    curl https://<your-function-app-name>.azurewebsites.net/api/ExportDataFunction?code=<the function key embedded to your function URL>
    ```

1. La respuesta debe contener los datos exportados de tu tabla ***employee_data*** en formato JSON.

Aunque esta función es un ejemplo sencillo, puedes ampliarla para incluir lógica y procesamiento de datos más complejos, como el filtrado, la ordenación y la agregación de datos, etc.  El código también se puede ampliar para incluir características de seguridad, registro y control de errores.

### Limpiar los recursos

Después de completar el laboratorio, puedes eliminar los recursos creados en este ejercicio para evitar incurrir en costes adicionales:

- Elimina la Azure SQL Database.
- Elimina la cuenta de Azure Storage.
- Elimina la aplicación de Azure Function.
- Elimina el grupo de recursos que contiene los recursos.
