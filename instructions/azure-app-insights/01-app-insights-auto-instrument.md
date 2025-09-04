---
lab:
  topic: Application Insights
  title: Supervisión de una aplicación con instrumentación automática
  description: "Aprenda a supervisar una aplicación en Application\_Insights sin modificar el código mediante la configuración de la instrumentación automática "
---

# Supervisión de una aplicación con instrumentación automática

En este ejercicio, creará una aplicación web de Azure App Service con Application Insights habilitado, configurará la instrumentación automática sin modificar el código, creará e implementará una aplicación Blazor y, después, verá las métricas de la aplicación y los datos de error en Application Insights. La implementación completa de la supervisión y observabilidad de aplicaciones, sin tener que realizar cambios en el código, simplifica las implementaciones y las migraciones.

Tareas realizadas en este ejercicio:

* Creación de un recurso de aplicación web con Application Insights habilitado
* Configure la instrumentación de la aplicación web.
* Cree una aplicación Blazor e impleméntela en el recurso de aplicación web.
* Visualización de la actividad de la aplicación en Application Insights
* Limpieza de recursos

Este ejercicio se realiza en aproximadamente **20** minutos.

## Creación de recursos en Azure

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.
1. Selecciona **+ Crear un recurso** ubicado en el encabezado **Servicios de Azure** situado cerca de la parte superior de la página principal. 
1. En la barra de búsqueda **Buscar en Marketplace**, escribe *aplicación web* y presiona **Entrar** para iniciar la búsqueda.
1. En el icono Aplicación web, selecciona la lista desplegable **Crear** y, a continuación, selecciona **Aplicación web**.

    ![Captura de pantalla del icono de la aplicación web.](./media/create-web-app-tile.png)

Al seleccionar **Crear**, se abrirá una plantilla con algunas pestañas para rellenar la información sobre la implementación. Los pasos siguientes te guiarán por los cambios que se deben realizar en las pestañas pertinentes.

1. Completa la pestaña **Datos básicos** con la información de la tabla siguiente:

    | Configuración | Acción |
    |--|--|
    | **Suscripción** | Conserve los valores predeterminados. |
    | **Grupo de recursos** | Selecciona Crear nuevo, escribe `rg-WebApp` y, a continuación, selecciona Aceptar. También puede seleccionar un grupo de recursos existente si lo prefiere. |
    | **Nombre** | Escriba un nombre único, por ejemplo **YOUR-INITIALS-monitorapp**. Reemplace **YOUR-INITIALS** por sus iniciales u otro valor. El nombre debe ser único, por lo que puede requerir algunos cambios. |
    | Control deslizante en la configuración de **Nombre** | Selecciona el control deslizante para desactivarlo. Este control deslizante solo aparece en algunas configuraciones de Azure. |
    | **Publicar** | Seleccione la opción **Código**. |
    | **Pila en tiempo de ejecución** | Seleccione **.NET 8 (LTS)** en el menú desplegable. |
    | **Sistema operativo** | Seleccione **Windows**. |
    | **Región** | Conserva la selección predeterminada o elige una región cercana. |
    | **Plan de Windows** | Conserva la selección predeterminada. |
    | **Plan de precios** | Selecciona la lista desplegable y elige el plan **F1 gratis**. |

1. Seleccione o vaya a la pestaña **Supervisar y proteger**, y escriba la información de la tabla siguiente:

    | Configuración | Acción |
    |--|--|
    | **Habilitar Application Insights** | Seleccione **Sí**. |
    | **Application Insights** | Seleccione **Crear nuevo** y aparecerá un cuadro de diálogo. Escriba `autoinstrument-insights` en el campo **Nombre** del cuadro de diálogo. Después, seleccione **Aceptar** para aceptar el nombre. |
    | **Área de trabajo** | Escriba `Workspace` si el campo aún no está rellenado y bloqueado. |

1. Seleccione **Revisar y crear** para revisar los detalles de la implementación. Después, seleccione **Create** para crear los recursos.

La implementación puede tardar unos minutos en completarse. Cuando termine, seleccione el botón **Ir al recurso**.

### Configuración de los valores de instrumentación

Para habilitar la supervisión sin cambios en el código, debe configurar la instrumentación de la aplicación en el nivel de servicio.

1. En el menú de navegación de la izquierda, expanda la sección **Supervisión** y seleccione **Información**.

1. Busque la sección **Instrumentar la aplicación** y seleccione **.NET Core**.

