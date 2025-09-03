---
lab:
  topic: Azure Storage
  title: "Creación de recursos de\_Blob Storage con la biblioteca cliente de .NET"
  description: "Aprenda a usar la biblioteca cliente .NET de Azure\_Storage para crear contenedores, cargar y enumerar blobs, y eliminar contenedores."
---

# Creación de recursos de Blob Storage con la biblioteca cliente de .NET

En este ejercicio, creará una cuenta de Azure Storage y compilará una aplicación de consola de .NET mediante la biblioteca cliente de Azure Storage Blob para crear contenedores, cargar archivos en Blob Storage, enumerar blobs y descargar archivos. Aprenderá a autenticarse con Azure, realizar operaciones de Blob Storage mediante programación y comprobar los resultados en Azure Portal.

Tareas realizadas en este ejercicio:

* Preparación de los recursos de Azure
* Creación de una aplicación de consola para crear y descargar datos
* Ejecución de la aplicación y comprobación de los resultados
* Limpieza de recursos

Este ejercicio se realiza en aproximadamente **30** minutos.

## Creación de una cuenta de Azure Storage

En esta sección del ejercicio, creará los recursos necesarios en Azure con la CLI de Azure.

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

1. Cree un grupo de recursos para los recursos necesarios para este ejercicio. Sustituya **myResourceGroup** por un nombre que quiera usar para el grupo de recursos. Puede reemplazar **eastus2** por una región cercana si es necesario. Si ya tiene un grupo de recursos que quiere usar. continúe al paso siguiente.

    ```
    az group create --location eastus2 --name myResourceGroup
    ```

