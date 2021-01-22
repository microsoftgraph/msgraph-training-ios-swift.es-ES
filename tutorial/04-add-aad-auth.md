<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso OAuth necesario para llamar a Microsoft Graph. Para ello, integrará la Biblioteca de autenticación de [Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.

1. Cree un nuevo archivo **de lista de** propiedades en el proyecto **GraphTutorial** denominado **AuthSettings.plist**.
1. Agregue los siguientes elementos al archivo en el **diccionario** raíz.

    | Key  | Tipo | Valor |
    |-----|------|-------|
    | `AppId` | Cadena | El id. de aplicación de Azure Portal |
    | `GraphScopes` | Matriz | Tres valores de cadena: `User.Read` `MailboxSettings.Read` , y `Calendars.ReadWrite` |

    ![Captura de pantalla del archivo AuthSettings.plist en Xcode](images/auth-settings.png)

> [!IMPORTANT]
> Si usas el control de código fuente como Git, ahora sería un buen momento para excluir el archivo **AuthSettings.plist** del control de código fuente para evitar la pérdida involuntaria del identificador de la aplicación.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

En esta sección configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.

### <a name="configure-project-for-msal"></a>Configurar el proyecto para MSAL

1. Agregue un nuevo grupo de llaves a las capacidades del proyecto.
    1. Seleccione el **proyecto GraphTutorial** y, a **continuación, & funcionalidades.**
    1. Seleccione **+ Capacidad y, a** continuación, haga doble clic en Compartir **cadenas de claves.**
    1. Agregue un grupo de llaves con el valor `com.microsoft.adalcache` .

1. Control click **Info.plist** and select **Open As**, then **Source Code**.
1. Agregue lo siguiente dentro del `<dict>` elemento.

    ```xml
    <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.$(PRODUCT_BUNDLE_IDENTIFIER)</string>
        </array>
      </dict>
    </array>
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>msauthv2</string>
        <string>msauthv3</string>
    </array>
    ```

1. Abra **AppDelegate.swift y** agregue la siguiente instrucción import en la parte superior del archivo.

    ```Swift
    import MSAL
    ```

1. Agregue la siguiente función a la clase `AppDelegate`.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Crear administrador de autenticación

1. Cree un nuevo **archivo Swift** en el **proyecto GraphTutorial** denominado **AuthenticationManager.swift**. Agregue el siguiente código al archivo.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Agregar inicio y cerrar sesión

1. Abra **SignInViewController.swift** y reemplace su contenido por el siguiente código.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. Abra **WelcomeViewController.swift** y reemplace la función `signOut` existente por lo siguiente.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. Guarda los cambios y reinicia la aplicación en simulador.

Si inicias sesión en la aplicación, deberías ver un token de acceso en la ventana de salida en Xcode.

![Captura de pantalla de la ventana de salida en Xcode que muestra un token de acceso](images/access-token-output.png)

## <a name="get-user-details"></a>Obtener detalles del usuario

En esta sección creará una clase auxiliar para retener todas las llamadas a Microsoft Graph y actualizará el uso de esta nueva clase para obtener el usuario que ha `WelcomeViewController` iniciado sesión.

1. Cree un nuevo **archivo Swift** en el **proyecto GraphTutorial** denominado **GraphManager.swift**. Agregue el siguiente código al archivo.

    ```Swift
    import Foundation
    import MSGraphClientSDK
    import MSGraphClientModels

    class GraphManager {

        // Implement singleton pattern
        static let instance = GraphManager()

        private let client: MSHTTPClient?

        public var userTimeZone: String

        private init() {
            client = MSClientFactory.createHTTPClient(with: AuthenticationManager.instance)
            userTimeZone = "UTC"
        }

        public func getMe(completion: @escaping(MSGraphUser?, Error?) -> Void) {
            // GET /me
            let select = "$select=displayName,mail,mailboxSettings,userPrincipalName"
            let meRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me?\(select)")!)
            let meDataTask = MSURLSessionDataTask(request: meRequest, client: self.client, completion: {
                (data: Data?, response: URLResponse?, graphError: Error?) in
                guard let meData = data, graphError == nil else {
                    completion(nil, graphError)
                    return
                }

                do {
                    // Deserialize response as a user
                    let user = try MSGraphUser(data: meData)
                    completion(user, nil)
                } catch {
                    completion(nil, error)
                }
            })

            // Execute the request
            meDataTask?.execute()
        }
    }
    ```

1. Abra **WelcomeViewController.swift** y agregue la siguiente `import` instrucción en la parte superior del archivo.

    ```Swift
    import MSGraphClientModels
    ```

1. Agregue la siguiente propiedad a la `WelcomeViewController` clase.

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. Reemplace el existente `viewDidLoad` por el código siguiente.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualiza con el nombre para mostrar y la dirección de correo electrónico del usuario.
