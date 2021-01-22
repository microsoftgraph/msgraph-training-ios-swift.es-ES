<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="06304-101">Comience por crear un nuevo proyecto swift.</span><span class="sxs-lookup"><span data-stu-id="06304-101">Begin by creating a new Swift project.</span></span>

1. <span data-ttu-id="06304-102">Abra Xcode.</span><span class="sxs-lookup"><span data-stu-id="06304-102">Open Xcode.</span></span> <span data-ttu-id="06304-103">En el **menú** Archivo, seleccione **Nuevo** y, a **continuación, Proyecto**.</span><span class="sxs-lookup"><span data-stu-id="06304-103">On the **File** menu, select **New**, then **Project**.</span></span>
1. <span data-ttu-id="06304-104">Elija la **plantilla aplicación** y seleccione **Siguiente**.</span><span class="sxs-lookup"><span data-stu-id="06304-104">Choose the **App** template and select **Next**.</span></span>

    ![Captura de pantalla del cuadro de diálogo de selección de plantilla Xcode](images/xcode-select-template.png)

1. <span data-ttu-id="06304-106">Establezca el **nombre del producto** en y el `GraphTutorial` **idioma** en **Swift**.</span><span class="sxs-lookup"><span data-stu-id="06304-106">Set the **Product Name** to `GraphTutorial` and the **Language** to **Swift**.</span></span>
1. <span data-ttu-id="06304-107">Rellene los campos restantes y seleccione **Siguiente**.</span><span class="sxs-lookup"><span data-stu-id="06304-107">Fill in the remaining fields and select **Next**.</span></span>
1. <span data-ttu-id="06304-108">Elija una ubicación para el proyecto y seleccione **Crear**.</span><span class="sxs-lookup"><span data-stu-id="06304-108">Choose a location for the project and select **Create**.</span></span>

## <a name="install-dependencies"></a><span data-ttu-id="06304-109">Instalar dependencias</span><span class="sxs-lookup"><span data-stu-id="06304-109">Install dependencies</span></span>

<span data-ttu-id="06304-110">Antes de seguir, instale algunas dependencias adicionales que usará más adelante.</span><span class="sxs-lookup"><span data-stu-id="06304-110">Before moving on, install some additional dependencies that you will use later.</span></span>

