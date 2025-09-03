---
lab:
  topic: Azure events and messaging
  title: "Envío y recepción de mensajes desde Azure\_Queue Storage"
  description: "Obtenga información sobre cómo enviar mensajes desde Azure\_Queue Storage con el SDK Azure.StorageQueues de .NET."
---

# Envío y recepción de mensajes desde Azure Queue Storage

En este ejercicio, creará y configurará recursos de Azure Queue Storage y, después, compilará una aplicación .NET para enviar y recibir mensajes mediante el SDK **Azure.Storage.Queues**. Aprenderá a aprovisionar recursos de almacenamiento, administrar mensajes de cola y limpiar el entorno cuando termine. 

Tareas realizadas en este ejercicio:

* Creación de recursos de Azure Queue Storage
* Asignación de un rol al nombre de usuario de Microsoft Entra
* Creación de una aplicación de consola de .NET para enviar y recibir mensajes
* Limpieza de recursos

Este ejercicio se realiza en aproximadamente **30** minutos.

## Creación de recursos de Azure Queue Storage

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
    storAcctName=storactname$RANDOM
    ```

1. Necesitará el nombre asignado a la cuenta de almacenamiento más adelante en este ejercicio. Ejecute el siguiente comando y registre la salida.

    ```
    echo $storAcctName
    ```

1. Ejecute el comando siguiente para crear una cuenta de almacenamiento con la variable que ha creado antes. La operación tarda unos minutos en completarse.

    ```bash
    az storage account create --resource-group $resourceGroup \
        --name $storAcctName --location $location --sku Standard_LRS
    ```

### Asignación de un rol al nombre de usuario de Microsoft Entra

Para permitir que la aplicación envíe y reciba mensajes, asigne el usuario de Microsoft Entra al rol **Colaborador de datos de la cola de Storage**. Esto proporciona a la cuenta de usuario permiso para crear colas y enviar o recibir mensajes mediante RBAC de Azure. Siga estos pasos en Cloud Shell.

1. Ejecute el siguiente comando para recuperar **userPrincipalName** de la cuenta. Esto representa a quién se le asignará el rol.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Ejecute los siguientes comandos para recuperar el id. de recurso de la cuenta de almacenamiento. El id. de recurso establece el ámbito de la asignación de roles en un espacio de nombres específico.

    ```
    resourceID=$(az storage account show --resource-group $resourceGroup \
        --name $storAcctName --query id --output tsv)
    ```

1. Ejecute el comando siguiente para crear y asignar el rol **Colaborador de datos de la cola de Storage**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Queue Data Contributor" \
        --scope $resourceID
    ```

## Creación de una aplicación de consola de .NET para enviar y recibir mensajes

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Siga estos pasos en Cloud Shell.

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. Ejecute los comandos siguientes para crear un directorio que contenga el proyecto y cambie al directorio del proyecto.

    ```
    mkdir queuestor
    cd queuestor
    ```

1. Cree la aplicación de consola .NET.

    ```
    dotnet new console
    ```

1. Ejecute los siguientes comandos para agregar los paquetes **Azure.Storage.Queues** y **Azure.Identity** al proyecto.

    ```
    dotnet add package Azure.Storage.Queues
    dotnet add package Azure.Identity
    ```

### Adición del código de inicio para el proyecto

1. Ejecuta el siguiente comando en Cloud Shell para empezar a editar la aplicación.

    ```
    code Program.cs
    ```

1. Reemplace el contenido existente por el código siguiente. Asegúrese de revisar los comentarios en el código y reemplace **<YOUR-STORAGE-ACCT-NAME>** por el nombre de la cuenta de almacenamiento que ha registrado antes.

    ```csharp
    using Azure;
    using Azure.Identity;
    using Azure.Storage.Queues;
    using Azure.Storage.Queues.Models;
    using System;
    using System.Threading.Tasks;
    
    // Create a unique name for the queue
    // TODO: Replace the <YOUR-STORAGE-ACCT-NAME> placeholder 
    string queueName = "myqueue-" + Guid.NewGuid().ToString();
    string storageAccountName = "<YOUR-STORAGE-ACCT-NAME>";
    
    // ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE
    
    
    
    // ADD CODE TO SEND AND LIST MESSAGES
    
    
    
    // ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES
    
    
    
    // ADD CODE TO DELETE MESSAGES AND THE QUEUE
    
    
    ```

1. Presione **Ctrl+S** para guardar los cambios.

### Adición de código para crear un cliente de cola y crear una cola

Es el momento de agregar código para crear el cliente de Queue Storage y crear una cola.

1. Busque el comentario **// ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Create a DefaultAzureCredentialOptions object to exclude certain credentials
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Instantiate a QueueClient to create and interact with the queue
    QueueClient queueClient = new QueueClient(
        new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"),
        new DefaultAzureCredential(options));
    
    Console.WriteLine($"Creating queue: {queueName}");
    
    // Create the queue
    await queueClient.CreateAsync();
    
    Console.WriteLine("Queue created, press Enter to add messages to the queue...");
    Console.ReadLine();
    ```

1. Presione **CTRL+S** para guardar el archivo y, después, continúe con el ejercicio.

### Adición de código para enviar y enumerar mensajes en una cola

1. Busque el comentario **// ADD CODE TO SEND AND LIST MESSAGES** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Send several messages to the queue with the SendMessageAsync method.
    await queueClient.SendMessageAsync("Message 1");
    await queueClient.SendMessageAsync("Message 2");
    
    // Send a message and save the receipt for later use
    SendReceipt receipt = await queueClient.SendMessageAsync("Message 3");
    
    Console.WriteLine("Messages added to the queue. Press Enter to peek at the messages...");
    Console.ReadLine();
    
    // Peeking messages lets you view the messages without removing them from the queue.
    
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to update a message in the queue...");
    Console.ReadLine();
    ```

1. Presione **CTRL+S** para guardar el archivo y, después, continúe con el ejercicio.

### Adición de código para actualizar un mensaje y enumerar los resultados

1. Busque el comentario **// ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Update a message with the UpdateMessageAsync method and the saved receipt
    await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Message 3 has been updated");
    
    Console.WriteLine("Message three updated. Press Enter to peek at the messages again...");
    Console.ReadLine();
    
    
    // Peek messages from the queue to compare updated content
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to delete messages from the queue...");
    Console.ReadLine();
    ```

1. Presione **CTRL+S** para guardar el archivo y, después, continúe con el ejercicio.

### Adición de código para eliminar mensajes y la cola

1. Busque el comentario **// ADD CODE TO DELETE MESSAGES AND THE QUEUE** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Delete messages from the queue with the DeleteMessagesAsync method.
    foreach (var message in (await queueClient.ReceiveMessagesAsync(maxMessages: 10)).Value)
    {
        // "Process" the message
        Console.WriteLine($"Deleting message: {message.MessageText}");
    
        // Let the service know we're finished with the message and it can be safely deleted.
        await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
    }
    Console.WriteLine("Messages deleted from the queue.");
    Console.WriteLine("\nPress Enter key to delete the queue...");
    Console.ReadLine();
    
    // Delete the queue with the DeleteAsync method.
    Console.WriteLine($"Deleting queue: {queueClient.Name}");
    await queueClient.DeleteAsync();
    
    Console.WriteLine("Done");
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

1. Expanda **> Almacenamiento de datos** en la navegación de la izquierda y seleccione **Colas**.

1. Seleccione la cola que crea la aplicación y puede ver los mensajes enviados y supervisar lo que hace la aplicación.

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, los recursos existentes fuera del ámbito de este 

