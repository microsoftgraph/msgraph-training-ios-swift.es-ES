<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD. Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph. Para ello, integrará la [biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.

1. Cree un nuevo archivo de **lista de propiedades** en el proyecto **GraphTutorial** denominado **AuthSettings. plist**.
1. Agregue los siguientes elementos al archivo en el diccionario **raíz** .

    | Key  | Tipo | Valor |
    |-----|------|-------|
    | `AppId` | Cadena | El identificador de la aplicación del portal de Azure |
    | `GraphScopes` | Matriz | Dos valores de cadena `User.Read` : y`Calendars.Read` |

    ![Captura de pantalla del archivo AuthSettings. plist en Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el archivo **AuthSettings. plist** del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

En esta sección, configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.

### <a name="configure-project-for-msal"></a>Configurar Project para MSAL

1. Agregue un nuevo grupo de llaves a las capacidades del proyecto.
    1. Seleccione el proyecto **GraphTutorial** y, a continuación, **firme funciones de &**.
    1. Seleccione **+ capacidad**y, a continuación, haga doble clic en **uso compartido de llaves**.
    1. Agregue un grupo de llaves con el `com.microsoft.adalcache`valor.

1. Control haga clic en **info. plist** y seleccione **Abrir como**y, a continuación, **código fuente**.
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

1. Abra **AppDelegate. SWIFT** y agregue la siguiente instrucción Import en la parte superior del archivo.

    ```Swift
    import MSAL
    ```

1. Agregue la siguiente función a la clase `AppDelegate`.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a>Crear el administrador de autenticación

1. Cree un nuevo **archivo SWIFT** en el proyecto **GraphTutorial** denominado **AuthenticationManager. SWIFT**. Agregue el siguiente código al archivo.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a>Agregar Inicio y cierre de sesión

1. Abra **SignInViewController. SWIFT** y reemplace su contenido por el código siguiente.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. Abra **WelcomeViewController. SWIFT** y reemplace la función `signOut` existente por lo siguiente.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. Guarde los cambios y reinicie la aplicación en el simulador.

Si inicia sesión en la aplicación, debería ver un token de acceso que se muestra en la ventana de salida de Xcode.

![Captura de pantalla de la ventana de salida de Xcode que muestra un token de acceso](./images/access-token-output.png)

## <a name="get-user-details"></a>Obtener detalles del usuario

En esta sección, creará una clase auxiliar que contendrá todas las llamadas a Microsoft Graph y actualizará el `WelcomeViewController` para que use esta nueva clase para obtener el usuario que ha iniciado sesión.

1. Cree un nuevo **archivo SWIFT** en el proyecto **GraphTutorial** denominado **GraphManager. SWIFT**. Agregue el siguiente código al archivo.

    ```Swift
    import Foundation
    import MSGraphClientSDK
    import MSGraphClientModels

    class GraphManager {

        // Implement singleton pattern
        static let instance = GraphManager()

        private let client: MSHTTPClient?

        private init() {
            client = MSClientFactory.createHTTPClient(with: AuthenticationManager.instance)
        }

        public func getMe(completion: @escaping(MSGraphUser?, Error?) -> Void) {
            // GET /me
            let meRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me")!)
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

1. Abra **WelcomeViewController. SWIFT** y agregue la siguiente `import` instrucción en la parte superior del archivo.

    ```Swift
    import MSGraphClientModels
    ```

1. Agregue la siguiente propiedad a la `WelcomeViewController` clase.

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. Reemplace el existente `viewDidLoad` por el código siguiente.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.
