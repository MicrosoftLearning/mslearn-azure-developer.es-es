---
lab:
  topic: Secure solutions in Azure
  title: "Recuperación de valores de configuración de Azure\_App Configuration"
  description: "Aprenda a crear un recurso de Azure\_App Configuration y a establecer información de configuración con la CLI de Azure. Después, use **ConfigurationBuilder** para recuperar la configuración de la aplicación."
---

# Recuperación de valores de configuración de Azure App Configuration

En este ejercicio, creará un recurso de Azure App Configuration, almacenará los valores de configuración mediante la CLI de Azure y compilará una aplicación de consola de .NET que use **ConfigurationBuilder** para recuperar los valores de configuración. Aprenderá a organizar la configuración con claves jerárquicas y a autenticar la aplicación para acceder a datos de configuración basados en la nube.

Tareas realizadas en este ejercicio:

* Creación de un recurso de Azure App Configuration
* Almacenamiento de la información de configuración de la cadena de conexión
* Creación de una aplicación de consola de .NET para recuperar la información de configuración
* Limpieza de recursos

Este ejercicio se realiza en aproximadamente **15** minutos.

## Creación de un recurso de Azure App Configuration y adición de información de configuración

En esta sección del ejercicio, creará los recursos necesarios en Azure con la CLI de Azure.

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

1. Cree un grupo de recursos para los recursos necesarios para este ejercicio. Si ya tiene un grupo de recursos que quiere usar. continúe al paso siguiente. Sustituya **myResourceGroup** por un nombre que quiera usar para el grupo de recursos. Puede reemplazar **eastus** por una región cercana si es necesario.

    ```
    az group create --name myResourceGroup --location eastus
    ```

1. Muchos de los comandos necesitan nombres únicos y usan los mismos parámetros. La creación de algunas variables reducirá los cambios necesarios para los comandos que crean recursos. Ejecute los comandos siguientes para crear las variables necesarias. Reemplace **myResourceGroup** por el nombre que vaya a usar para este ejercicio. Si ha cambiado la ubicación en el paso anterior, realice el mismo cambio en la variable **location**.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    appConfigName=appconfigname$RANDOM
    ```

1. Ejecute el siguiente comando para obtener el nombre del recurso de App Configuration. Registre el nombre, lo necesitará más adelante en el ejercicio.

    ```
    echo $appConfigName
    ```

1. Ejecute el siguiente comando para asegurarse de que el proveedor **Microsoft.AppConfiguration** está registrado para la suscripción.

    ```
    az provider register --namespace Microsoft.AppConfiguration
    ```

1. El registro puede tardar unos minutos en completarse. Ejecute el comando siguiente para comprobar el estado del registro. Continúe con el paso siguiente cuando los resultados devuelvan **Registrado**.

    ```
    az provider show --namespace Microsoft.AppConfiguration --query "registrationState"
    ```

1. Ejecute el comando siguiente para crear un recurso de Azure App Configuration. Esta operación puede tardar unos minutos en ejecutarse.

    ```
    az appconfig create --location $location \
        --name $appConfigName \
        --resource-group $resourceGroup
        --sku Free
    ```

    >**Sugerencia:** Si hay un problema al crear el recurso AppConfig debido a restricciones de cuota mediante el valor de SKU **Gratis**, use **Desarrollador** en su lugar.
    

### Asignación de un rol al nombre de usuario de Microsoft Entra

Para recuperar información de configuración, debe asignar el usuario de Microsoft Entra al rol **Lector de datos de App Configuration**. 

1. Ejecute el siguiente comando para recuperar **userPrincipalName** de la cuenta. Esto representa a quién se le asignará el rol.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Ejecute el comando siguiente para recuperar el id. de recurso del servicio App Configuration. El id. de recurso establece el ámbito de la asignación de roles.

    ```
    resourceID=$(az appconfig show --resource-group $resourceGroup \
        --name $appConfigName --query id --output tsv)
    ```

1. Ejecute el comando siguiente para crear y asignar el rol **Lector de datos de App Configuration**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "App Configuration Data Reader" \
        --scope $resourceID
    ```

