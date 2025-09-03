---
lab:
  topic: Azure container services
  title: "Compilación y ejecución de una imagen de contenedor con Azure\_Container Registry Tasks"
  description: "Aprenda a usar comandos de la CLI de Azure para crear y ejecutar imágenes de contenedor con Azure\_Container Registry Tasks."
---

# Compilación y ejecución de una imagen de contenedor con Azure Container Registry Tasks

En este ejercicio, creará una imagen de contenedor a partir del código de la aplicación y la insertará en Azure Container Registry mediante la CLI de Azure. Aprenderá a preparar la aplicación para la contenedorización, a crear una instancia de ACR y a almacenar la imagen de contenedor en Azure.

Tareas realizadas en este ejercicio:

* Creación de un recurso de Azure Container Registry
* Creación e inserción de una imagen desde un Dockerfile
* Verificación de los resultados
* Ejecución de la imagen en Azure Container Registry

Este ejercicio se realiza en aproximadamente **20** minutos.

## Creación de un recurso de Azure Container Registry

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. Cree un grupo de recursos para los recursos necesarios para este ejercicio. Sustituya **myResourceGroup** por un nombre que quiera usar para el grupo de recursos. Puede reemplazar **eastus** por una región cercana si es necesario. Si ya tiene un grupo de recursos que quiere usar. continúe al paso siguiente.

    ```
    az group create --location eastus --name myResourceGroup
    ```

1. Ejecute el siguiente comando para crear un registro de contenedor básico. El nombre del registro debe ser único dentro de Azure y contener entre 5 y 50 caracteres alfanuméricos. Reemplace **myResourceGroup** por el nombre que ha usado antes y **myContainerRegistry** por un valor único.

    ```bash
    az acr create --resource-group myResourceGroup \
        --name myContainerRegistry --sku Basic
    ```

    > **Nota:** El comando crea un registro *Básico*, una opción de costo optimizado para los desarrolladores que están aprendiendo a usar Azure Container Registry.

## Creación e inserción de una imagen desde un Dockerfile

A continuación, compilará e insertará una imagen basada en un Dockerfile.

1. Ejecute el siguiente comando para crear el Dockerfile. El Dockerfile contiene una sola línea que hace referencia a la imagen *hello-world* hospedada en Microsoft Container Registry.

    ```bash
    echo FROM mcr.microsoft.com/hello-world > Dockerfile
    ```

1. Ejecute el comando **az acr build** siguiente, que crea la imagen y, una vez que la imagen se ha creado correctamente, la inserta en el registro. Reemplace **myContainerRegistry** por el nombre que ha creado antes.

    ```bash
    az acr build --image sample/hello-world:v1  \
        --registry myContainerRegistry \
        --file Dockerfile .
    ```

    A continuación se muestra un ejemplo abreviado de la salida del comando anterior que muestra las últimas líneas con los resultados finales. En el campo *repository* puede ver que aparece la imagen *sample/hello-word*.

    ```
    - image:
        registry: myContainerRegistry.azurecr.io
        repository: sample/hello-world
        tag: v1
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      runtime-dependency:
        registry: mcr.microsoft.com
        repository: hello-world
        tag: latest
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      git: {}
    
    
    Run ID: cf1 was successful after 11s
    ```

## Verificación de los resultados

1. Ejecute el siguiente comando para enumerar los repositorios en el registro. Reemplace **myContainerRegistry** por el nombre que ha creado antes.

    ```bash
    az acr repository list --name myContainerRegistry --output table
    ```

    Salida:

    ```
    Result
    ----------------
    sample/hello-world
    ```

1. Ejecute el comando siguiente para enumerar las etiquetas en el repositorio **sample/hello-world**. Reemplace **myContainerRegistry** por el nombre que ha usado antes.

    ```bash
    az acr repository show-tags --name myContainerRegistry \
        --repository sample/hello-world --output table
    ```

    Salida:

    ```
    Result
    --------
    v1
    ```

## Ejecución de la imagen en ACR

1. Ejecute la imagen de contenedor *sample/hello-world:v1* desde el registro de contenedor mediante el comando **az acr run**. En el ejemplo siguiente se utiliza **$Registry** para especificar el registro en que se ejecuta el comando. Reemplace **myContainerRegistry** por el nombre que ha usado antes.

    ```bash
    az acr run --registry myContainerRegistry \
        --cmd '$Registry/sample/hello-world:v1' /dev/null
    ```

    El parámetro **cmd** de este ejemplo ejecuta el contenedor en su configuración predeterminada, pero **cmd** admite otros parámetros **docker run**, o incluso otros comandos **docker**. 

    La salida de ejemplo siguiente se abrevia:

    ```
    Packing source code into tar to upload...
    Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
    Sending context (1.855 KiB) to registry: mycontainerre...
    Queued a run with ID: cab
    Waiting for an agent...
    2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
    2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
    2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
    2019/03/19 19:01:53 Setting up Docker configuration...
    2019/03/19 19:01:54 Successfully set up Docker configuration
    2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
    2019/03/19 19:01:55 Launching container with name: acb_step_0
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    2019/03/19 19:01:56 Successfully executed container: acb_step_0
    2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)
    
    Run ID: cab was successful after 6s
    ```

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, también se eliminarán los recursos existentes fuera del ámbito de este ejercicio.
