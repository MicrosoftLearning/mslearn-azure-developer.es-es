---
lab:
  topic: Azure events and messaging
  title: "Enrutamiento de eventos a un punto de conexión personalizado con Azure\_Event Grid"
  description: "Aprenda a usar Azure\_Event Grid para enruta eventos a un punto de conexión personalizado."
---

# Enrutamiento de eventos a un punto de conexión personalizado con Azure Event Grid

En este ejercicio, creará un tema de Azure Event Grid y un punto de conexión de aplicación web y, después, compilará una aplicación de consola de .NET que envía eventos personalizados al tema de Event Grid. Aprenderá a configurar suscripciones de eventos, a autenticarse con Event Grid y comprobar que los eventos se enrutan correctamente al punto de conexión mediante su visualización en la aplicación web.

Tareas realizadas en este ejercicio:

* Creación de recursos de Azure Event Grid
* Habilitar un proveedor de recursos de Event Grid
* Creación de un tema en Event Grid
* Creación de un punto de conexión de mensaje
* Suscripción al tema
* Envío de un evento con una aplicación de consola de .NET
* Limpieza de recursos

Este ejercicio se realiza en aproximadamente **30** minutos.

## Creación de recursos de Azure Event Grid

En esta sección del ejercicio, creará los recursos necesarios en Azure con la CLI de Azure.

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

1. Cree un grupo de recursos para los recursos necesarios para este ejercicio. Si ya tiene un grupo de recursos que quiere usar. continúe al paso siguiente. Sustituya **myResourceGroup** por un nombre que quiera usar para el grupo de recursos. Puede reemplazar **eastus** por una región cercana si es necesario.

    ```bash
    az group create --name myResourceGroup --location eastus
    ```

1. Muchos de los comandos necesitan nombres únicos y usan los mismos parámetros. La creación de algunas variables reducirá los cambios necesarios para los comandos que crean recursos. Ejecute los comandos siguientes para crear las variables necesarias. Reemplace **myResourceGroup** por el nombre que vaya a usar para este ejercicio. Si ha cambiado la ubicación en el paso anterior, realice el mismo cambio en la variable **location**.

    ```bash
    let rNum=$RANDOM
    resourceGroup=myResourceGroup
    location=eastus
    topicName="mytopic-evgtopic-${rNum}"
    siteName="evgsite-${rNum}"
    siteURL="https://${siteName}.azurewebsites.net"
    ```

### Habilitar un proveedor de recursos de Event Grid

Un proveedor de recursos de Azure es un servicio que define y administra tipos específicos de recursos en Azure. Es lo que Azure usa en segundo plano al implementar o administrar recursos. Registre el proveedor de recursos de Event Grid con el comando **az provider register**. 

```bash
az provider register --namespace Microsoft.EventGrid
```

El registro puede tardar unos minutos en completarse. Puede comprobar el estado con el siguiente comando.

```bash
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

> **Nota:** Este paso solo es necesario en las suscripciones que no han usado Event Grid previamente.

### Creación de un tema en Event Grid

Cree un tema mediante el comando **az eventgrid topic create**. El nombre debe ser único porque forma parte de la entrada DNS.  

```bash
az eventgrid topic create --name $topicName \
    --location $location \
    --resource-group $resourceGroup
