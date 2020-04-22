<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="37154-101">En este ejercicio, ampliará la aplicación del ejercicio anterior para admitir la autenticación con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="37154-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="37154-102">Esto es necesario para obtener el token de acceso de OAuth necesario para llamar a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="37154-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="37154-103">Para ello, integrará la [biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="37154-103">To do this, you will integrate the [Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) into the application.</span></span>

1. <span data-ttu-id="37154-104">Cree un nuevo archivo de **lista de propiedades** en el proyecto **GraphTutorial** denominado **AuthSettings. plist**.</span><span class="sxs-lookup"><span data-stu-id="37154-104">Create a new **Property List** file in the **GraphTutorial** project named **AuthSettings.plist**.</span></span>
1. <span data-ttu-id="37154-105">Agregue los siguientes elementos al archivo en el diccionario **raíz** .</span><span class="sxs-lookup"><span data-stu-id="37154-105">Add the following items to the file in the **Root** dictionary.</span></span>

    | <span data-ttu-id="37154-106">Key </span><span class="sxs-lookup"><span data-stu-id="37154-106">Key</span></span> | <span data-ttu-id="37154-107">Tipo</span><span class="sxs-lookup"><span data-stu-id="37154-107">Type</span></span> | <span data-ttu-id="37154-108">Valor</span><span class="sxs-lookup"><span data-stu-id="37154-108">Value</span></span> |
    |-----|------|-------|
    | `AppId` | <span data-ttu-id="37154-109">Cadena</span><span class="sxs-lookup"><span data-stu-id="37154-109">String</span></span> | <span data-ttu-id="37154-110">El identificador de la aplicación del portal de Azure</span><span class="sxs-lookup"><span data-stu-id="37154-110">The application ID from the Azure portal</span></span> |
    | `GraphScopes` | <span data-ttu-id="37154-111">Matriz</span><span class="sxs-lookup"><span data-stu-id="37154-111">Array</span></span> | <span data-ttu-id="37154-112">Dos valores de cadena `User.Read` : y`Calendars.Read`</span><span class="sxs-lookup"><span data-stu-id="37154-112">Two String values: `User.Read` and `Calendars.Read`</span></span> |

    ![Captura de pantalla del archivo AuthSettings. plist en Xcode](./images/auth-settings.png)

> [!IMPORTANT]
> <span data-ttu-id="37154-114">Si usa un control de código fuente como GIT, ahora sería un buen momento para excluir el archivo **AuthSettings. plist** del control de código fuente para evitar la pérdida inadvertida del identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="37154-114">If you're using source control such as git, now would be a good time to exclude the **AuthSettings.plist** file from source control to avoid inadvertently leaking your app ID.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="37154-115">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="37154-115">Implement sign-in</span></span>

<span data-ttu-id="37154-116">En esta sección, configurará el proyecto para MSAL, creará una clase de administrador de autenticación y actualizará la aplicación para iniciar y cerrar sesión.</span><span class="sxs-lookup"><span data-stu-id="37154-116">In this section you will configure the project for MSAL, create an authentication manager class, and update the app to sign in and sign out.</span></span>

### <a name="configure-project-for-msal"></a><span data-ttu-id="37154-117">Configurar Project para MSAL</span><span class="sxs-lookup"><span data-stu-id="37154-117">Configure project for MSAL</span></span>

1. <span data-ttu-id="37154-118">Agregue un nuevo grupo de llaves a las capacidades del proyecto.</span><span class="sxs-lookup"><span data-stu-id="37154-118">Add a new keychain group to your project's capabilities.</span></span>
    1. <span data-ttu-id="37154-119">Seleccione el proyecto **GraphTutorial** y, a continuación, **firme funciones de &**.</span><span class="sxs-lookup"><span data-stu-id="37154-119">Select the **GraphTutorial** project, then **Signing & Capabilities**.</span></span>
    1. <span data-ttu-id="37154-120">Seleccione **+ capacidad**y, a continuación, haga doble clic en **uso compartido de llaves**.</span><span class="sxs-lookup"><span data-stu-id="37154-120">Select **+ Capability**, then double-click **Keychain Sharing**.</span></span>
    1. <span data-ttu-id="37154-121">Agregue un grupo de llaves con el `com.microsoft.adalcache`valor.</span><span class="sxs-lookup"><span data-stu-id="37154-121">Add a keychain group with the value `com.microsoft.adalcache`.</span></span>

1. <span data-ttu-id="37154-122">Control haga clic en **info. plist** y seleccione **Abrir como**y, a continuación, **código fuente**.</span><span class="sxs-lookup"><span data-stu-id="37154-122">Control click **Info.plist** and select **Open As**, then **Source Code**.</span></span>
1. <span data-ttu-id="37154-123">Agregue lo siguiente dentro del `<dict>` elemento.</span><span class="sxs-lookup"><span data-stu-id="37154-123">Add the following inside the `<dict>` element.</span></span>

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

1. <span data-ttu-id="37154-124">Abra **AppDelegate. SWIFT** y agregue la siguiente instrucción Import en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="37154-124">Open **AppDelegate.swift** and add the following import statement at the top of the file.</span></span>

    ```Swift
    import MSAL
    ```

1. <span data-ttu-id="37154-125">Agregue la siguiente función a la clase `AppDelegate`.</span><span class="sxs-lookup"><span data-stu-id="37154-125">Add the following function to the `AppDelegate` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AppDelegate.swift" id="HandleMsalResponseSnippet":::

### <a name="create-authentication-manager"></a><span data-ttu-id="37154-126">Crear el administrador de autenticación</span><span class="sxs-lookup"><span data-stu-id="37154-126">Create authentication manager</span></span>

1. <span data-ttu-id="37154-127">Cree un nuevo **archivo SWIFT** en el proyecto **GraphTutorial** denominado **AuthenticationManager. SWIFT**.</span><span class="sxs-lookup"><span data-stu-id="37154-127">Create a new **Swift File** in the **GraphTutorial** project named **AuthenticationManager.swift**.</span></span> <span data-ttu-id="37154-128">Agregue el siguiente código al archivo.</span><span class="sxs-lookup"><span data-stu-id="37154-128">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/AuthenticationManager.swift" id="AuthManagerSnippet":::

### <a name="add-sign-in-and-sign-out"></a><span data-ttu-id="37154-129">Agregar Inicio y cierre de sesión</span><span class="sxs-lookup"><span data-stu-id="37154-129">Add sign-in and sign-out</span></span>

1. <span data-ttu-id="37154-130">Abra **SignInViewController. SWIFT** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="37154-130">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SignInViewController.swift" id="SignInViewSnippet":::

1. <span data-ttu-id="37154-131">Abra **WelcomeViewController. SWIFT** y reemplace la función `signOut` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="37154-131">Open **WelcomeViewController.swift** and replace the existing `signOut` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="SignOutSnippet":::

1. <span data-ttu-id="37154-132">Guarde los cambios y reinicie la aplicación en el simulador.</span><span class="sxs-lookup"><span data-stu-id="37154-132">Save your changes and restart the application in Simulator.</span></span>

<span data-ttu-id="37154-133">Si inicia sesión en la aplicación, debería ver un token de acceso que se muestra en la ventana de salida de Xcode.</span><span class="sxs-lookup"><span data-stu-id="37154-133">If you sign in to the app, you should see an access token displayed in the output window in Xcode.</span></span>

![Captura de pantalla de la ventana de salida de Xcode que muestra un token de acceso](./images/access-token-output.png)

## <a name="get-user-details"></a><span data-ttu-id="37154-135">Obtener detalles del usuario</span><span class="sxs-lookup"><span data-stu-id="37154-135">Get user details</span></span>

<span data-ttu-id="37154-136">En esta sección, creará una clase auxiliar que contendrá todas las llamadas a Microsoft Graph y actualizará el `WelcomeViewController` para que use esta nueva clase para obtener el usuario que ha iniciado sesión.</span><span class="sxs-lookup"><span data-stu-id="37154-136">In this section you will create a helper class to hold all of the calls to Microsoft Graph and update the `WelcomeViewController` to use this new class to get the logged-in user.</span></span>

1. <span data-ttu-id="37154-137">Cree un nuevo **archivo SWIFT** en el proyecto **GraphTutorial** denominado **GraphManager. SWIFT**.</span><span class="sxs-lookup"><span data-stu-id="37154-137">Create a new **Swift File** in the **GraphTutorial** project named **GraphManager.swift**.</span></span> <span data-ttu-id="37154-138">Agregue el siguiente código al archivo.</span><span class="sxs-lookup"><span data-stu-id="37154-138">Add the following code to the file.</span></span>

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

1. <span data-ttu-id="37154-139">Abra **WelcomeViewController. SWIFT** y agregue la siguiente `import` instrucción en la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="37154-139">Open **WelcomeViewController.swift** and add the following `import` statement at the top of the file.</span></span>

    ```Swift
    import MSGraphClientModels
    ```

1. <span data-ttu-id="37154-140">Agregue la siguiente propiedad a la `WelcomeViewController` clase.</span><span class="sxs-lookup"><span data-stu-id="37154-140">Add the following property to the `WelcomeViewController` class.</span></span>

    ```Swift
    private let spinner = SpinnerViewController()
    ```

1. <span data-ttu-id="37154-141">Reemplace el existente `viewDidLoad` por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="37154-141">Replace the existing `viewDidLoad` with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/WelcomeViewController.swift" id="ViewDidLoadSnippet":::

<span data-ttu-id="37154-142">Si guarda los cambios y reinicia la aplicación ahora, después de iniciar sesión, la interfaz de usuario se actualizará con el nombre para mostrar y la dirección de correo electrónico del usuario.</span><span class="sxs-lookup"><span data-stu-id="37154-142">If you save your changes and restart the app now, after sign-in the UI is updated with the user's display name and email address.</span></span>
