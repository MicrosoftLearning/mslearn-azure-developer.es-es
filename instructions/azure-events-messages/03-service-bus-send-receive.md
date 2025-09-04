---
lab:
  topic: Azure events and messaging
  title: Envío y recepción de mensajes de Azure Service Bus
  description: "Obtenga información sobre cómo enviar mensajes desde Azure\_Service Bus con el SDK Azure.Messaging.ServiceBus de .NET."
---

# Envío y recepción de mensajes de Azure Service Bus

En este ejercicio, creará y configurará recursos de Azure Service Bus y, después, compilará una aplicación .NET para enviar y recibir mensajes mediante el SDK **Azure.Messaging.ServiceBus**. Aprenderá a aprovisionar un espacio de nombres y una cola de Service Bus, asignar permisos e interactuar con mensajes mediante programación. 

Tareas realizadas en este ejercicio:

* Creación de recursos de Azure Service Bus
* Asignación de un rol al nombre de usuario de Microsoft Entra
* Creación de una aplicación de consola de .NET para enviar y recibir mensajes
* Limpieza de recursos

Este ejercicio se realiza en aproximadamente **30** minutos.

## Creación de recursos de Azure Event Hubs

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
    namespaceName=svcbusns$RANDOM
    ```

1. Necesitará el nombre asignado al espacio de nombres más adelante en este ejercicio. Ejecute el siguiente comando y registre la salida.

    ```
    echo $namespaceName
    ```

### Creación de un espacio de nombres y una cola de Azure Service Bus

1. Cree un espacio de nombres de mensajería de Service Bus. El comando siguiente crea un espacio de nombres con la variable que ha creado antes. La operación tarda unos minutos en completarse.

    ```bash
    az servicebus namespace create \
        --resource-group $resourceGroup \
        --name $namespaceName \
        --location $location
    ```

1. Ahora que se ha creado un espacio de nombres, debe crear una cola para contener los mensajes. Ejecute el comando siguiente para crear una cola denominada **myqueue**.

    ```bash
    az servicebus queue create --resource-group $resourceGroup \
        --namespace-name $namespaceName \
        --name myqueue
    ```

### Asignación de un rol al nombre de usuario de Microsoft Entra

Para permitir que la aplicación envíe y reciba mensajes, asigne el usuario de Microsoft Entra al rol **Propietario de datos de Azure Service Bus** en el nivel de espacio de nombres de Service Bus. Esto proporciona a la cuenta de usuario permiso para administrar y acceder a colas y temas mediante RBAC de Azure. Siga estos pasos en Cloud Shell.

1. Ejecute el siguiente comando para recuperar **userPrincipalName** de la cuenta. Esto representa a quién se le asignará el rol.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Ejecute el comando siguiente para recuperar el id. de recurso del espacio de nombres de Service Bus. El id. de recurso establece el ámbito de la asignación de roles en un espacio de nombres específico.

    ```
    resourceID=$(az servicebus namespace show --name $namespaceName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. Ejecute el comando siguiente para crear y asignar el rol **Propietario de datos de Azure Service Bus**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Azure Service Bus Data Owner" \
        --scope $resourceID
    ```

## Creación de una aplicación de consola de .NET para enviar y recibir mensajes

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Siga estos pasos en Cloud Shell.

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. Ejecute los comandos siguientes para crear un directorio que contenga el proyecto y cambie al directorio del proyecto.

    ```
    mkdir svcbus
    cd svcbus
    ```

1. Cree la aplicación de consola .NET.

    ```
    dotnet new console
    ```

1. Ejecute los siguientes comandos para agregar los paquetes **Azure.Messaging.ServiceBus** y **Azure.Identity** al proyecto.

    ```
    dotnet add package Azure.Messaging.ServiceBus
    dotnet add package Azure.Identity
    ```

### Adición del código de inicio para el proyecto

1. Ejecuta el siguiente comando en Cloud Shell para empezar a editar la aplicación.

    ```
    code Program.cs
    ```

1. Reemplace el contenido existente por el código siguiente. Asegúrese de revisar los comentarios en el código y reemplace **<YOUR-NAMESPACE>** por el espacio de nombres de Service Bus que ha registrado antes.

    ```csharp
    using Azure.Messaging.ServiceBus;
    using Azure.Identity;
    using System.Timers;
    
    
    // TODO: Replace <YOUR-NAMESPACE> with your Service Bus namespace
    string svcbusNameSpace = "<YOUR-NAMESPACE>.servicebus.windows.net";
    string queueName = "myQueue";
    
    
    // ADD CODE TO CREATE A SERVICE BUS CLIENT
    
    
    
    // ADD CODE TO SEND MESSAGES TO THE QUEUE
    
    
    
    // ADD CODE TO PROCESS MESSAGES FROM THE QUEUE
    
    
    
    // Dispose client after use
    await client.DisposeAsync();
    ```

1. Presione **Ctrl+S** para guardar los cambios.

### Adición de código para enviar mensajes a la cola

Es el momento de agregar código para crear el cliente de Service Bus y enviar un lote de mensajes a la cola.

1. Busque el comentario **// ADD CODE TO CREATE A SERVICE BUS CLIENT** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a Service Bus client using the namespace and DefaultAzureCredential
    // The DefaultAzureCredential will use the Azure CLI credentials, so ensure you are logged in
    ServiceBusClient client = new(svcbusNameSpace, new DefaultAzureCredential(options));
    ```

1. Busque el comentario **// ADD CODE TO SEND MESSAGES TO THE QUEUE** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Create a sender for the specified queue
    ServiceBusSender sender = client.CreateSender(queueName);
    
    // create a batch 
    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();
    
    // number of messages to be sent to the queue
    const int numOfMessages = 3;
    
    for (int i = 1; i <= numOfMessages; i++)
    {
        // try adding a message to the batch
        if (!messageBatch.TryAddMessage(new ServiceBusMessage($"Message {i}")))
        {
            // if it is too large for the batch
            throw new Exception($"The message {i} is too large to fit in the batch.");
        }
    }
    
    try
    {
        // Use the producer client to send the batch of messages to the Service Bus queue
        await sender.SendMessagesAsync(messageBatch);
        Console.WriteLine($"A batch of {numOfMessages} messages has been published to the queue.");
    }
    finally
    {
        // Calling DisposeAsync on client types is required to ensure that network
        // resources and other unmanaged objects are properly cleaned up.
        await sender.DisposeAsync();
    }
    
    Console.WriteLine("Press any key to continue");
    Console.ReadKey();
    ```

1. Presione **CTRL+S** para guardar el archivo y, después, continúe con el ejercicio.

### Adición de código para procesar mensajes en la cola

1. Busque el comentario **// ADD CODE TO PROCESS MESSAGES FROM THE QUEUE** y agregue el código siguiente directamente después. Asegúrese de revisar el código y los comentarios.

    ```csharp
    // Create a processor that we can use to process the messages in the queue
    ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());
    
    // Idle timeout in milliseconds, the idle timer will stop the processor if there are no more 
    // messages in the queue to process
    const int idleTimeoutMs = 3000;
    System.Timers.Timer idleTimer = new(idleTimeoutMs);
    idleTimer.Elapsed += async (s, e) =>
    {
        Console.WriteLine($"No messages received for {idleTimeoutMs / 1000} seconds. Stopping processor...");
        await processor.StopProcessingAsync();
    };
    
    try
    {
        // add handler to process messages
        processor.ProcessMessageAsync += MessageHandler;
    
        // add handler to process any errors
        processor.ProcessErrorAsync += ErrorHandler;
    
        // start processing 
        idleTimer.Start();
        await processor.StartProcessingAsync();
    
        Console.WriteLine($"Processor started. Will stop after {idleTimeoutMs / 1000} seconds of inactivity.");
        // Wait for the processor to stop
        while (processor.IsProcessing)
        {
            await Task.Delay(500);
        }
        idleTimer.Stop();
        Console.WriteLine("Stopped receiving messages");
    }
    finally
    {
        // Dispose processor after use
        await processor.DisposeAsync();
    }
    
    // handle received messages
    async Task MessageHandler(ProcessMessageEventArgs args)
    {
        string body = args.Message.Body.ToString();
        Console.WriteLine($"Received: {body}");
    
        // Reset the idle timer on each message
        idleTimer.Stop();
        idleTimer.Start();
    
        // complete the message. message is deleted from the queue. 
        await args.CompleteMessageAsync(args.Message);
    }
    
    // handle any errors when receiving messages
    Task ErrorHandler(ProcessErrorEventArgs args)
    {
        Console.WriteLine(args.Exception.ToString());
        return Task.CompletedTask;
    }
    ```

1. Presione **Ctrl+S** para guardar el archivo y después **Ctr+Q** para salir del editor.

## Inicie sesión en Azure y ejecuta la aplicación.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulte [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para más información.

1. Ejecute el siguiente comando para iniciar la aplicación de consola. La aplicación se pausará durante varias fases y le pedirá que presione una tecla para continuar. Esto le ofrece la oportunidad de ver los mensajes en Azure Portal.

    ```
    dotnet run
    ```

    

1. En Azure Portal, vaya al espacio de nombres de Service Bus que ha creado. 

1. Seleccione **myqueue** en la parte inferior de la ventana **Información general**.

1. Seleccione **Service Bus Explorer** en el panel de navegación de la izquierda.

1. Seleccione **Ver desde el principio** y los tres mensajes deberían aparecer después de unos segundos.

1. En Cloud Shell, presione cualquier tecla para continuar y la aplicación procesará los tres mensajes. 
 
1. Vuelva al portal después de que la aplicación haya completado el procesamiento de los mensajes. Vuelva a seleccionar **Ver desde el principio** y observe que no hay ningún mensaje en la cola.

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, los recursos existentes fuera del ámbito de este 

