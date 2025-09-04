---
lab:
  topic: Azure authentication and authorization
  title: Implementación de la autenticación interactiva con MSAL.NET
  description: Obtenga información sobre cómo implementar la autenticación interactiva mediante el SDK de MSAL.NET y adquirir un token.
---

# Implementación de la autenticación interactiva con MSAL.NET

En este ejercicio, registrará una aplicación en Microsoft Entra ID y después creará una aplicación de consola de .NET que usa MSAL.NET para realizar la autenticación interactiva y adquirir un token de acceso para Microsoft Graph. Aprenderá a configurar ámbitos de autenticación, controlar el consentimiento del usuario y ver cómo se almacenan en caché los tokens para las ejecuciones posteriores. 

Tareas realizadas en este ejercicio:

* Registro de una aplicación en la plataforma de identidad de Microsoft
* Cree una aplicación de consola de .NET que implemente la clase **PublicClientApplicationBuilder** para configurar la autenticación.
* Adquiera un token de forma interactiva mediante el permiso **user.read** de Microsoft Graph.

Este ejercicio se realiza en aproximadamente **15** minutos.

## Antes de comenzar

Para completar el ejercicio, necesita lo siguiente:

* Suscripción a Azure. Si aún no tiene uno, puede [registrarse para obtenerlo](https://azure.microsoft.com/).

* [Visual Studio Code](https://code.visualstudio.com/) en una de las [plataformas admitidas](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

* [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) o posterior.

* [Kit de desarrollo de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) para Visual Studio Code.

## Registro de una nueva aplicación

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. En el portal, busque y seleccione **Registros de aplicaciones**. 

1. Seleccione **+ Nuevo registro** y, cuando aparezca la página **Registrar una aplicación**, escriba la información de registro de la aplicación:

    | Campo | Valor |
    |--|--|
    | **Nombre** | Escriba `myMsalApplication`.  |
    | **Tipos de cuenta admitidos** | Seleccione **Solo las cuentas de este directorio organizativo**. |
    | **URI de redireccionamiento (opcional)** | Seleccione **Cliente público o nativo (móvil y escritorio)** y escriba `http://localhost` en el cuadro de la derecha. |

1. Seleccione **Registrar**. Microsoft Entra ID le asigna un identificador de aplicación único (cliente) a la aplicación y le lleva a la página **Información general** de la aplicación. 

1. En la sección **Essentials** de la página **Información general**, registre el **Id. de aplicación (cliente)** y el **Id. de directorio (inquilino)**. La información es necesaria para la aplicación.

    ![Recorte de pantalla en el que se muestra la ubicación de los campos que se van a copiar.](./media/01-app-directory-id-location.png)
 
## Creación de una aplicación de consola de .NET para adquirir un token

Ahora que los recursos necesarios están implementados en Azure, el siguiente paso consiste en configurar la aplicación de consola. Los pasos siguientes se realizan en el entorno local.

1. Cree una carpeta denominada **authapp** o un nombre de su elección para el proyecto.

1. Inicie **Visual Studio Code**, seleccione **Archivo > Abrir carpeta...** y luego la carpeta del proyecto.

1. Seleccione **Ver > Terminal** para abrir un terminal.

1. Ejecute el siguiente comando en el terminal de VS Code para crear la aplicación de consola de .NET.

    ```
    dotnet new console
    ```

1. Ejecute los siguientes comandos para agregar los paquetes **Microsoft.Identity.Client** y **dotenv.net** al proyecto.

    ```
    dotnet add package Microsoft.Identity.Client
    dotnet add package dotenv.net
    ```

### Configuración de la aplicación de consola

En esta sección creará y editará un archivo **.env** para almacenar los secretos que ha registrado antes. 

1. Seleccione **Archivo > Nuevo archivo...** y cree un archivo denominado *.env* en la carpeta del proyecto.

1. Abra el archivo **.env** y agregue el código siguiente. Reemplace **YOUR_CLIENT_ID** y **YOUR_TENANT_ID** por los valores que ha registrado antes.

    ```
    CLIENT_ID="YOUR_CLIENT_ID"
    TENANT_ID="YOUR_TENANT_ID"
    ```

1. Presione **Ctrl+S** para guardar los cambios.

### Adición del código de inicio para el proyecto

1. Abra el archivo *Program.cs* y reemplace el contenido existente por el código siguiente. Asegúrese de revisar los comentarios en el código.

    ```csharp
    using Microsoft.Identity.Client;
    using dotenv.net;
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Retrieve Azure AD Application ID and tenant ID from environment variables
    string _clientId = envVars["CLIENT_ID"];
    string _tenantId = envVars["TENANT_ID"];
    
    // ADD CODE TO DEFINE SCOPES AND CREATE CLIENT 
    
    
    
    // ADD CODE TO ACQUIRE AN ACCESS TOKEN
    
    
    ```

1. Presione **Ctrl+S** para guardar los cambios.

### Adición de código para completar la aplicación

1. Busque el comentario **// ADD CODE TO DEFINE SCOPES AND CREATE CLIENT** y agregue el código siguiente directamente después. Asegúrese de revisar los comentarios en el código.

    ```csharp
    // Define the scopes required for authentication
    string[] _scopes = { "User.Read" };
    
    // Build the MSAL public client application with authority and redirect URI
    var app = PublicClientApplicationBuilder.Create(_clientId)
        .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
        .WithDefaultRedirectUri()
        .Build();
    ```

1. Busque el comentario **// ADD CODE TO ACQUIRE AN ACCESS TOKEN** y agregue el código siguiente directamente después. Asegúrese de revisar los comentarios en el código.

    ```csharp
    // Attempt to acquire an access token silently or interactively
    AuthenticationResult result;
    try
    {
        // Try to acquire token silently from cache for the first available account
        var accounts = await app.GetAccountsAsync();
        result = await app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
    }
    catch (MsalUiRequiredException)
    {
        // If silent token acquisition fails, prompt the user interactively
        result = await app.AcquireTokenInteractive(_scopes)
                    .ExecuteAsync();
    }
    
    // Output the acquired access token to the console
    Console.WriteLine($"Access Token:\n{result.AccessToken}");
    ```

1. Presione **Ctrl+S** para guardar el archivo y después **Ctr+Q** para salir del editor.

## Ejecución de la aplicación

Ahora que la aplicación está completa, llega el momento de ejecutarla. 

1. Para iniciar la aplicación, ejecute el siguiente comando:

    ```
    dotnet run
    ```

1. La aplicación abre el explorador predeterminado que le pide que seleccione la cuenta con la que quiere autenticarse. Si hay varias cuentas en la lista, seleccione la asociada al inquilino que se usa en la aplicación.

1. Si es la primera vez que se autentica en la aplicación registrada, recibirá una notificación de **Permisos solicitados** que le pide que apruebe que la aplicación inicie sesión y lea su perfil, y mantenga el acceso a los datos para los que le haya concedido acceso. Seleccione **Aceptar**.

    ![Recorte de pantalla en el que se muestra la notificación de permisos solicitados](./media/01-granting-permission.png)

1. Debería ver resultados similares al ejemplo siguiente en la consola.

    ```
    Access Token:
    eyJ0eXAiOiJKV1QiLCJub25jZSI6IlZF.........
    ```

1. Inicie la aplicación una segunda vez y observe que ya no recibe la notificación **Permisos solicitados**. El permiso que ha concedido antes se ha almacenado en caché.

## Limpieza de recursos

Ahora que ha terminado el ejercicio, debe eliminar el registro de la aplicación que ha creado antes.

1. En Azure Portal, vaya al registro de aplicaciones que ha creado.
1. En la barra de herramientas, seleccione **Eliminar**.
1. Confirme la eliminación.
