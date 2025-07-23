---
lab:
  title: Configuración e implementación de canalizaciones de CI/CD para proyectos de Azure SQL Database
  module: Develop for an Azure SQL Database
---

# Configuración e implementación de canalizaciones CI/CD para proyectos de Azure SQL Database

En este ejercicio creará, configurará e implementará canalizaciones CI/CD para proyectos de Azure SQL Database mediante Visual Studio Code y Acciones de GitHub. Así se familiarizará con el proceso de configuración de canalizaciones CI/CD para proyectos de Azure SQL Database.

Este ejercicio debería tardar en completarse **30** minutos aproximadamente.

## Antes de comenzar

Antes de empezar este ejercicio, necesitas:

- Una suscripción a Azure con los permisos adecuados para crear y administrar recursos.
- [Visual Studio Code](https://code.visualstudio.com/download) instalado en tu equipo con las siguientes extensiones:
  - [Proyectos de SQL Database](https://marketplace.visualstudio.com/items?itemName=ms-mssql.mssql).
  - [Solicitud de incorporación de cambios de GitHub](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github).
- Cuentas de GitHub.
- Conocimientos básicos de canalizaciones de *Acciones de GitHub*.

## Crear una instancia de Azure SQL Database

En primer lugar, debes crear una nueva Azure SQL Database

1. Inicie sesión en [Azure Portal](https://portal.azure.com?azure-portal=true). 
1. Vaya a la página **Azure SQL** y seleccione **+ Crear**.
1. Seleccione **SQL Database**, *Base de datos* única y el botón **Crear**.
1. Rellena la información necesaria en el cuadro de diálogo **Crear SQL Database**, selecciona **Aceptar** y deja todas las demás opciones con su configuración predeterminada.

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
1. Selecciona la opción **Permitir que los servicios y recursos de Azure accedan a este servidor**. Esta opción permite que Acciones de GitHub acceda a la base de datos.

    > **Nota:** en un entorno de producción, restringe el acceso solo a las direcciones IP necesarias. Además, considera la posibilidad de usar identidades administradas para la Acción de GitHub para acceder a la base de datos en lugar de la autenticación de SQL. Para obtener más información, vea las [Identidades administradas en Microsoft Entra para Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-azure-ad-user-assigned-managed-identity?azure-portal=true).

1. Seleccione **Guardar**.

## Configuración de un repositorio de GitHub

Después, debes crear un nuevo repositorio de GitHub.

1. Abre el sitio web de [GitHub](https://github.com).
1. Inicie sesión en su cuenta de GitHub.
1. Ve a **Repositorios** en tu cuenta y selecciona **Nuevo**.
1. Selecciona tu cuenta en **Propietario**. Escribe el nombre **my-sql-db-repo**.
1. Establece el repositorio en **Privado**.
1. Seleccione **Crear repositorio**.

### Instalación de las extensiones de Visual Studio Code y clonación del repositorio

Antes de clonar el repositorio, asegúrate de que has instalado las extensiones necesarias de **Visual Studio Code**. Consulta la sección **Antes de empezar** para obtener más información.

1. En el Visual Studio Code, seleccione **Ver** > **Paleta de comandos**.
1. En la paleta de comandos, escribe `Git: Clone` y selecciónalo.
1. Escribe la dirección URL del repositorio que has creado en el paso anterior y selecciona **Clonar**. Tu dirección URL debería seguir este formato: *https://github.com/<your_account>/<your_repository>.git*
1. Seleccione o cree una carpeta para almacenar los archivos del repositorio.

## Creación y configuración de un proyecto de Azure SQL Database

Un proyecto de Azure SQL Database en Visual Studio te permite desarrollar, compilar, probar y publicar el esquema y los datos de la base de datos. En esta sección, crearás un proyecto y lo configurarás para conectarse a la instancia de Azure SQL Database que has configurado anteriormente.

1. En el Visual Studio Code, seleccione **Ver** > **Paleta de comandos**.
1. En la paleta de comandos, escribe `Database projects: New` y selecciónalo.
    > **Nota:** la instalación del servicio de herramientas de SQL para la extensión MSSQL puede tardar unos minutos.
1. Seleccione **Azure SQL Database**.
1. Escribe el nombre **MyDBProj** y presiona **Entrar** para confirmar.
1. Selecciona la carpeta del repositorio de GitHub clonada para guardar el proyecto.
1. En **proyectos de estilo SDK**, selecciona **Sí (recomendado)**.
    > **Nota:** observa que se crea un nuevo proyecto con el nombre **MyDBProj**.

### Creación de un nuevo archivo SQL en el proyecto

Con el proyecto de Azure SQL Database creado, vamos a agregar un nuevo archivo SQL al proyecto para crear una nueva tabla.

1. En Visual Studio Code, selecciona el icono **Proyectos de base de datos** ubicado en la barra de actividades de la izquierda.
1. Haz clic con el botón derecho en el nombre del proyecto y selecciona **Agregar tabla**.
1. Asigna un nombre a la tabla **Empleados** y presiona **Entrar**.
1. Reemplaza el script existente por el siguiente código.

    ```sql
    CREATE TABLE [dbo].[Employees]
    (
        EmployeeID INT PRIMARY KEY,
        FirstName NVARCHAR(50),
        LastName NVARCHAR(50),
        Department NVARCHAR(50)
    );
    ```

1. Cierre el editor. Observa que el archivo `Employees.sql` se guarda en el proyecto.

## Confirma los cambios en el repositorio

Con el proyecto de Azure SQL Database creado y el script de tabla agregado al proyecto, vamos a confirmar los cambios en el repositorio.

1. En Visual Studio Code, selecciona el icono **Control de código fuente** ubicado en la barra de actividades de la izquierda.
1. Escribe el mensaje *Proyecto creado y agregado un script de creación de tabla*.
1. Selecciona **Confirmar** para confirmar los cambios.
1. En los puntos suspensivos, selecciona **Insertar** para enviar los cambios al repositorio.

## Comprobación de los cambios en el repositorio

Ahora que has insertado los cambios, vamos a comprobarlos en el repositorio de GitHub.

1. Abre el sitio web de [GitHub](https://github.com).
1. Ve al repositorio **my-sql-db-repo**.
1. En la pestaña **<>Código**, abre la carpeta **MyDBProj**.
1. Comprueba si los cambios en el archivo **Employees.sql** están actualizados.

## Configurar la integración continua (CI) con Acciones de GitHub

Acciones de GitHub te permite automatizar, personalizar y ejecutar tus flujos de trabajo de desarrollo de software directamente en tu repositorio de GitHub. En esta sección, configurarás un flujo de trabajo de Acciones de GitHub para compilar y probar el proyecto de Azure SQL Database mediante la creación de una nueva tabla en la base de datos.

### Creación de una entidad de servicio

1. Selecciona el icono **Cloud Shell** en la esquina superior derecha de Azure Portal. Parece un símbolo `>_`. Si se te solicita, elige **Bash** como tipo de shell.

1. Ejecuta el siguiente comando en el terminal de Cloud Shell. Reemplaza los valores `<your_subscription_id>`, y `<your_resource_group_name>` por tus valores reales. Puedes obtener estos valores en las páginas **Suscripciones** y **Grupos de recursos** de Azure Portal.

    ```azurecli
    az ad sp create-for-rbac --name "MyDBProj" --role contributor --scopes /subscriptions/<your_subscription_id>/resourceGroups/<your_resource_group_name>
    ```

    Abre un editor de texto y usa la salida del comando anterior para crear un fragmento de código de credenciales similar al siguiente:
    
    ```
    {
    "clientId": <your_service_principal_appId>,
    "clientSecret": <your_service_principal_password>,
    "tenantId": <your_service_principal_tenant>,
    "subscriptionId": <your_subscription_id>
    }
    ```

1. Deje abierto el editor de texto. Haremos referencia a él en la siguiente sección.

### Incorporación de secretos al repositorio

1. En el repositorio de GitHub, selecciona **Configuración**.
1. Seleccione **Secretos y variables** y después **Acciones**.
1. En la pestaña **Secretos**, selecciona **Nuevo secreto del repositorio**, y proporciona la siguiente información.

    | NOMBRE | Valor |
    | --- | --- |
    | AZURE_CREDENTIALS | Salida de la entidad de servicio copiada en la sección anterior.|
    | AZURE_CONN_STRING | Su cadena de conexión. |
   
    Tu cadena de conexión debe tener un aspecto similar al siguiente:

    ```Server=<your_sqldb_server>.database.windows.net;Initial Catalog=MyDB;Persist Security Info=False;User ID=sqladmin;Password=<your_password>;Encrypt=True;Connection Timeout=30;```

### Creación de un flujo de trabajo de Acciones de GitHub

1. En el repositorio de GitHub, seleccione la pestaña **Acciones** .
1. Selecciona el vínculo **Configurar un flujo de trabajo usted mismo**.
1. Copia el siguiente código en tu archivo **main.yml**. El código incluye los pasos para compilar e implementar el proyecto de base de datos.

    {% raw %}
    ```yaml
    name: Build and Deploy SQL Database Project
    on:
      push:
        branches:
          - main
    jobs:
      build:
        permissions:
          contents: 'read'
          id-token: 'write'
          
        runs-on: ubuntu-latest  # Can also use windows-latest depending on your environment
        steps:
          - name: Checkout repository
            uses: actions/checkout@v3

        # Install the SQLpackage tool
          - name: sqlpack install
            run: dotnet tool install -g microsoft.sqlpackage
    
          - name: Login to Azure
            uses: azure/login@v1
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}
    
          # Build and Deploy SQL Project
          - name: Build and Deploy SQL Project
            uses: azure/sql-action@v2.3
            with:
              connection-string: ${{ secrets.AZURE_CONN_STRING }}
              path: './MyDBProj/MyDBProj.sqlproj'
              action: 'publish'
              build-arguments: '-c Release'
              arguments: '/p:DropObjectsNotInSource=true'  # Optional: Customize as needed
    ```
    {% endraw %}
   
      El paso **Compilar e implementar un proyecto de SQL** de tu archivo YAML se conecta a tu base de datos Azure SQL Database mediante la cadena de conexión almacenada en el secreto `AZURE_CONN_STRING`. La acción especifica la ruta de acceso al archivo de proyecto de SQL, establece la acción que se va a publicar para implementar el proyecto e incluye argumentos de compilación que se van a compilar en modo de versión. Además, usa el argumento `/p:DropObjectsNotInSource=true` para garantizar que cualquier objeto que no esté presente en el origen se quite de la base de datos de destino durante la implementación.

1. Confirma los cambios.

### Prueba del flujo de trabajo de Acciones de GitHub

1. En el repositorio de GitHub, seleccione la pestaña **Acciones** .
1. Selecciona el flujo de trabajo **Compilar e implementar un proyecto de SQL Database**.
    > **Nota:** verás el flujo de trabajo en curso. Espere hasta que se complete la operación. Si ya ha finalizado, selecciona la ejecución más reciente para ver los detalles.

### Comprobación de los cambios en Azure SQL Database

Con el flujo de trabajo de Acciones de GitHub configurado para compilar e implementar el proyecto de Azure SQL Database, es el momento de comprobar los cambios en tu Azure SQL Database.

1. Inicie sesión en [Azure Portal](https://portal.azure.com?azure-portal=true). 
1. Ve a la **MyDB** de la base de datos de Azure SQL:
1. Seleccione **Editor de consultas**.
1. Conéctate a la base de datos usando las credenciales **sqladmin**.
1. En la sección **Tablas**, comprueba que se ha creado la tabla **Empleados**. Si es necesario, actualiza.

Has configurado correctamente un flujo de trabajo de Acciones de GitHub para compilar e implementar el proyecto de Azure SQL Database.

## Limpieza

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. 

Dejar que los recursos se ejecuten innecesariamente puede dar lugar a costes adicionales. Puede eliminar recursos de forma individual o bien eliminar el grupo de recursos completo desde [Azure Portal](https://portal.azure.com?azure-portal=true).

## Más información

Para obtener más información sobre la extensión Proyectos de SQL Database para Azure SQL Database, consulta [Introducción a la extensión Proyectos de base de datos SQL](https://learn.microsoft.com/azure-data-studio/extensions/sql-database-project-extension-getting-started?azure-portal=true).