1. Seleccione **Recomendado** en la sección **Nivel de colección**.

1. Seleccione **Aplicar** y confirme los cambios.

1. En el menú de navegación de la izquierda, seleccione **Información general**.

## Creación e implementación de una aplicación Blazor

En esta sección del ejercicio se crea una aplicación Blazor en Cloud Shell y se implementa en la aplicación web que ha creado. Todos los pasos de esta sección se realizan en Cloud Shell.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Si se le pide que seleccione una cuenta de almacenamiento para conservar los archivos, seleccione **No se requiere ninguna cuenta de almacenamiento**, la suscripción y, después, seleccione **Aplicar**.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. Ejecute los comandos siguientes a fin de crear un directorio para la aplicación Blazor y cambie al directorio.

    ```
    mkdir blazor
    cd blazor
    ```

1. Ejecute el siguiente comando para crear una aplicación Blazor en la carpeta.

    ```
    dotnet new blazor
    ```

1. Ejecute el siguiente comando a fin de compilar la aplicación para asegurarse de que no ha habido ningún problema durante la creación.

    ```
    dotnet build
    ```

### Implementación de la aplicación en App Service

Para implementar la aplicación primero debe publicarla con el comando **dotnet publish** y, después, crear un archivo *.zip* para la implementación.

1. Ejecute el siguiente comando para publicar la aplicación en un directorio *publish*.

    ```
    dotnet publish -c Release -o ./publish
    ```

1. Ejecute los comandos siguientes para crear un archivo *.zip* de la aplicación publicada. El archivo *.zip* se ubicará en el directorio raíz de la aplicación.

    ```
    cd publish
    zip -r ../app.zip .
    cd ..
    ```

1. Ejecute el siguiente comando para implementar la aplicación en App Service. Reemplace **YOUR-WEB-APP-NAME** y **YOUR-RESOURCE-GROUP** por los valores que ha usado al crear los recursos de App Service anteriormente en el ejercicio.

    ```
    az webapp deploy --name YOUR-WEB-APP-NAME \
        --resource-group YOUR-RESOURCE-GROUP \
        --src-path ./app.zip
    ```

1. Una vez que se complete la implementación, seleccione el vínculo en el campo **Dominio predeterminado** ubicado en la sección **Essentials** para abrir la aplicación en una nueva pestaña del explorador.

Ahora es el momento de ver algunas métricas básicas de la aplicación en Application Insights. No cierre esta pestaña, la usará en el resto del ejercicio.

## Visualización de métricas en Application Insights

Vuelva a la pestaña con Azure Portal y vaya al recurso de Application Insights que ha creado antes. En la pestaña **Información general** se muestran algunos gráficos básicos:

* Error en las solicitudes
* Tiempo de respuesta del servidor
* Solicitudes de servidor
* Disponibilidad

En esta sección realizará algunas acciones en la aplicación web y, después, volverá a esta página para ver la actividad. El informe de actividades se retrasa, por lo que puede tardar unos minutos en aparecer en los gráficos.

Siga estos pasos en la aplicación web.

1. Navegue entre las opciones **Inicio**, **+ Contador** y **Tiempo** en el menú de la aplicación web.

1. Actualice la página web varias veces para generar datos de **Tiempo de respuesta del servidor** y **Solicitudes del servidor**.

1. Para crear algunos errores, seleccione el botón **Inicio** y, después, anexe **/failures** a la dirección URL. Esta ruta no existe en la aplicación web y generará un error. Actualiza la página varias veces para generar más errores.

1. Vuelva a la pestaña donde se ejecuta Application Insights y espere uno o dos minutos hasta que la información aparezca en los gráficos. 

1. En el panel de navegación de la izquierda, expanda la sección **Investigar** y seleccione **Errores**. Muestra el recuento de solicitudes con errores junto con información más detallada sobre los códigos de respuesta de los errores.

Explore otras opciones de creación de informes para hacerse una idea de qué otros tipos de información hay disponibles. 

## Limpieza de recursos

Ahora que has terminado el ejercicio, debes eliminar los recursos en la nube que has creado para evitar el uso innecesario de recursos.

1. Ve al grupo de recursos que creaste y visualiza el contenido de los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

> **PRECAUCIÓN:** Al eliminar un grupo de recursos se eliminan todos los recursos que contiene. Si ha elegido otro grupo de recursos para este ejercicio, también se eliminarán los recursos existentes fuera del ámbito de este ejercicio.