```

### Creación de un punto de conexión de mensaje

Antes de suscribirse al tema personalizado, es necesario crear el punto de conexión para el mensaje del evento. Normalmente, el punto de conexión realiza acciones en función de los datos del evento. El script siguiente usa una aplicación web precompilada que muestra los mensajes del evento. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

1. Ejecute los siguientes comandos para crear una punto de conexión de mensajes. El comando **echo** mostrará la dirección URL del sitio para el punto de conexión.

    ```bash
    az deployment group create \
        --resource-group $resourceGroup \
        --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/main/azuredeploy.json" \
        --parameters siteName=$siteName hostingPlanName=viewerhost
    
    echo "Your web app URL: ${siteURL}"
    ```

    > **Nota:** Este comando puede tardar varios minutos en completarse.

1. Abra una nueva pestaña del explorador y vaya a la dirección URL generada al final del script anterior para asegurarse de que la aplicación web se ejecuta. Debería ver el sitio, que no muestra ningún mensaje actualmente.

    > **Sugerencia:** Deje el explorador en ejecución, ya que se usa para mostrar las actualizaciones.

### Suscripción al tema

Suscríbase a un tema de Event Grid para indicarle los eventos de los que quiere realizar un seguimiento y el lugar al que deben enviarse dichos eventos. 

1. Suscríbase al tema de mediante el comando **az eventgrid event-subscription create**. El script siguiente recupera el id. de suscripción de la cuenta y lo usa en la creación de la suscripción al evento.

    ```bash
    endpoint="${siteURL}/api/updates"
    topicId=$(az eventgrid topic show --resource-group $resourceGroup \
        --name $topicName --query "id" --output tsv)
    
    az eventgrid event-subscription create \
        --source-resource-id $topicId \
        --name TopicSubscription \
        --endpoint $endpoint
    ```

1. Vuelva a la aplicación web y observe que se ha enviado un evento de validación de suscripción. Seleccione el icono del ojo para expandir los datos del evento. Event Grid envía el evento de validación para que el punto de conexión pueda verificar que desea recibir datos de eventos. La aplicación web incluye código para validar la suscripción.

## Envío de un evento con una aplicación de consola de .NET

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Siga estos pasos en Cloud Shell.

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. Ejecute los comandos siguientes para crear un directorio que contenga el proyecto y cambie al directorio del proyecto.

    ```bash
    mkdir eventgrid
    cd eventgrid
    ```

1. Cree la aplicación de consola .NET.

    ```bash
    dotnet new console
    ```

1. Ejecute los siguientes comandos para agregar los paquetes **Azure.Messaging.EventGrid** y **dotenv.net** al proyecto.

    ```bash
    dotnet add package Azure.Messaging.EventGrid
    dotnet add package dotenv.net
    ```

### Configuración de la aplicación de consola

En esta sección, recuperará el punto de conexión del tema y la clave de acceso para que se puedan agregar a un archivo **.env** para almacenar esos secretos.

1. Ejecute los siguientes comandos para recuperar la dirección URL y la clave de acceso del tema que ha creado antes. Asegúrese de registrar estos valores.

    ```bash
    az eventgrid topic show --name $topicName -g $resourceGroup --query "endpoint" --output tsv
    az eventgrid topic key list --name $topicName -g $resourceGroup --query "key1" --output tsv
    ```

1. Ejecute el comando siguiente para crear el archivo **.env** para contener los secretos y, después, ábralo en el editor de código.

    ```bash
    touch .env
    code .env
    ```

1. Agregue el siguiente código al archivo **.env**. Reemplace **YOUR_TOPIC_ENDPOINT** y **YOUR_TOPIC_ACCESS_KEY** por los valores que ha registrado antes.

    ```
    TOPIC_ENDPOINT="YOUR_TOPIC_ENDPOINT"
    TOPIC_ACCESS_KEY="YOUR_TOPIC_ACCESS_KEY"
    ```

1. Presione **Ctrl+S** para guardar el archivo y después **Ctr+Q** para salir del editor.

Ahora es el momento de reemplazar el código de plantilla en el archivo **Program.cs** mediante el editor de Cloud Shell.

### Adición del código al proyecto

1. Ejecuta el siguiente comando en Cloud Shell para empezar a editar la aplicación.

    ```bash
    code Program.cs
    ```

1. Reemplace el código existente por el siguiente. Asegúrese de revisar los comentarios en el código.

    ```csharp
    using dotenv.net; 
    using Azure.Messaging.EventGrid; 
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Start the asynchronous process to send an Event Grid event
    ProcessAsync().GetAwaiter().GetResult();
    
    async Task ProcessAsync()
    {
        // Retrieve Event Grid topic endpoint and access key from environment variables
        var topicEndpoint = envVars["TOPIC_ENDPOINT"];
        var topicKey = envVars["TOPIC_ACCESS_KEY"];
        
        // Check if the required environment variables are set
        if (string.IsNullOrEmpty(topicEndpoint) || string.IsNullOrEmpty(topicKey))
        {
            Console.WriteLine("Please set TOPIC_ENDPOINT and TOPIC_ACCESS_KEY in your .env file.");
            return;
        }
    
        // Create an EventGridPublisherClient to send events to the specified topic
        EventGridPublisherClient client = new EventGridPublisherClient
            (new Uri(topicEndpoint),
            new Azure.AzureKeyCredential(topicKey));
    
        // Create a new EventGridEvent with sample data
        var eventGridEvent = new EventGridEvent(
            subject: "ExampleSubject",
            eventType: "ExampleEventType",
            dataVersion: "1.0",
            data: new { Message = "Hello, Event Grid!" }
        );
    
        // Send the event to Azure Event Grid
        await client.SendEventAsync(eventGridEvent);
        Console.WriteLine("Event sent successfully.");
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

1. Ejecute el siguiente comando en Cloud Shell para iniciar la aplicación de consola. Verá el mensaje **Evento enviado correctamente.** cuando se envíe el mensaje.

    ```bash
    dotnet run
    ```

1. Vaya a la aplicación web para ver el evento que acaba de enviar. Seleccione el icono del ojo para expandir los datos del evento.

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, también se eliminarán los recursos existentes fuera del ámbito de este ejercicio.