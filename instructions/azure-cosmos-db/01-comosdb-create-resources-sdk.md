---
lab:
  topic: Azure Cosmos DB
  title: "Creación de recursos en Azure\_Cosmos\_DB for NoSQL mediante .NET"
  description: Aprende a crear recursos de contenedor y base de datos en Azure Cosmos DB con el SDK de Microsoft .NET v3.
---

# Creación de recursos en Azure Cosmos DB for NoSQL mediante .NET

En este ejercicio, creará una cuenta de Azure Cosmos DB y compilará una aplicación de consola de .NET que use el SDK de Microsoft Azure Cosmos DB para crear una base de datos, un contenedor y un elemento de ejemplo. Aprenderá a configurar la autenticación, a realizar operaciones de base de datos mediante programación y a comprobar los resultados en Azure Portal.

Tareas realizadas en este ejercicio:

* Creación de una cuenta de Azure Cosmos DB
* Creación de una aplicación de consola que crea un contenedor, una base de datos y un elemento
* Ejecución de la aplicación de consola y comprobación de los resultados

Este ejercicio se realiza en aproximadamente **30** minutos.

## Creación de una cuenta de Azure Cosmos DB

En esta sección de ejercicio, creará un grupo de recursos y una cuenta de Azure Cosmos DB. También registrarás el punto de conexión y la clave de acceso de la cuenta.

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

1. Cree un grupo de recursos para los recursos necesarios para este ejercicio. Si ya tiene un grupo de recursos que quiere usar. continúe al paso siguiente. Sustituya **myResourceGroup** por un nombre que quiera usar para el grupo de recursos. Puede reemplazar **eastus** por una región cercana si es necesario.

    ```
    az group create --location eastus --name myResourceGroup
    ```

1. Muchos de los comandos necesitan nombres únicos y usan los mismos parámetros. La creación de algunas variables reducirá los cambios necesarios para los comandos que crean recursos. Ejecute los comandos siguientes para crear las variables necesarias. Reemplace **myResourceGroup** por el nombre que vaya a usar para este ejercicio.

    ```
    resourceGroup=myResourceGroup
    accountName=cosmosexercise$RANDOM
    ```

1. Ejecute los comandos siguientes para crear la cuenta de Azure Cosmos DB; el nombre de cada cuenta debe ser único. 

    ```
    az cosmosdb create --name $accountName \
        --resource-group $resourceGroup
    ```

1.  Ejecuta el siguiente comando para recuperar el **documentEndpoint** para la cuenta de Azure Cosmos DB. Registra el punto de conexión a partir de los resultados del comando; es necesario más adelante en el ejercicio.

    ```
    az cosmosdb show --name $accountName \
        --resource-group $resourceGroup \
        --query "documentEndpoint" --output tsv
    ```

1. Recupere la clave principal de la cuenta mediante el comando siguiente. Registre la clave principal a partir de los resultados del comando; la necesitará más adelante en el ejercicio.

    ```
    az cosmosdb keys list --name $accountName \
        --resource-group $resourceGroup \
        --query "primaryMasterKey" --output tsv
    ```

## Creación de recursos de datos y un elemento con una aplicación de consola de .NET

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Siga estos pasos en Cloud Shell.

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

1. Cree una carpeta para el proyecto y vaya a la carpeta.

    ```bash
    mkdir cosmosdb
    cd cosmosdb
    ```

1. Cree la aplicación de consola de .NET.

    ```bash
    dotnet new console
    ```

### Configuración de la aplicación de consola

1. Ejecute los siguientes comandos para agregar los paquetes **Microsoft.Azure.Cosmos**, **Newtonsoft.Json** y **dotenv.net** al proyecto.

    ```bash
    dotnet add package Microsoft.Azure.Cosmos --version 3.*
    dotnet add package Newtonsoft.Json --version 13.*
    dotnet add package dotenv.net
    ```

1. Ejecute el comando siguiente para crear el archivo **.env** para contener los secretos y, después, ábralo en el editor de código.

    ```bash
    touch .env
    code .env
    ```

1. Agregue el siguiente código al archivo **.env**. Reemplace **YOUR_DOCUMENT_ENDPOINT** y **YOUR_ACCOUNT_KEY** por los valores que ha registrado antes.

    ```
    DOCUMENT_ENDPOINT="YOUR_DOCUMENT_ENDPOINT"
    ACCOUNT_KEY="YOUR_ACCOUNT_KEY"
    ```

1. Presione **Ctrl+S** para guardar el archivo y después **Ctr+Q** para salir del editor.

Ahora es el momento de reemplazar el código de plantilla en el archivo **Program.cs** mediante el editor de Cloud Shell.

### Adición del código inicial para el proyecto

1. Ejecuta el siguiente comando en Cloud Shell para empezar a editar la aplicación.

    ```bash
    code Program.cs
    ```

