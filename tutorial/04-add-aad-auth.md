<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="28201-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="28201-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="28201-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="28201-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="28201-103">Para ello, integrará la [biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="28201-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="28201-104">Cree un nuevo archivo de **lista de propiedades** en el proyecto **GraphTutorial** denominado **AuthSettings. plist**.</span><span class="sxs-lookup"><span data-stu-id="28201-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="28201-105">Agregue los siguientes elementos al archivo en el diccionario **raíz** .</span><span class="sxs-lookup"><span data-stu-id="28201-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="28201-106">Key </span><span class="sxs-lookup"><span data-stu-id="28201-106">Key</span></span> | <span data-ttu-id="28201-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="28201-107">Type</span></span> | <span data-ttu-id="28201-108">Valor</span><span class="sxs-lookup"><span data-stu-id="28201-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="28201-109">Cadena</span><span class="sxs-lookup"><span data-stu-id="28201-109">String</span></span> | <span data-ttu-id="28201-110">El identificador de la aplicación del portal de Azure</span><span class="sxs-lookup"><span data-stu-id="28201-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="28201-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="28201-111">Array</span></span> | <span data-ttu-id="28201-112">Dos valores de cadena `User.Read` : y`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="28201-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Captura de pantalla del archivo AuthSettings. plist en Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="28201-114">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el archivo **AuthSettings. plist** del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="28201-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="28201-115">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="28201-115">Implement sign-in</span></span>

<span data-ttu-id="28201-116">En esta sección, configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="28201-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="28201-117">Configurar Project para MSAL</span><span class="sxs-lookup"><span data-stu-id="28201-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="28201-118">Agregue un nuevo grupo de llaves a las capacidades del proyecto.</span><span class="sxs-lookup"><span data-stu-id="28201-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="28201-119">Seleccione el proyecto **GraphTutorial** y, a continuación, **firme funciones de &**.</span><span class="sxs-lookup"><span data-stu-id="28201-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="28201-120">Seleccione **+ capacidad**y, a continuación, haga doble clic en **uso compartido de llaves**.</span><span class="sxs-lookup"><span data-stu-id="28201-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="28201-121">Agregue un grupo de llaves con el `com.microsoft.adalcache`valor.</span><span class="sxs-lookup"><span data-stu-id="28201-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="28201-122">Control haga clic en **info. plist** y seleccione **Abrir como**y, a continuación, **código fuente**.</span><span class="sxs-lookup"><span data-stu-id="28201-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="28201-123">Agregue lo siguiente dentro del `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="28201-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="28201-124">Abra **AppDelegate. SWIFT** y agregue la siguiente instrucción Import en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="28201-124">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="28201-125">Agregue la siguiente función a la clase `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="28201-125">Add the following function to the `AppDelegate` class.</span></span>

    ```Swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
    ```

### <a name="create-authentication-manager"></a><span data-ttu-id="28201-126">Crear el administrador de autenticación</span><span class="sxs-lookup"><span data-stu-id="28201-126">Create authentication manager</span></span>

1. <span data-ttu-id="28201-127">Cree un nuevo **archivo SWIFT** en el proyecto **GraphTutorial** denominado **AuthenticationManager. SWIFT**.</span><span class="sxs-lookup"><span data-stu-id="28201-127">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="28201-128">Agregue el siguiente código al archivo.</span><span class="sxs-lookup"><span data-stu-id="28201-128">Add the following code to the file.</span></span>

    ```Swift
    import Foundation
    import MSAL
    import MSGraphClientSDK

    // Implement the MSAuthenticationProvider interface so
    // this class can be used as an auth provider for the Graph SDK
    class AuthenticationManager: NSObject, MSAuthenticationProvider {

        // Implement singleton pattern
        static let instance = AuthenticationManager()

        private let publicClient: MSALPublicClientApplication?
        private let appId: String
        private let graphScopes: Array<String>

        private override init() {
            // Get app ID and scopes from AuthSettings.plist
            let bundle = Bundle.main
            let authConfigPath = bundle.path(forResource: "AuthSettings", ofType: "plist")!
            let authConfig = NSDictionary(contentsOfFile: authConfigPath)!

            self.appId = authConfig["AppId"] as! String
            self.graphScopes = authConfig["GraphScopes"] as! Array<String>

            do {
                // Create the MSAL client
                try self.publicClient = MSALPublicClientApplication(clientId: self.appId)
            } catch {
                print("Error creating MSAL public client: \(error)")
                self.publicClient = nil
            }
        }

        // Required function for the MSAuthenticationProvider interface
        func getAccessToken(for authProviderOptions: MSAuthenticationProviderOptions!, andCompletion completion: ((String?, Error?) -> Void)!) {
            getTokenSilently(completion: completion)
        }

        public func getTokenInteractively(parentView: UIViewController, completion: @escaping(_ accessToken: String?, Error?) -> Void) {
            let webParameters = MSALWebviewParameters(parentViewController: parentView)
            let interactiveParameters = MSALInteractiveTokenParameters(scopes: self.graphScopes,
                                                                       webviewParameters: webParameters)

            // Call acquireToken to open a browser so the user can sign in
            publicClient?.acquireToken(with: interactiveParameters, completionBlock: {
                (result: MSALResult?, error: Error?) in
                guard let tokenResult = result, error == nil else {
                    print("Error getting token interactively: \(String(describing: error))")
                    completion(nil, error)
                    return
                }

                print("Got token interactively: \(tokenResult.accessToken)")
                completion(tokenResult.accessToken, nil)
            })
        }

        public func getTokenSilently(completion: @escaping(_ accessToken: String?, Error?) -> Void) {
            // Check if there is an account in the cache
            var userAccount: MSALAccount?

            do {
                userAccount = try publicClient?.allAccounts().first
            } catch {
                print("Error getting account: \(error)")
            }

            if (userAccount != nil) {
                // Attempt to get token silently
                let silentParameters = MSALSilentTokenParameters(scopes: self.graphScopes, account: userAccount!)
                publicClient?.acquireTokenSilent(with: silentParameters, completionBlock: {
                    (result: MSALResult?, error: Error?) in
                    guard let tokenResult = result, error == nil else {
                        print("Error getting token silently: \(String(describing: error))")
                        completion(nil, error)
                        return
                    }

                    print("Got token silently: \(tokenResult.accessToken)")
                    completion(tokenResult.accessToken, nil)
                })
            } else {
                print("No account in cache")
                completion(nil, NSError(domain: "AuthenticationManager",
                                        code: MSALError.interactionRequired.rawValue, userInfo: nil))
            }
        }

        public func signOut() -> Void {
            do {
                // Remove all accounts from the cache
                let accounts = try publicClient?.allAccounts()

                try accounts!.forEach({
                    (account: MSALAccount) in
                    try publicClient?.remove(account)
                })
            } catch {
                print("Sign out error: \(String(describing: error))")
            }
        }
    }
    ```

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="28201-129">Agregar Inicio y cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="28201-129">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="28201-130">Abra **SignInViewController. SWIFT** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="28201-130">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class SignInViewController: UIViewController {

        private let spinner = SpinnerViewController()

        override func viewDidLoad() {
            super.viewDidLoad()
            // Do any additional setup after loading the view.

            // See if a user is already signed in
            spinner.start(container: self)

            AuthenticationManager.instance.getTokenSilently {
                (token: String?, error: Error?) in

                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let _ = token, error == nil else {
                        // If there is no token or if there's an error,
                        // no user is signed in, so stay here
                        return
                    }

                    // Since we got a token, a user is signed in
                    // Go to welcome page
                    self.performSegue(withIdentifier: "userSignedIn", sender: nil)
                }
            }
        }

        @IBAction func signIn() {
            spinner.start(container: self)

            // Do an interactive sign in
            AuthenticationManager.instance.getTokenInteractively(parentView: self) {
                (token: String?, error: Error?) in

                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let _ = token, error == nil else {
                        // Show the error and stay on the sign-in page
                        let alert = UIAlertController(title: "Error signing in",
                                                      message: error.debugDescription,
                                                      preferredStyle: .alert)

                        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
                        self.present(alert, animated: true)
                        return
                    }

                    // Signed in successfully
                    // Go to welcome page
                    self.performSegue(withIdentifier: "userSignedIn", sender: nil)
                }
            }
        }
    }
    ```

1. <span data-ttu-id="28201-131">Abra **WelcomeViewController. SWIFT** y reemplace la función `signOut` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="28201-131">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    ```Swift
    @IBAction func signOut() {
        AuthenticationManager.instance.signOut()
        self.performSegue(withIdentifier: "userSignedOut", sender: nil)
    }
    ```

1. <span data-ttu-id="28201-132">Guarde los cambios y reinicie la aplicación en el simulador.</span><span class="sxs-lookup"><span data-stu-id="28201-132">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="28201-133">Si inicia sesión en la aplicación, debería ver un token de acceso que se muestra en la ventana de salida de Xcode.</span><span class="sxs-lookup"><span data-stu-id="28201-133">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Captura de pantalla de la ventana de salida de Xcode que muestra un token de acceso](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="28201-135">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="28201-135">Get user details</span></span>

<span data-ttu-id="28201-136">En esta sección, creará una clase auxiliar que contendrá todas las llamadas a Microsoft Graph y actualizará el `WelcomeViewController` para que use esta nueva clase para obtener el usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="28201-136">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="28201-137">Cree un nuevo **archivo SWIFT** en el proyecto **GraphTutorial** denominado **GraphManager. SWIFT**.</span><span class="sxs-lookup"><span data-stu-id="28201-137">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="28201-138">Agregue el siguiente código al archivo.</span><span class="sxs-lookup"><span data-stu-id="28201-138">Add the following code to the file.</span></span>

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

1. <span data-ttu-id="28201-139">Abra **WelcomeViewController. SWIFT** y agregue la siguiente `import` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="28201-139">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="28201-140">Agregue la siguiente propiedad a la `WelcomeViewController` clase.</span><span class="sxs-lookup"><span data-stu-id="28201-140">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="28201-141">Reemplace el existente `viewDidLoad` por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="28201-141">Replace the existing `viewDidLoad` with the following code.</span></span>

    ```Swift
    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.

        self.spinner.start(container: self)

        // Get the signed-in user
        self.userProfilePhoto.image = UIImage(imageLiteralResourceName: "DefaultUserPhoto")

        GraphManager.instance.getMe {
            (user: MSGraphUser?, error: Error?) in

            DispatchQueue.main.async {
                self.spinner.stop()

                guard let currentUser = user, error == nil else {
                    print("Error getting user: \(String(describing: error))")
                    return
                }

                // Set display name
                self.userDisplayName.text = currentUser.displayName ?? "Mysterious Stranger"
                self.userDisplayName.sizeToFit()

                // AAD users have email in the mail attribute
                // Personal accounts have email in the userPrincipalName attribute
                self.userEmail.text = currentUser.mail ?? currentUser.userPrincipalName ?? ""
                self.userEmail.sizeToFit()
            }
        }
    }
    ```

<span data-ttu-id="28201-142">Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="28201-142">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
