<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f9643-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="f9643-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="f9643-102">Esto es necesario para obtener el token de acceso OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="f9643-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="f9643-103">Para ello, integrará la Biblioteca de autenticación de [Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="f9643-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="f9643-104">Cree un nuevo archivo **de lista de** propiedades en el proyecto **GraphTutorial** denominado **AuthSettings.plist**.</span><span class="sxs-lookup"><span data-stu-id="f9643-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="f9643-105">Agregue los siguientes elementos al archivo en el **diccionario** raíz.</span><span class="sxs-lookup"><span data-stu-id="f9643-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="f9643-106">Key </span><span class="sxs-lookup"><span data-stu-id="f9643-106">Key</span></span> | <span data-ttu-id="f9643-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="f9643-107">Type</span></span> | <span data-ttu-id="f9643-108">Valor</span><span class="sxs-lookup"><span data-stu-id="f9643-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="f9643-109">Cadena</span><span class="sxs-lookup"><span data-stu-id="f9643-109">String</span></span> | <span data-ttu-id="f9643-110">El id. de aplicación de Azure Portal</span><span class="sxs-lookup"><span data-stu-id="f9643-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="f9643-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="f9643-111">Array</span></span> | <span data-ttu-id="f9643-112">Tres valores de cadena: `User.Read` `MailboxSettings.Read` , y `Calendars.ReadWrite`</span><span class="sxs-lookup"><span data-stu-id="f9643-112">Three String values: `User.Read`, `MailboxSettings.Read`, and `Calendars.ReadWrite`</span></span> |

    ![Captura de pantalla del archivo AuthSettings.plist en Xcode](images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="f9643-114">Si usas el control de código fuente como Git, ahora sería un buen momento para excluir el archivo **AuthSettings.plist** del control de código fuente para evitar la pérdida involuntaria del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="f9643-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="f9643-115">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="f9643-115">Implement sign-in</span></span>

<span data-ttu-id="f9643-116">En esta sección configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="f9643-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="f9643-117">Configurar el proyecto para MSAL</span><span class="sxs-lookup"><span data-stu-id="f9643-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="f9643-118">Agregue un nuevo grupo de llaves a las capacidades del proyecto.</span><span class="sxs-lookup"><span data-stu-id="f9643-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="f9643-119">Seleccione el **proyecto GraphTutorial** y, a **continuación, & funcionalidades.**</span><span class="sxs-lookup"><span data-stu-id="f9643-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="f9643-120">Seleccione **+ Capacidad y, a** continuación, haga doble clic en Compartir **cadenas de claves.**</span><span class="sxs-lookup"><span data-stu-id="f9643-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="f9643-121">Agregue un grupo de llaves con el valor `com.microsoft.adalcache` .</span><span class="sxs-lookup"><span data-stu-id="f9643-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="f9643-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span><span class="sxs-lookup"><span data-stu-id="f9643-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="f9643-123">Agregue lo siguiente dentro del `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="f9643-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="f9643-124">Abra **AppDelegate.swift y** agregue la siguiente instrucción import en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="f9643-124">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="f9643-125">Agregue la siguiente función a la clase `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="f9643-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="f9643-126">Crear administrador de autenticación</span><span class="sxs-lookup"><span data-stu-id="f9643-126">Create authentication manager</span></span>

1. <span data-ttu-id="f9643-127">Cree un nuevo **archivo Swift** en el **proyecto GraphTutorial** denominado **AuthenticationManager.swift**.</span><span class="sxs-lookup"><span data-stu-id="f9643-127">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="f9643-128">Agregue el siguiente código al archivo.</span><span class="sxs-lookup"><span data-stu-id="f9643-128">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="f9643-129">Agregar inicio y cerrar sesión</span><span class="sxs-lookup"><span data-stu-id="f9643-129">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="f9643-130">Abra **SignInViewController.swift** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="f9643-130">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. <span data-ttu-id="f9643-131">Abra **WelcomeViewController.swift** y reemplace la función `signOut` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="f9643-131">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. <span data-ttu-id="f9643-132">Guarda los cambios y reinicia la aplicación en simulador.</span><span class="sxs-lookup"><span data-stu-id="f9643-132">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="f9643-133">Si inicias sesión en la aplicación, deberías ver un token de acceso en la ventana de salida en Xcode.</span><span class="sxs-lookup"><span data-stu-id="f9643-133">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Captura de pantalla de la ventana de salida en Xcode que muestra un token de acceso](images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="f9643-135">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="f9643-135">Get user details</span></span>

<span data-ttu-id="f9643-136">En esta sección creará una clase auxiliar para retener todas las llamadas a Microsoft Graph y actualizará el uso de esta nueva clase para obtener el usuario que ha `WelcomeViewController` iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="f9643-136">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="f9643-137">Cree un nuevo **archivo Swift** en el **proyecto GraphTutorial** denominado **GraphManager.swift**.</span><span class="sxs-lookup"><span data-stu-id="f9643-137">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="f9643-138">Agregue el siguiente código al archivo.</span><span class="sxs-lookup"><span data-stu-id="f9643-138">Add the following code to the file.</span></span>

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

1. <span data-ttu-id="f9643-139">Abra **WelcomeViewController.swift** y agregue la siguiente `import` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="f9643-139">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="f9643-140">Agregue la siguiente propiedad a la `WelcomeViewController` clase.</span><span class="sxs-lookup"><span data-stu-id="f9643-140">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="f9643-141">Reemplace el existente `viewDidLoad` por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="f9643-141">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

<span data-ttu-id="f9643-142">Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualiza con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="f9643-142">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