1. Reemplace el código existente ahí por el siguiente fragmento de código. 

    El código proporciona la estructura general de la aplicación. Revise los comentarios del código para comprender cómo funciona. Para completar la aplicación, agregará código en áreas especificadas más adelante en el ejercicio. 

    ```csharp
    using Microsoft.Azure.Cosmos;
    using dotenv.net;
    
    string databaseName = "myDatabase"; // Name of the database to create or use
    string containerName = "myContainer"; // Name of the container to create or use
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    string cosmosDbAccountUrl = envVars["DOCUMENT_ENDPOINT"];
    string accountKey = envVars["ACCOUNT_KEY"];
    
    if (string.IsNullOrEmpty(cosmosDbAccountUrl) || string.IsNullOrEmpty(accountKey))
    {
        Console.WriteLine("Please set the DOCUMENT_ENDPOINT and ACCOUNT_KEY environment variables.");
        return;
    }
    
    // CREATE THE COSMOS DB CLIENT USING THE ACCOUNT URL AND KEY
    
    
    try
    {
        // CREATE A DATABASE IF IT DOESN'T ALREADY EXIST
    
    
        // CREATE A CONTAINER WITH A SPECIFIED PARTITION KEY
    
    
        // DEFINE A TYPED ITEM (PRODUCT) TO ADD TO THE CONTAINER
    
    
        // ADD THE ITEM TO THE CONTAINER
    
    
    }
    catch (CosmosException ex)
    {
        // Handle Cosmos DB-specific exceptions
        // Log the status code and error message for debugging
        Console.WriteLine($"Cosmos DB Error: {ex.StatusCode} - {ex.Message}");
    }
    catch (Exception ex)
    {
        // Handle general exceptions
        // Log the error message for debugging
        Console.WriteLine($"Error: {ex.Message}");
    }
    
    // This class represents a product in the Cosmos DB container
    public class Product
    {
        public string? id { get; set; }
        public string? name { get; set; }
        public string? description { get; set; }
    }
    ```

A continuación, agregará código en áreas especificadas de los proyectos para crear: el cliente, la base de datos y el contenedor, y agregará un elemento de ejemplo al contenedor.

### Adición de código para crear el cliente y realizar operaciones 

1. Agregue el código siguiente en el espacio después del comentario **// CREATE THE COSMOS DB CLIENT USING THE ACCOUNT URL AND KEY**. Este código define el cliente que se usa para conectarse a la cuenta de Azure Cosmos DB.

    ```csharp
    CosmosClient client = new(
        accountEndpoint: cosmosDbAccountUrl,
        authKeyOrResourceToken: accountKey
    );
    ```

    >Nota: el procedimiento recomendado es usar **DefaultAzureCredential** de la biblioteca de *identidad de Azure*. Esto puede necesitar algunos requisitos de configuración adicional en Azure en función de cómo esté configurada la suscripción. 

1. Agregue el código siguiente en el espacio después del comentario **// CREATE A DATABASE IF IT DOESN'T ALREADY EXIST**. 

    ```csharp
    Database database = await client.CreateDatabaseIfNotExistsAsync(databaseName);
    Console.WriteLine($"Created or retrieved database: {database.Id}");
    ```

1. Agregue el código siguiente en el espacio después del comentario **// CREATE A CONTAINER WITH A SPECIFIED PARTITION KEY**. 

    ```csharp
    Container container = await database.CreateContainerIfNotExistsAsync(
        id: containerName,
        partitionKeyPath: "/id"
    );
    Console.WriteLine($"Created or retrieved container: {container.Id}");
    ```

1. Agregue el código siguiente en el espacio después del comentario **// DEFINE A TYPED ITEM (PRODUCT) TO ADD TO THE CONTAINER**. Esto define el elemento que se agrega al contenedor.

    ```csharp
    Product newItem = new Product
    {
        id = Guid.NewGuid().ToString(), // Generate a unique ID for the product
        name = "Sample Item",
        description = "This is a sample item in my Azure Cosmos DB exercise."
    };
    ```

1. Agregue el código siguiente en el espacio después del comentario **// ADD THE ITEM TO THE CONTAINER**. 

    ```csharp
    ItemResponse<Product> createResponse = await container.CreateItemAsync(
        item: newItem,
        partitionKey: new PartitionKey(newItem.id)
    );

    Console.WriteLine($"Created item with ID: {createResponse.Resource.id}");
    Console.WriteLine($"Request charge: {createResponse.RequestCharge} RUs");
    ```

1. Ahora que el código está completo, guarde el progreso: presione **CTRL + S** para guardar el archivo y **CTRL + Q** para salir del editor.

1. Ejecuta el comando siguiente en Cloud Shell para probar si hay errores en el proyecto. Si ves errores, abre el archivo *Program.cs* en el editor y comprueba si falta código o errores de pegado.

    ```
    dotnet build
    ```

Ahora que el proyecto ha finalizado, es el momento de ejecutar la aplicación y comprobar los resultados en Azure Portal.

## Ejecución de la aplicación y comprobación de los resultados

1. Si estás en Cloud Shell, ejecuta el comando `dotnet run`. La salida debe reflejar algo parecido al siguiente ejemplo.

    ```
    Created or retrieved database: myDatabase
    Created or retrieved container: myContainer
    Created item: c549c3fa-054d-40db-a42b-c05deabbc4a6
    Request charge: 6.29 RUs
    ```

1. En Azure Portal, ve hasta el recurso de Azure Cosmos DB que has creado antes. En el panel de navegación izquierdo, selecciona **Explorador de datos**. En el **Explorador de datos**, selecciona **myDatabase** y expande **myContainer**. Para ver el elemento que creaste, selecciona **Elementos**.

    ![Captura de pantalla que muestra la ubicación de los elementos en el Explorador de datos.](./media/01/cosmos-data-explorer.png)

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, también se eliminarán los recursos existentes fuera del ámbito de este ejercicio.