A continuación, agregue una cadena de conexión de marcador de posición a App Configuration.

### Adición de información de configuración con la CLI de Azure

En Azure App Configuration, una clave como **Dev:conStr** es una clave jerárquica o con espacio de nombres. Los dos puntos (:) actúan como un delimitador que crea una jerarquía lógica, donde:

* **Dev** representa el espacio de nombres o el prefijo de entorno (que indica que esta configuración es para el entorno de desarrollo)
* **conStr** representa el nombre de la configuración

Esta estructura jerárquica permite organizar los valores de configuración por entorno, característica o componente de la aplicación, lo que facilita la administración y recuperación de la configuración relacionada.

Ejecute el comando siguiente para almacenar la cadena de conexión de marcador de posición. 

```
az appconfig kv set --name $appConfigName \
    --key Dev:conStr \
    --value connectionString \
    --yes
```

Este comando devuelve código JSON. La última línea contiene la contraseña en texto sin formato. 

```json
"value": "connectionString"
```

## Creación de una aplicación de consola de .NET para recuperar la información de configuración

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Siga estos pasos en Cloud Shell.

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. Ejecute los comandos siguientes para crear un directorio que contenga el proyecto y cambie al directorio del proyecto.

    ```
    mkdir appconfig
    cd appconfig
    ```

1. Cree la aplicación de consola .NET.

    ```
    dotnet new console
    ```

1. Ejecute los siguientes comandos para agregar los paquetes **Azure.Identity** y **Microsoft.Extensions.Configuration.AzureAppConfiguration** al proyecto.

    ```
    dotnet add package Azure.Identity
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

### Adición del código al proyecto

1. Ejecuta el siguiente comando en Cloud Shell para empezar a editar la aplicación.

    ```
    code Program.cs
    ```

1. Reemplace el contenido existente por el código siguiente. Asegúrese de reemplazar **YOUR_APP_CONFIGURATION_NAME** por el nombre que ha registrado antes y lea los comentarios del código.

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    using Azure.Identity;
    
    // Set the Azure App Configuration endpoint, replace YOUR_APP_CONFIGURATION_NAME
    // with the name of your actual App Configuration service
    
    string endpoint = "https://YOUR_APP_CONFIGURATION_NAME.azconfig.io"; 
    
    // Configure which authentication methods to use
    // DefaultAzureCredential tries multiple auth methods automatically
    DefaultAzureCredentialOptions credentialOptions = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a configuration builder to combine multiple config sources
    var builder = new ConfigurationBuilder();
    
    // Add Azure App Configuration as a source
    // This connects to Azure and loads configuration values
    builder.AddAzureAppConfiguration(options =>
    {
        
        options.Connect(new Uri(endpoint), new DefaultAzureCredential(credentialOptions));
    });
    
    // Build the final configuration object
    try
    {
        var config = builder.Build();
        
        // Retrieve a configuration value by key name
        Console.WriteLine(config["Dev:conStr"]);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error connecting to Azure App Configuration: {ex.Message}");
    }
    ```

1. Presione **Ctrl+S** para guardar el archivo y después **Ctr+Q** para salir del editor.

## Inicie sesión en Azure y ejecuta la aplicación.

1. En Cloud Shell, escriba el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulte [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para más información.

1. Ejecute el siguiente comando para iniciar la aplicación de consola. La aplicación mostrará el valor **connectionString** que ha asignado al valor **Dev:conStr** anteriormente en el ejercicio.

    ```
    dotnet run
    ```

    La aplicación mostrará el valor **connectionString** que ha asignado al valor **Dev:conStr** anteriormente en el ejercicio.

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, los recursos existentes fuera del ámbito de este 
