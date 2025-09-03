---
lab:
  topic: Azure container services
  title: "Implementación de un contenedor en Azure\_Container Apps con la CLI de Azure"
  description: "Aprenda a usar comandos de la CLI de Azure para crear un entorno seguro de Azure\_Container Apps e implementar un contenedor."
---

# Implementación de un contenedor en Azure Container Apps con la CLI de Azure

En este ejercicio, implementará una aplicación contenedorizada en Azure Container Apps mediante la CLI de Azure. Aprenderá a crear un entorno de aplicación de contenedor, a implementar el contenedor y a comprobar que la aplicación se ejecuta en Azure.

Tareas realizadas en este ejercicio:

* Creación de recursos en Azure
* Crear un entorno de Azure Container Apps
* Implementación de una aplicación de contenedor en el entorno

Este ejercicio se realiza en aproximadamente **15** minutos.

## Creación de un grupo de recursos y preparación del entorno de Azure

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. Cree un grupo de recursos para los recursos necesarios para este ejercicio. Sustituya **myResourceGroup** por un nombre que quiera usar para el grupo de recursos. Puede reemplazar **eastus** por una región cercana si es necesario. Si ya tiene un grupo de recursos que quiere usar. continúe al paso siguiente.

    ```azurecli
    az group create --location eastus --name myResourceGroup
    ```

1. Ejecute los siguientes comandos para asegurarse de que tiene la versión más reciente de la extensión Azure Container Apps instalada para la CLI.

    ```azurecli
    az extension add --name containerapp --upgrade
    ```

### Registro de espacios de nombres

Se deben registrar dos espacios de nombres para Azure Container Apps y asegurarse de que están registrados en los pasos siguientes. Cada registro puede tardar unos minutos en completarse si aún no están configurados en la suscripción. 

1. Registre el espacio de nombres **Microsoft.App**. 

    ```bash
    az provider register --namespace Microsoft.App
    ```

1. Registre el proveedor **Microsoft.OperationalInsights** para el área de trabajo de Log Analytics de Azure Monitor si no lo ha usado antes.

    ```bash
    az provider register --namespace Microsoft.OperationalInsights
    ```

## Crear un entorno de Azure Container Apps

Un entorno de Azure Container Apps crea un límite seguro alrededor de un grupo de aplicaciones de contenedor. Las aplicaciones de contenedor implementadas en el mismo entorno se implementan en la misma red virtual y escriben registros en la misma área de trabajo de Log Analytics.

1. Cree un entorno de con el comando **az containerapp env create**. Reemplace **myResourceGroup** y **myLocation** por los valores que ha usado antes. La operación tarda unos minutos en completarse.

    ```bash
    az containerapp env create \
        --name my-container-env \
        --resource-group myResourceGroup \
        --location myLocation
    ```

## Implementación de una aplicación de contenedor en el entorno

Una vez que el entorno de la aplicación contenedora finalice la implementación, puede implementar una imagen de contenedor en el entorno.

1. Implemente una imagen de contenedor de aplicación de ejemplo con el comando **containerapp create**. Reemplace **myResourceGroup** por el valor que ha usado antes.

    ```bash
    az containerapp create \
        --name my-container-app \
        --resource-group myResourceGroup \
        --environment my-container-env \
        --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
        --target-port 80 \
        --ingress 'external' \
        --query properties.configuration.ingress.fqdn
    ```

    Al establecer **--ingress** en **external**, la aplicación contenedora estará disponible para solicitudes públicas. El comando devuelve un vínculo para acceder a la aplicación.

    ```
    Container app created. Access your app at <url>
    ```

Para comprobar la implementación, seleccione la URL devuelta por el comando **az containerapp create** para comprobar que la aplicación contenedora está en ejecución.

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, también se eliminarán los recursos existentes fuera del ámbito de este ejercicio.
