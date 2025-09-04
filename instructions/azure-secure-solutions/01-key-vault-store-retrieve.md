---
lab:
  topic: Secure solutions in Azure
  title: Creación y recuperación de secretos de Azure Key Vault
  description: Aprenda a crear un almacén de claves y a crear y recuperar secretos con la CLI de Azure y también mediante programación.
---

# Creación y recuperación de secretos de Azure Key Vault

En este ejercicio, creará una instancia de Azure Key Vault, almacenará secretos mediante la CLI de Azure y compilará una aplicación de consola de .NET que pueda crear y recuperar secretos del almacén de claves. Obtendrá información sobre cómo configurar la autenticación, administrar secretos mediante programación y limpiará los recursos cuando termine.  

Tareas realizadas en este ejercicio:

* Creación de recursos de Azure Key Vault
* Almacenamiento de un secreto en un almacén de claves mediante la CLI de Azure
* Creación de una aplicación de consola de .NET para crear y recuperar secretos
* Limpieza de recursos

Este ejercicio se realiza en aproximadamente **30** minutos.

## Creación de una instancia de Azure Key Vault y adición de un secreto

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
    keyVaultName=mykeyvaultname$RANDOM
    ```

1. Ejecute el siguiente comando para obtener el nombre del almacén de claves y registrarlo. Lo necesitará más adelante en el ejercicio.

    ```
    echo $keyVaultName
    ```

1. Ejecute el siguiente comando para crear un recurso de Azure Key Vault. Esta operación puede tardar unos minutos en ejecutarse.

    ```
    az keyvault create --name $keyVaultName \
        --resource-group $resourceGroup --location $location
    ```

### Asignación de un rol al nombre de usuario de Microsoft Entra

Para crear y recuperar un secreto, asigne el usuario de Microsoft Entra al rol **Agente de secretos de Key Vault**. Esto proporciona a la cuenta de usuario permiso para establecer, eliminar y enumerar secretos. En un escenario típico, es posible que quiera separar las acciones de creación y lectura mediante la asignación de **Agente de secretos de Key Vault** a un grupo y **Usuario de secretos de Key Vault** (puede obtener y enumerar secretos) a otro.

1. Ejecute el siguiente comando para recuperar **userPrincipalName** de la cuenta. Esto representa a quién se le asignará el rol.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Ejecute el siguiente comando para recuperar el id. de recurso del almacén de claves. El id. de recurso establece el ámbito de la asignación de roles en un almacén de claves específico.

    ```
    resourceID=$(az keyvault show --resource-group $resourceGroup \
        --name $keyVaultName --query id --output tsv)
    ```

1. Ejecute el siguiente comando para crear y asignar el rol **Agente de secretos de Key Vault**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Key Vault Secrets Officer" \
        --scope $resourceID
    ```

A continuación, agregue un secreto al almacén de claves que ha creado.

### Adición y recuperación de un secreto con la CLI de Azure

1. Ejecute el siguiente comando para crear un secreto. 

    ```
    az keyvault secret set --vault-name $keyVaultName \
        --name "MySecret" --value "My secret value"
    ```

1. Ejecute el siguiente comando para recuperar el secreto y comprobar que se ha establecido.

    ```
    az keyvault secret show --name "MySecret" --vault-name $keyVaultName
    ```

    Este comando devuelve código JSON. La última línea contiene la contraseña en texto sin formato. 

    ```json
    "value": "My secret value"
    ```

## Creación de una aplicación de consola de .NET para almacenar y recuperar secretos

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Siga estos pasos en Cloud Shell.

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. Ejecute los comandos siguientes para crear un directorio que contenga el proyecto y cambie al directorio del proyecto.

    ```
    mkdir keyvault
    cd keyvault
    ```

1. Cree la aplicación de consola .NET.

    ```
    dotnet new console
    ```

1. Ejecute los siguientes comandos para agregar los paquetes **Azure.Identity** y **Azure.Security.KeyVault.Secrets** al proyecto.

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.Security.KeyVault.Secrets
    ```

### Adición del código de inicio para el proyecto

1. Ejecuta el siguiente comando en Cloud Shell para empezar a editar la aplicación.

    ```
    code Program.cs
    ```

1. Reemplace el contenido existente por el código siguiente. Reemplace **YOUR-KEYVAULT-NAME** con el nombre del almacén de claves real.

    ```csharp
    using Azure.Identity;
    using Azure.Security.KeyVault.Secrets;
    
    // Replace YOUR-KEYVAULT-NAME with your actual Key Vault name
    string KeyVaultUrl = "https://YOUR-KEYVAULT-NAME.vault.azure.net/";
    
    
    // ADD CODE TO CREATE A CLIENT
    
    
    
    // ADD CODE TO CREATE A MENU SYSTEM
    
    
    
    // ADD CODE TO CREATE A SECRET
    
    
    
    // ADD CODE TO LIST SECRETS
    
    
    ```

1. Presione **Ctrl+S** para guardar los cambios.

### Adición de código para completar la aplicación

Ahora es el momento de agregar código para completar la aplicación.

1. Busque el comentario **// ADD CODE TO CREATE A CLIENT** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Configure authentication options for connecting to Azure Key Vault
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create the Key Vault client using the URL and authentication credentials
    var client = new SecretClient(new Uri(KeyVaultUrl), new DefaultAzureCredential(options));
    ```