1. Muchos de los comandos necesitan nombres únicos y usan los mismos parámetros. La creación de algunas variables reducirá los cambios necesarios para los comandos que crean recursos. Ejecute los comandos siguientes para crear las variables necesarias. Reemplace **myResourceGroup** por el nombre que vaya a usar para este ejercicio.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    accountName=storageacct$RANDOM
    ```

1. Ejecute los siguientes comandos para crear la cuenta de Azure Storage; cada nombre de cuenta debe ser único. El primer comando crea una variable con un nombre único para la cuenta de almacenamiento. Registre el nombre de la cuenta desde la salida del comando **echo**. 

    ```
    az storage account create --name $accountName \
        --resource-group $resourceGroup \
        --location $location \
        --sku Standard_LRS 
    
    echo $accountName
    ```

### Asignación de un rol al nombre de usuario de Microsoft Entra

Para permitir que la aplicación cree recursos y elementos, asigne el usuario de Microsoft Entra al rol **Propietario de datos de blobs de almacenamiento**. Siga estos pasos en Cloud Shell.

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. Ejecute el siguiente comando para recuperar **userPrincipalName** de la cuenta. Esto representa a quién se le asignará el rol.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Ejecute los siguientes comandos para recuperar el id. de recurso de la cuenta de almacenamiento. El id. de recurso establece el ámbito de la asignación de roles en un espacio de nombres específico.

    ```
    resourceID=$(az storage account show --name $accountName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. Ejecute el comando siguiente para crear y asignar el rol **Propietario de datos de blobs de almacenamiento**. Este rol le proporciona los permisos para administrar contenedores y elementos.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Blob Data Owner" \
        --scope $resourceID
    ```

## Creación de una aplicación de consola de .NET para crear contenedores y elementos

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Siga estos pasos en Cloud Shell.

1. Ejecute los comandos siguientes para crear un directorio que contenga el proyecto y cambie al directorio del proyecto.

    ```
    mkdir azstor
    cd azstor
    ```

1. Cree la aplicación de consola .NET.

    ```
    dotnet new console
    ```

1. Ejecute los comandos siguientes para agregar los paquetes necesarios en la aplicación.

    ```
    dotnet add package Azure.Storage.Blobs
    dotnet add package Azure.Identity
    ```

1. Ejecute el siguiente comando para crear una carpeta **data** en el proyecto. 

    ```
    mkdir data
    ```

Ahora es el momento de agregar el código para el proyecto.

### Adición del código de inicio para el proyecto

1. Ejecuta el siguiente comando en Cloud Shell para empezar a editar la aplicación.

    ```
    code Program.cs
    ```

1. Reemplace el contenido existente por el código siguiente. Asegúrese de revisar los comentarios en el código.

    ```csharp
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Identity;
    
    Console.WriteLine("Azure Blob Storage exercise\n");
    
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Run the examples asynchronously, wait for the results before proceeding
    await ProcessAsync();
    
    Console.WriteLine("\nPress enter to exit the sample application.");
    Console.ReadLine();
    
    async Task ProcessAsync()
    {
        // CREATE A BLOB STORAGE CLIENT
        
    
    
        // CREATE A CONTAINER
        
    
    
        // CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE
        
    
    
        // UPLOAD THE FILE TO BLOB STORAGE
        
    
    
        // LIST BLOBS IN THE CONTAINER
        
    
    
        // DOWNLOAD THE BLOB TO A LOCAL FILE
        
    
    }
    ```

1. Presione **CTRL+S** para guardar los cambios y continuar con el paso siguiente.


## Adición de código para completar el proyecto

En el resto del ejercicio agregará código en áreas especificadas para crear la aplicación completa. 

1. Busque el comentario **// CREATE A BLOB STORAGE CLIENT** y, después, agregue el código siguiente directamente debajo. **BlobServiceClient** actúa como punto de entrada principal para administrar contenedores y blobs en una cuenta de almacenamiento. El cliente usa * DefaultAzureCredential* para la autenticación. Asegúrese de reemplazar **YOUR_ACCOUNT_NAME** por el nombre que ha registrado antes.

    ```csharp
    // Create a credential using DefaultAzureCredential with configured options
    string accountName = "YOUR_ACCOUNT_NAME"; // Replace with your storage account name
    
    // Use the DefaultAzureCredential with the options configured at the top of the program
    DefaultAzureCredential credential = new DefaultAzureCredential(options);
    
    // Create the BlobServiceClient using the endpoint and DefaultAzureCredential
    string blobServiceEndpoint = $"https://{accountName}.blob.core.windows.net";
    BlobServiceClient blobServiceClient = new BlobServiceClient(new Uri(blobServiceEndpoint), credential);
    ```

1. Presione **CTRL+S** para guardar los cambios y continuar con el paso siguiente.

1. Busque el comentario **// CREATE A CONTAINER** y, después, agregue el código siguiente directamente debajo. La creación de un contenedor incluye la creación de una instancia de la clase **BlobServiceClient** y, después, la llamada al método **CreateBlobContainerAsync** para crear el contenedor en la cuenta de almacenamiento. Se anexa un valor GUID al nombre del contenedor para asegurarse de que es único. Se produce un error en el método **CreateBlobContainerAsync** si el contenedor ya existe.

    ```csharp
    // Create a unique name for the container
    string containerName = "wtblob" + Guid.NewGuid().ToString();
    
    // Create the container and return a container client object
    Console.WriteLine("Creating container: " + containerName);
    BlobContainerClient containerClient = 
        await blobServiceClient.CreateBlobContainerAsync(containerName);
    
    // Check if the container was created successfully
    if (containerClient != null)
    {
        Console.WriteLine("Container created successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("Failed to create the container, exiting program.");
        return;
    }
    ```

1. Presione **CTRL+S** para guardar los cambios y continuar con el paso siguiente.

1. Busque el comentario **// CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE** y, después, agregue el código siguiente directamente debajo. Esto crea un archivo en el directorio de datos que se carga en el contenedor.

    ```csharp
    // Create a local file in the ./data/ directory for uploading and downloading
    Console.WriteLine("Creating a local file for upload to Blob storage...");
    string localPath = "./data/";
    string fileName = "wtfile" + Guid.NewGuid().ToString() + ".txt";
    string localFilePath = Path.Combine(localPath, fileName);
    
    // Write text to the file
    await File.WriteAllTextAsync(localFilePath, "Hello, World!");
    Console.WriteLine("Local file created, press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Presione **CTRL+S** para guardar los cambios y continuar con el paso siguiente.

1. Busque el comentario **// UPLOAD THE FILE TO BLOB STORAGE** y, después, agregue el código siguiente directamente debajo. El código obtiene una referencia a un objeto **BlobClient** mediante la llamada al método **GetBlobClient** en el contenedor creado en la sección anterior. Después, carga un archivo local generado mediante el método **UploadAsync**. Esta operación crea el blob si no existe y lo sobrescribe, en caso de que ya exista.

    ```csharp
    // Get a reference to the blob and upload the file
    BlobClient blobClient = containerClient.GetBlobClient(fileName);
    
    Console.WriteLine("Uploading to Blob storage as blob:\n\t {0}", blobClient.Uri);
    
    // Open the file and upload its data
    using (FileStream uploadFileStream = File.OpenRead(localFilePath))
    {
        await blobClient.UploadAsync(uploadFileStream);
        uploadFileStream.Close();
    }
    
    // Verify if the file was uploaded successfully
    bool blobExists = await blobClient.ExistsAsync();
    if (blobExists)
    {
        Console.WriteLine("File uploaded successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("File upload failed, exiting program..");
        return;
    }
    ```

1. Presione **CTRL+S** para guardar los cambios y continuar con el paso siguiente.

1. Busque el comentario **// LIST BLOBS IN THE CONTAINER** y, después, agregue el código siguiente directamente debajo. Enumere los blobs en el contenedor con el método **GetBlobsAsync**. En este caso, solo se agregó un blob al contenedor, por lo que la operación de enumeración devuelve solo ese blob. 

    ```csharp
    Console.WriteLine("Listing blobs in container...");
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        Console.WriteLine("\t" + blobItem.Name);
    }
    
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Presione **CTRL+S** para guardar los cambios y continuar con el paso siguiente.

1. Busque el comentario **// DOWNLOAD THE BLOB TO A LOCAL FILE** y, después, agregue el código siguiente directamente debajo. El código usa el método **DownloadAsync** para descargar en el sistema de archivos local el blob creado anteriormente. El código de ejemplo agrega un sufijo "DOWNLOADED" al nombre del blob para que pueda ver ambos archivos en el sistema de archivos local. 

    ```csharp
    // Adds the string "DOWNLOADED" before the .txt extension so it doesn't 
    // overwrite the original file
    
    string downloadFilePath = localFilePath.Replace(".txt", "DOWNLOADED.txt");
    
    Console.WriteLine("Downloading blob to: {0}", downloadFilePath);
    
    // Download the blob's contents and save it to a file
    BlobDownloadInfo download = await blobClient.DownloadAsync();
    
    using (FileStream downloadFileStream = File.OpenWrite(downloadFilePath))
    {
        await download.Content.CopyToAsync(downloadFileStream);
    }
    
    Console.WriteLine("Blob downloaded successfully to: {0}", downloadFilePath);
    ```

1. Presione **Ctrl+S** para guardar el archivo y después **Ctr+Q** para salir del editor.

## Inicie sesión en Azure y ejecuta la aplicación.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulte [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para más información.

1. Ejecute el siguiente comando para iniciar la aplicación de consola. La aplicación se pausará muchas veces durante la ejecución, a la espera de que presione cualquier tecla para continuar. Esto le ofrece la oportunidad de ver los mensajes en Azure Portal.

    ```
    dotnet run
    ```

1. En Azure Portal, navegue hasta la cuenta de Azure Storage que ha creado. 

1. Expanda **> Almacenamiento de datos** en la navegación de la izquierda y seleccione **Contenedores**.

1. Seleccione el contenedor que ha creado la aplicación para ver el blob que se ha cargado.

1. Ejecute los dos comandos siguientes para cambiar al directorio **data** y enumerar los archivos que se han cargado y descargado.

    ```
    cd data
    ls
    ```

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, los recursos existentes fuera del ámbito de este 