- <span data-ttu-id="06304-111">[Biblioteca de autenticación de Microsoft (MSAL) para iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) para autenticarse con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="06304-111">[Microsoft Authentication Library (MSAL) for iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) for authenticating to with Azure AD.</span></span>
- <span data-ttu-id="06304-112">[SDK de Microsoft Graph para objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="06304-112">[Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="06304-113">[SDK de modelos de Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-objc-models) para Objective C para objetos fuertemente tipos que representan recursos de Microsoft Graph como usuarios o eventos.</span><span class="sxs-lookup"><span data-stu-id="06304-113">[Microsoft Graph Models SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc-models) for strongly-typed objects representing Microsoft Graph resources like users or events.</span></span>

1. <span data-ttu-id="06304-114">Salga de Xcode.</span><span class="sxs-lookup"><span data-stu-id="06304-114">Quit Xcode.</span></span>
1. <span data-ttu-id="06304-115">Abra Terminal y cambie el directorio a la ubicación del **proyecto graphTutorial.**</span><span class="sxs-lookup"><span data-stu-id="06304-115">Open Terminal and change the directory to the location of your **GraphTutorial** project.</span></span>
1. <span data-ttu-id="06304-116">Ejecute el siguiente comando para crear un Podfile.</span><span class="sxs-lookup"><span data-stu-id="06304-116">Run the following command to create a Podfile.</span></span>

    ```Shell
    pod init
    ```

1. <span data-ttu-id="06304-117">Abre el Podfile y agrega las siguientes líneas justo después de la `use_frameworks!` línea.</span><span class="sxs-lookup"><span data-stu-id="06304-117">Open the Podfile and add the following lines just after the `use_frameworks!` line.</span></span>

    ```Ruby
    pod 'MSAL', '~> 1.1.13'
    pod 'MSGraphClientSDK', ' ~> 1.0.0'
    pod 'MSGraphClientModels', '~> 1.3.0'
    ```

1. <span data-ttu-id="06304-118">Guarde el Podfile y, a continuación, ejecute el siguiente comando para instalar las dependencias.</span><span class="sxs-lookup"><span data-stu-id="06304-118">Save the Podfile, then run the following command to install the dependencies.</span></span>

    ```Shell
    pod install
    ```

1. <span data-ttu-id="06304-119">Una vez completado el comando, abra el **graphTutorial.xcworkspace** recién creado en Xcode.</span><span class="sxs-lookup"><span data-stu-id="06304-119">Once the command completes, open the newly created **GraphTutorial.xcworkspace** in Xcode.</span></span>

## <a name="design-the-app"></a><span data-ttu-id="06304-120">Diseñar la aplicación</span><span class="sxs-lookup"><span data-stu-id="06304-120">Design the app</span></span>

<span data-ttu-id="06304-121">En esta sección creará las vistas para la aplicación: una página de inicio de sesión, un navegador de barras de pestañas, una página de bienvenida y una página de calendario.</span><span class="sxs-lookup"><span data-stu-id="06304-121">In this section you will create the views for the app: a sign in page, a tab bar navigator, a welcome page, and a calendar page.</span></span> <span data-ttu-id="06304-122">También crearás una superposición de indicador de actividad.</span><span class="sxs-lookup"><span data-stu-id="06304-122">You'll also create an activity indicator overlay.</span></span>

### <a name="create-sign-in-page"></a><span data-ttu-id="06304-123">Crear página de inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="06304-123">Create sign in page</span></span>

1. <span data-ttu-id="06304-124">Expanda la **carpeta GraphTutorial** en Xcode y, a continuación, **seleccione ViewController.swift**.</span><span class="sxs-lookup"><span data-stu-id="06304-124">Expand the **GraphTutorial** folder in Xcode, then select **ViewController.swift**.</span></span>
1. <span data-ttu-id="06304-125">En el **Inspector de** archivos, cambie el **nombre** del archivo a `SignInViewController.swift` .</span><span class="sxs-lookup"><span data-stu-id="06304-125">In the **File Inspector**, change the **Name** of the file to `SignInViewController.swift`.</span></span>

    ![Captura de pantalla del Inspector de archivos](images/rename-file.png)

1. <span data-ttu-id="06304-127">Abra **SignInViewController.swift** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="06304-127">Open **SignInViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class SignInViewController: UIViewController {

        override func viewDidLoad() {
            super.viewDidLoad()
            // Do any additional setup after loading the view.
        }

        @IBAction func signIn() {
            self.performSegue(withIdentifier: "userSignedIn", sender: nil)
        }
    }
    ```

1. <span data-ttu-id="06304-128">Abra **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="06304-128">Open **Main.storyboard**.</span></span> <span data-ttu-id="06304-129">Expanda **Ver escena del controlador** y, a continuación, seleccione Ver **controlador.**</span><span class="sxs-lookup"><span data-stu-id="06304-129">Expand **View Controller Scene**, then select **View Controller**.</span></span>

    ![Captura de pantalla de Xcode con el controlador de vista seleccionado](images/storyboard-select-view-controller.png)

1. <span data-ttu-id="06304-131">Seleccione el **inspector de identidad** y, a continuación, cambie el desplegable **de** clases a **SignInViewController**.</span><span class="sxs-lookup"><span data-stu-id="06304-131">Select the **Identity Inspector**, then change the **Class** dropdown to **SignInViewController**.</span></span>

    ![Captura de pantalla del inspector de identidad](images/change-class.png)

1. <span data-ttu-id="06304-133">Seleccione la **biblioteca y,** a continuación, arrastre un **botón** al controlador **de vista de inicio de sesión.**</span><span class="sxs-lookup"><span data-stu-id="06304-133">Select the **Library**, then drag a **Button** onto the **Sign In View Controller**.</span></span>

    ![Captura de pantalla de la biblioteca en Xcode](images/add-button-to-view.png)

1. <span data-ttu-id="06304-135">Con el botón seleccionado, seleccione el **Inspector de atributos** y cambie el **título** del botón a `Sign In` .</span><span class="sxs-lookup"><span data-stu-id="06304-135">With the button selected, select the **Attributes Inspector** and change the **Title** of the button to `Sign In`.</span></span>

    ![Captura de pantalla del campo Título en el Inspector de atributos en Xcode](images/set-button-title.png)

1. <span data-ttu-id="06304-137">Con el botón seleccionado, selecciona el botón **Alinear** en la parte inferior del guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="06304-137">With the button selected, select the **Align** button at the bottom of the storyboard.</span></span> <span data-ttu-id="06304-138">Seleccione horizontalmente en **contenedor y** **verticalmente** en restricciones de contenedor, deje sus valores como 0 y, a continuación, seleccione **Agregar 2 restricciones**.</span><span class="sxs-lookup"><span data-stu-id="06304-138">Select both the **Horizontally in container** and **Vertically in container** constraints, leave their values as 0, then select **Add 2 constraints**.</span></span>

    ![Captura de pantalla de la configuración de restricciones de alineación en Xcode](images/add-alignment-constraints.png)

1. <span data-ttu-id="06304-140">Seleccione el **controlador de vista de inicio de sesión** y, a continuación, seleccione el Inspector de **conexiones**.</span><span class="sxs-lookup"><span data-stu-id="06304-140">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="06304-141">En **Acciones recibidas,** arrastre el círculo sin rellenar junto a **signIn** en el botón.</span><span class="sxs-lookup"><span data-stu-id="06304-141">Under **Received Actions**, drag the unfilled circle next to **signIn** onto the button.</span></span> <span data-ttu-id="06304-142">Selecciona **Tocar dentro en** el menú emergente.</span><span class="sxs-lookup"><span data-stu-id="06304-142">Select **Touch Up Inside** on the pop-up menu.</span></span>

    ![Captura de pantalla de arrastrar la acción signIn al botón en Xcode](images/connect-sign-in-button.png)

### <a name="create-tab-bar"></a><span data-ttu-id="06304-144">Crear barra de pestañas</span><span class="sxs-lookup"><span data-stu-id="06304-144">Create tab bar</span></span>

1. <span data-ttu-id="06304-145">Selecciona la **biblioteca y,** a continuación, arrastra un controlador **de barra de pestañas** al guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="06304-145">Select the **Library**, then drag a **Tab Bar Controller** onto the storyboard.</span></span>
1. <span data-ttu-id="06304-146">Seleccione el **controlador de vista de inicio de sesión** y, a continuación, seleccione el Inspector de **conexiones**.</span><span class="sxs-lookup"><span data-stu-id="06304-146">Select the **Sign In View Controller**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="06304-147">En **Segues desencadenados,** arrastra el círculo sin rellenar junto a **manual** en el controlador de la barra **de pestañas** en el guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="06304-147">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Tab Bar Controller** on the storyboard.</span></span> <span data-ttu-id="06304-148">Seleccione **Presentar modalmente** en el menú emergente.</span><span class="sxs-lookup"><span data-stu-id="06304-148">Select **Present Modally** in the pop-up menu.</span></span>

    ![Captura de pantalla de arrastrar un segue manual al nuevo controlador de barra de pestañas en Xcode](images/add-segue.png)

1. <span data-ttu-id="06304-150">Seleccione el segue que acaba de agregar y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="06304-150">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="06304-151">Establezca el **campo Identificador** en y `userSignedIn` establezca **Presentación** en **Pantalla completa.**</span><span class="sxs-lookup"><span data-stu-id="06304-151">Set the **Identifier** field to `userSignedIn`, and set **Presentation** to **Full Screen**.</span></span>

    ![Captura de pantalla del campo Identificador en el Inspector de atributos en Xcode](images/set-segue-identifier.png)

1. <span data-ttu-id="06304-153">Seleccione la **escena elemento 1** y, a continuación, seleccione el **Inspector de conexiones**.</span><span class="sxs-lookup"><span data-stu-id="06304-153">Select the **Item 1 Scene**, then select the **Connections Inspector**.</span></span>
1. <span data-ttu-id="06304-154">En **Segues desencadenado,** arrastra el círculo sin rellenar junto a **manual** en el controlador de **vista de** inicio de sesión en el guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="06304-154">Under **Triggered Segues**, drag the unfilled circle next to **manual** onto the **Sign In View Controller** on the storyboard.</span></span> <span data-ttu-id="06304-155">Seleccione **Presentar modalmente** en el menú emergente.</span><span class="sxs-lookup"><span data-stu-id="06304-155">Select **Present Modally** in the pop-up menu.</span></span>
1. <span data-ttu-id="06304-156">Seleccione el segue que acaba de agregar y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="06304-156">Select the segue you just added, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="06304-157">Establezca el **campo Identificador** en y `userSignedOut` establezca **Presentación** en **Pantalla completa.**</span><span class="sxs-lookup"><span data-stu-id="06304-157">Set the **Identifier** field to `userSignedOut`, and set **Presentation** to **Full Screen**.</span></span>

### <a name="create-welcome-page"></a><span data-ttu-id="06304-158">Crear página principal</span><span class="sxs-lookup"><span data-stu-id="06304-158">Create welcome page</span></span>

1. <span data-ttu-id="06304-159">Selecciona el **archivo Assets.xcassets.**</span><span class="sxs-lookup"><span data-stu-id="06304-159">Select the **Assets.xcassets** file.</span></span>
1. <span data-ttu-id="06304-160">En el **menú Editor,** seleccione **Agregar nuevo activo** y, a continuación, Conjunto de **imágenes.**</span><span class="sxs-lookup"><span data-stu-id="06304-160">On the **Editor** menu, select **Add New Asset**, then **Image Set**.</span></span>
1. <span data-ttu-id="06304-161">Seleccione el nuevo **activo de** imagen y use el **Inspector de** atributos para establecer su **nombre** en `DefaultUserPhoto` .</span><span class="sxs-lookup"><span data-stu-id="06304-161">Select the new **Image** asset and use the **Attribute Inspector** to set its **Name** to `DefaultUserPhoto`.</span></span>
1. <span data-ttu-id="06304-162">Agrega cualquier imagen que quieras que sirva como foto de perfil de usuario predeterminada.</span><span class="sxs-lookup"><span data-stu-id="06304-162">Add any image you like to serve as a default user profile photo.</span></span>

    ![Captura de pantalla de la vista de activos del conjunto de imágenes en Xcode](images/add-default-user-photo.png)

1. <span data-ttu-id="06304-164">Cree un nuevo **archivo de clase Cocoa Touch** en la carpeta **GraphTutorial** denominada `WelcomeViewController` .</span><span class="sxs-lookup"><span data-stu-id="06304-164">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `WelcomeViewController`.</span></span> <span data-ttu-id="06304-165">Elija **UIViewController** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="06304-165">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="06304-166">Abra **WelcomeViewController.swift** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="06304-166">Open **WelcomeViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class WelcomeViewController: UIViewController {

        @IBOutlet var userProfilePhoto: UIImageView!
        @IBOutlet var userDisplayName: UILabel!
        @IBOutlet var userEmail: UILabel!

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.

            // TEMPORARY
            self.userProfilePhoto.image = UIImage(imageLiteralResourceName: "DefaultUserPhoto")
            self.userDisplayName.text = "Default User"
            self.userEmail.text = "default@contoso.com"
        }

        @IBAction func signOut() {
            self.performSegue(withIdentifier: "userSignedOut", sender: nil)
        }
    }
    ```

1. <span data-ttu-id="06304-167">Abra **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="06304-167">Open **Main.storyboard**.</span></span> <span data-ttu-id="06304-168">Seleccione la **escena elemento 1** y, a continuación, seleccione el **inspector de identidad**.</span><span class="sxs-lookup"><span data-stu-id="06304-168">Select the **Item 1 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="06304-169">Cambie el **valor de** clase a **WelcomeViewController**.</span><span class="sxs-lookup"><span data-stu-id="06304-169">Change the **Class** value to **WelcomeViewController**.</span></span>
1. <span data-ttu-id="06304-170">Mediante la **biblioteca**, agregue los siguientes elementos a la **escena de elemento 1**.</span><span class="sxs-lookup"><span data-stu-id="06304-170">Using the **Library**, add the following items to the **Item 1 Scene**.</span></span>

    - <span data-ttu-id="06304-171">Una **vista de imagen**</span><span class="sxs-lookup"><span data-stu-id="06304-171">One **Image View**</span></span>
    - <span data-ttu-id="06304-172">Dos **etiquetas**</span><span class="sxs-lookup"><span data-stu-id="06304-172">Two **Labels**</span></span>
    - <span data-ttu-id="06304-173">Un **botón**</span><span class="sxs-lookup"><span data-stu-id="06304-173">One **Button**</span></span>
1. <span data-ttu-id="06304-174">Con el **Inspector de** conexiones, realice las siguientes conexiones.</span><span class="sxs-lookup"><span data-stu-id="06304-174">Using the **Connections Inspector**, make the following connections.</span></span>

    - <span data-ttu-id="06304-175">Vincula la **salida userDisplayName** a la primera etiqueta.</span><span class="sxs-lookup"><span data-stu-id="06304-175">Link the **userDisplayName** outlet to the first label.</span></span>
    - <span data-ttu-id="06304-176">Vincular la **salida userEmail** a la segunda etiqueta.</span><span class="sxs-lookup"><span data-stu-id="06304-176">Link the **userEmail** outlet to the second label.</span></span>
    - <span data-ttu-id="06304-177">Vincular la **salida userProfilePhoto** a la vista de imagen.</span><span class="sxs-lookup"><span data-stu-id="06304-177">Link the **userProfilePhoto** outlet to the image view.</span></span>
    - <span data-ttu-id="06304-178">Vincular la **acción de inicio de** sesión recibido a la pantalla Táctil dentro del **botón.**</span><span class="sxs-lookup"><span data-stu-id="06304-178">Link the **signOut** received action to the button's **Touch Up Inside**.</span></span>

1. <span data-ttu-id="06304-179">Seleccione la vista de imagen y, a continuación, seleccione **el Inspector de tamaño**.</span><span class="sxs-lookup"><span data-stu-id="06304-179">Select the image view, then select the **Size Inspector**.</span></span>
1. <span data-ttu-id="06304-180">Establezca el **ancho** y **el alto** en 196.</span><span class="sxs-lookup"><span data-stu-id="06304-180">Set the **Width** and **Height** to 196.</span></span>
1. <span data-ttu-id="06304-181">Usa el **botón Alinear** para agregar la restricción **Horizontal en** contenedor con un valor de 0.</span><span class="sxs-lookup"><span data-stu-id="06304-181">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="06304-182">Usa el **botón Agregar nuevas restricciones** (junto **al** botón Alinear) para agregar las siguientes restricciones:</span><span class="sxs-lookup"><span data-stu-id="06304-182">Use the **Add New Constraints** button (next to the **Align** button) to add the following constraints:</span></span>

    - <span data-ttu-id="06304-183">Alinear arriba a: área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="06304-183">Align Top to: Safe Area, value: 0</span></span>
    - <span data-ttu-id="06304-184">Espacio inferior para: Nombre para mostrar del usuario, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="06304-184">Bottom Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="06304-185">Alto, valor: 196</span><span class="sxs-lookup"><span data-stu-id="06304-185">Height, value: 196</span></span>
    - <span data-ttu-id="06304-186">Ancho, valor: 196</span><span class="sxs-lookup"><span data-stu-id="06304-186">Width, value: 196</span></span>

    ![Captura de pantalla de la nueva configuración de restricciones en Xcode](./images/add-new-constraints.png)

1. <span data-ttu-id="06304-188">Selecciona la primera etiqueta y, a continuación, usa el botón **Alinear** para agregar horizontalmente la restricción **de** contenedor con un valor de 0.</span><span class="sxs-lookup"><span data-stu-id="06304-188">Select the first label, then use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="06304-189">Use el **botón Agregar nuevas restricciones** para agregar las siguientes restricciones:</span><span class="sxs-lookup"><span data-stu-id="06304-189">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="06304-190">Espacio superior para: Foto de perfil de usuario, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="06304-190">Top Space to: User Profile Photo, value: Standard</span></span>
    - <span data-ttu-id="06304-191">Espacio inferior a: Correo electrónico del usuario, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="06304-191">Bottom Space to: User Email, value: Standard</span></span>

1. <span data-ttu-id="06304-192">Seleccione la segunda etiqueta y, a continuación, seleccione **el Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="06304-192">Select the second label, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="06304-193">Cambie el **color a** gris **oscuro y** cambie **la** fuente al **sistema 12.0**.</span><span class="sxs-lookup"><span data-stu-id="06304-193">Change the **Color** to **Dark Gray Color**, and change the **Font** to **System 12.0**.</span></span>
1. <span data-ttu-id="06304-194">Usa el **botón Alinear** para agregar la restricción **Horizontal en** contenedor con un valor de 0.</span><span class="sxs-lookup"><span data-stu-id="06304-194">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="06304-195">Use el **botón Agregar nuevas restricciones** para agregar las siguientes restricciones:</span><span class="sxs-lookup"><span data-stu-id="06304-195">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="06304-196">Espacio superior para: Nombre para mostrar del usuario, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="06304-196">Top Space to: User Display Name, value: Standard</span></span>
    - <span data-ttu-id="06304-197">Espacio inferior para: Cerrar sesión, valor: 14</span><span class="sxs-lookup"><span data-stu-id="06304-197">Bottom Space to: Sign Out, value: 14</span></span>

1. <span data-ttu-id="06304-198">Seleccione el botón y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="06304-198">Select the button, then select the **Attributes Inspector**.</span></span>
1. <span data-ttu-id="06304-199">Cambie el **título** a `Sign Out` .</span><span class="sxs-lookup"><span data-stu-id="06304-199">Change the **Title** to `Sign Out`.</span></span>
1. <span data-ttu-id="06304-200">Usa el **botón Alinear** para agregar la restricción **Horizontal en** contenedor con un valor de 0.</span><span class="sxs-lookup"><span data-stu-id="06304-200">Use the **Align** button to add the **Horizontally in container** constraint with a value of 0.</span></span>
1. <span data-ttu-id="06304-201">Use el **botón Agregar nuevas restricciones** para agregar las siguientes restricciones:</span><span class="sxs-lookup"><span data-stu-id="06304-201">Use the **Add New Constraints** button to add the following constraints:</span></span>

    - <span data-ttu-id="06304-202">Espacio superior a: Correo electrónico del usuario, valor: 14</span><span class="sxs-lookup"><span data-stu-id="06304-202">Top Space to: User Email, value: 14</span></span>

1. <span data-ttu-id="06304-203">Seleccione el elemento de la barra de pestañas en la parte inferior de la escena y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="06304-203">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="06304-204">Cambie el **título** a `Me` .</span><span class="sxs-lookup"><span data-stu-id="06304-204">Change the **Title** to `Me`.</span></span>

<span data-ttu-id="06304-205">La escena de bienvenida debería ser similar a esta una vez que termines.</span><span class="sxs-lookup"><span data-stu-id="06304-205">The welcome scene should look similar to this once you're done.</span></span>

![Captura de pantalla del diseño de la escena de bienvenida](images/welcome-scene-layout.png)

### <a name="create-calendar-page"></a><span data-ttu-id="06304-207">Crear página de calendario</span><span class="sxs-lookup"><span data-stu-id="06304-207">Create calendar page</span></span>

1. <span data-ttu-id="06304-208">Cree un nuevo **archivo de clase Cocoa Touch** en la carpeta **GraphTutorial** denominada `CalendarViewController` .</span><span class="sxs-lookup"><span data-stu-id="06304-208">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `CalendarViewController`.</span></span> <span data-ttu-id="06304-209">Elija **UIViewController** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="06304-209">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="06304-210">Abra **CalendarViewController.swift** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="06304-210">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit

    class CalendarViewController: UIViewController {

        @IBOutlet var calendarJSON: UITextView!

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.

            // TEMPORARY
            calendarJSON.text = "Calendar"
            calendarJSON.sizeToFit()
        }
    }
    ```

1. <span data-ttu-id="06304-211">Abra **Main.storyboard**.</span><span class="sxs-lookup"><span data-stu-id="06304-211">Open **Main.storyboard**.</span></span> <span data-ttu-id="06304-212">Seleccione la **escena del elemento 2** y, a continuación, seleccione el **inspector de identidad**.</span><span class="sxs-lookup"><span data-stu-id="06304-212">Select the **Item 2 Scene**, then select the **Identity Inspector**.</span></span> <span data-ttu-id="06304-213">Cambie el **valor de** clase a **CalendarViewController**.</span><span class="sxs-lookup"><span data-stu-id="06304-213">Change the **Class** value to **CalendarViewController**.</span></span>
1. <span data-ttu-id="06304-214">Mediante la **biblioteca**, agregue una **vista de texto** a la escena de elemento **2**.</span><span class="sxs-lookup"><span data-stu-id="06304-214">Using the **Library**, add a **Text View** to the **Item 2 Scene**.</span></span>
1. <span data-ttu-id="06304-215">Seleccione la vista de texto que acaba de agregar.</span><span class="sxs-lookup"><span data-stu-id="06304-215">Select the text view you just added.</span></span> <span data-ttu-id="06304-216">En el **menú Editor,** elija **Insertar en** y, a continuación, Vista **de desplazamiento.**</span><span class="sxs-lookup"><span data-stu-id="06304-216">On the **Editor** menu, choose **Embed In**, then **Scroll View**.</span></span>
1. <span data-ttu-id="06304-217">Cambia el tamaño de la vista de desplazamiento y la vista de texto para que ocupa toda la pantalla.</span><span class="sxs-lookup"><span data-stu-id="06304-217">Resize the scroll view and text view to take up the entire screen.</span></span>
1. <span data-ttu-id="06304-218">Mediante el **Inspector de conexiones,** conecte **la salida calendarJSON** a la vista de texto.</span><span class="sxs-lookup"><span data-stu-id="06304-218">Using the **Connections Inspector**, connect the **calendarJSON** outlet to the text view.</span></span>
1. <span data-ttu-id="06304-219">Seleccione el elemento de la barra de pestañas en la parte inferior de la escena y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="06304-219">Select the tab bar item at the bottom of the scene, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="06304-220">Cambie el **título** a `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="06304-220">Change the **Title** to `Calendar`.</span></span>
1. <span data-ttu-id="06304-221">En el **menú Editor,** seleccione Resolver problemas **de** diseño automático y, a continuación, seleccione Agregar **restricciones** que faltan debajo de Todas las **vistas en el controlador de vista de calendario.**</span><span class="sxs-lookup"><span data-stu-id="06304-221">On the **Editor** menu, select **Resolve Auto Layout Issues**, then select **Add Missing Constraints** underneath **All Views in Calendar View Controller**.</span></span>

<span data-ttu-id="06304-222">La escena del calendario debería ser similar a esta una vez que termines.</span><span class="sxs-lookup"><span data-stu-id="06304-222">The calendar scene should look similar to this once you're done.</span></span>

![Captura de pantalla del diseño de la escena calendario](images/calendar-scene-layout.png)

### <a name="create-activity-indicator"></a><span data-ttu-id="06304-224">Crear indicador de actividad</span><span class="sxs-lookup"><span data-stu-id="06304-224">Create activity indicator</span></span>

1. <span data-ttu-id="06304-225">Cree un nuevo **archivo de clase Cocoa Touch** en la carpeta **GraphTutorial** denominada `SpinnerViewController` .</span><span class="sxs-lookup"><span data-stu-id="06304-225">Create a new **Cocoa Touch Class** file in the **GraphTutorial** folder named `SpinnerViewController`.</span></span> <span data-ttu-id="06304-226">Elija **UIViewController** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="06304-226">Choose **UIViewController** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="06304-227">Abre **SpinnerViewController.swift** y reemplaza su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="06304-227">Open **SpinnerViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/SpinnerViewController.swift" id="SpinnerSnippet":::

## <a name="test-the-app"></a><span data-ttu-id="06304-228">Probar la aplicación</span><span class="sxs-lookup"><span data-stu-id="06304-228">Test the app</span></span>

<span data-ttu-id="06304-229">Guarda los cambios e inicia la aplicación.</span><span class="sxs-lookup"><span data-stu-id="06304-229">Save your changes and launch the app.</span></span> <span data-ttu-id="06304-230">Debería poder moverse entre las pantallas con los botones Iniciar y cerrar **sesión** y la barra de pestañas. </span><span class="sxs-lookup"><span data-stu-id="06304-230">You should be able to move between the screens using the **Sign In** and **Sign Out** buttons and the tab bar.</span></span>

![Capturas de pantalla de la aplicación](images/app-screens.png)
