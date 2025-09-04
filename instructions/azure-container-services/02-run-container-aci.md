---
lab:
  topic: Azure container services
  title: "Implementación de un contenedor en Azure\_Container Instances mediante comandos de la CLI de Azure"
  description: "Aprenda a usar comandos de la CLI de Azure para implementar un contenedor en Azure\_Container Instances."
---

# Implementación de un contenedor en Azure Container Instances mediante comandos de la CLI de Azure

En este ejercicio, implementará y ejecutará un contenedor en Azure Container Instances (ACI) con la CLI de Azure. Aprenderá a crear un grupo de contenedores, especificar la configuración del contenedor y comprobar que la aplicación contenedorizada se ejecuta en la nube.

Tareas realizadas en este ejercicio:

* Creación de recursos de Azure Container Instances en Azure
* Creación e implementación de un contenedor
* Comprobación de que el contenedor se está ejecutando

Este ejercicio se realiza en aproximadamente **15** minutos.

## Crear un grupo de recursos

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. Cree un grupo de recursos para los recursos necesarios para este ejercicio. Sustituya **myResourceGroup** por un nombre que quiera usar para el grupo de recursos. Puede reemplazar **eastus** por una región cercana si es necesario. Si ya tiene un grupo de recursos que quiere usar. continúe al paso siguiente.

    ```
    az group create --location eastus --name myResourceGroup
    ```

## Creación e implementación de un contenedor

Para crear un contenedor, debe especificar un nombre, una imagen de Docker y un grupo de recursos de Azure para el comando **az container create**. Expondrá el contenedor a Internet especificando una etiqueta de nombre DNS.

1. Ejecute el comando siguiente para crear un nombre DNS usado para exponer el contenedor a Internet. El nombre DNS debe ser único; ejecute este comando desde Cloud Shell para crear una variable que contenga un nombre único.

    ```bash
    DNS_NAME_LABEL=aci-example-$RANDOM
    ```

1. Ejecute el comando siguiente para crear una instancia de contenedor. Reemplace **myResourceGroup** y **myLocation** por los valores que ha usado antes. La operación tarda unos minutos en completarse.

    ```bash
    az container create --resource-group myResourceGroup \
        --name mycontainer \
        --image mcr.microsoft.com/azuredocs/aci-helloworld \
        --ports 80 \
        --dns-name-label $DNS_NAME_LABEL --location myLocation \
        --os-type Linux \
        --cpu 1 \
        --memory 1.5 
    ```

    En el comando anterior, **$DNS_NAME_LABEL** especifica el nombre DNS. El nombre de la imagen, **mcr.microsoft.com/azuredocs/aci-helloworld**, hace referencia a una imagen de Docker que ejecuta una aplicación web Node.js básica.

Vaya a la sección siguiente después de que finalice el comando **az container create**.

## Comprobación de que el contenedor se está ejecutando

Puede comprobar el estado de compilación del contenedor con el comando **az container show**. 

1. Ejecute el comando siguiente para comprobar el estado de aprovisionamiento del contenedor que ha creado. Reemplace **myResourceGroup** por el valor que ha usado antes.

    ```bash
    az container show --resource-group myResourceGroup \
        --name mycontainer \
        --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
        --out table 
    ```

    Verá el nombre de dominio completo (FQDN) del contenedor y su estado de aprovisionamiento. Este es un ejemplo.

    ```
    FQDN                                    ProvisioningState
    --------------------------------------  -------------------
    aci-wt.eastus.azurecontainer.io         Succeeded
    ```

    > **Nota:** Si el contenedor está en el estado **Creando**, espere unos instantes y vuelva a ejecutar el comando hasta que vea el estado **Correcto**.

1. Desde un explorador, vaya al FQDN del contenedor para ver su ejecución. Es posible que reciba una advertencia de que el sitio no es seguro.

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, también se eliminarán los recursos existentes fuera del ámbito de este ejercicio.