1. Busque el comentario **// ADD CODE TO CREATE A MENU SYSTEM** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Main application loop - continues until user types 'quit'
    while (true)
    {
        // Display menu options to the user
        Console.Clear();
        Console.WriteLine("\nPlease select an option:");
        Console.WriteLine("1. Create a new secret");
        Console.WriteLine("2. List all secrets");
        Console.WriteLine("Type 'quit' to exit");
        Console.Write("Enter your choice: ");
    
        // Read user input and convert to lowercase for easier comparison
        string? input = Console.ReadLine()?.Trim().ToLower();
        
        // Check if user wants to exit the application
        if (input == "quit")
        {
            Console.WriteLine("Goodbye!");
            break;
        }
    
        // Process the user's menu selection
        switch (input)
        {
            case "1":
                // Call the method to create a new secret
                await CreateSecretAsync(client);
                break;
            case "2":
                // Call the method to list all existing secrets
                await ListSecretsAsync(client);
                break;
            default:
                // Handle invalid input
                Console.WriteLine("Invalid option. Please enter 1, 2, or 'quit'.");
                break;
        }
    }
    ```

1. Busque el comentario **// ADD CODE TO CREATE A SECRET** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    async Task CreateSecretAsync(SecretClient client)
    {
        try
        {
            Console.Clear();
            Console.WriteLine("\nCreating a new secret...");
            
            // Get the secret name from user input
            Console.Write("Enter secret name: ");
            string? secretName = Console.ReadLine()?.Trim();
    
            // Validate that the secret name is not empty
            if (string.IsNullOrEmpty(secretName))
            {
                Console.WriteLine("Secret name cannot be empty.");
                return;
            }
            
            // Get the secret value from user input
            Console.Write("Enter secret value: ");
            string? secretValue = Console.ReadLine()?.Trim();
    
            // Validate that the secret value is not empty
            if (string.IsNullOrEmpty(secretValue))
            {
                Console.WriteLine("Secret value cannot be empty.");
                return;
            }
    
            // Create a new KeyVaultSecret object with the provided name and value
            var secret = new KeyVaultSecret(secretName, secretValue);
            
            // Store the secret in Azure Key Vault
            await client.SetSecretAsync(secret);
    
            Console.WriteLine($"Secret '{secretName}' created successfully!");
            Console.WriteLine("Press Enter to continue...");
            Console.ReadLine();
        }
        catch (Exception ex)
        {
            // Handle any errors that occur during secret creation
            Console.WriteLine($"Error creating secret: {ex.Message}");
        }
    }
    ```

1. Busque el comentario **// ADD CODE TO LIST SECRETS** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    async Task ListSecretsAsync(SecretClient client)
    {
        try
        {
            Console.Clear();
            Console.WriteLine("Listing all secrets in the Key Vault...");
            Console.WriteLine("----------------------------------------");
    
            // Get an async enumerable of all secret properties in the Key Vault
            var secretProperties = client.GetPropertiesOfSecretsAsync();
            bool hasSecrets = false;
    
            // Iterate through each secret property to retrieve full secret details
            await foreach (var secretProperty in secretProperties)
            {
                hasSecrets = true;
                try
                {
                    // Retrieve the actual secret value and metadata using the secret name
                    var secret = await client.GetSecretAsync(secretProperty.Name);
                    
                    // Display the secret information to the console
                    Console.WriteLine($"Name: {secret.Value.Name}");
                    Console.WriteLine($"Value: {secret.Value.Value}");
                    Console.WriteLine($"Created: {secret.Value.Properties.CreatedOn}");
                    Console.WriteLine("----------------------------------------");
                }
                catch (Exception ex)
                {
                    // Handle errors for individual secrets (e.g., access denied, secret not found)
                    Console.WriteLine($"Error retrieving secret '{secretProperty.Name}': {ex.Message}");
                    Console.WriteLine("----------------------------------------");
                }
            }
    
            // Inform user if no secrets were found in the Key Vault
            if (!hasSecrets)
            {
                Console.WriteLine("No secrets found in the Key Vault.");
            }
        }
        catch (Exception ex)
        {
            // Handle general errors that occur during the listing operation
            Console.WriteLine($"Error listing secrets: {ex.Message}");
        
        }
        Console.WriteLine("Press Enter to continue...");
        Console.ReadLine();
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

1. Ejecute el siguiente comando para iniciar la aplicación de consola. La aplicación mostrará el sistema de menús de la aplicación. 

    ```
    dotnet run
    ```

1. Ha creado un secreto al principio de este ejercicio, escriba **2** para recuperarlo y mostrarlo.

1. Escriba **1** y un nombre y un valor de secreto para crear un secreto.

1. Vuelva a enumerar los secretos para ver la nueva adición.

Escriba **quit** cuando haya terminado con la aplicación.

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, los recursos existentes fuera del ámbito de este 
