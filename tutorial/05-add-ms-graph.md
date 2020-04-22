<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="143cc-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="143cc-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="143cc-102">Para esta aplicación, usará el [SDK de Microsoft Graph para Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="143cc-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="143cc-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="143cc-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="143cc-104">En esta sección, ampliará la `GraphManager` clase para agregar una función para obtener los eventos del usuario y actualizar `CalendarViewController` para usar estas funciones nuevas.</span><span class="sxs-lookup"><span data-stu-id="143cc-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="143cc-105">Abra **GraphManager. SWIFT** y agregue el siguiente método a la `GraphManager` clase.</span><span class="sxs-lookup"><span data-stu-id="143cc-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

    ```Swift
    public func getEvents(completion: @escaping(Data?, Error?) -> Void) {
        // GET /me/events?$select='subject,organizer,start,end'$orderby=createdDateTime DESC
        // Only return these fields in results
        let select = "$select=subject,organizer,start,end"
        // Sort results by when they were created, newest first
        let orderBy = "$orderby=createdDateTime+DESC"
        let eventsRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me/events?\(select)&\(orderBy)")!)
        let eventsDataTask = MSURLSessionDataTask(request: eventsRequest, client: self.client, completion: {
            (data: Data?, response: URLResponse?, graphError: Error?) in
            guard let eventsData = data, graphError == nil else {
                completion(nil, graphError)
                return
            }

            // TEMPORARY
            completion(eventsData, nil)
        })

        // Execute the request
        eventsDataTask?.execute()
    }
    ```

    > [!NOTE]
    > <span data-ttu-id="143cc-106">Tenga en cuenta lo que `getEvents` hace el código.</span><span class="sxs-lookup"><span data-stu-id="143cc-106">Consider what the code in `getEvents` is doing.</span></span>
    >
    > - <span data-ttu-id="143cc-107">La dirección URL a la que se `/v1.0/me/events`llamará es.</span><span class="sxs-lookup"><span data-stu-id="143cc-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="143cc-108">El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la aplicación usará realmente.</span><span class="sxs-lookup"><span data-stu-id="143cc-108">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="143cc-109">El `orderby` parámetro de consulta ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="143cc-109">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="143cc-110">Abra **CalendarViewController. SWIFT** y reemplace todo su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="143cc-110">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

    ```Swift
    import UIKit
    import MSGraphClientModels

    class CalendarViewController: UIViewController {

        @IBOutlet var calendarJSON: UITextView!

        private let spinner = SpinnerViewController()

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
            self.spinner.start(container: self)

            GraphManager.instance.getEvents {
                (data: Data?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let eventsData = data, error == nil else {
                        self.calendarJSON.text = error.debugDescription
                        return
                    }

                    let jsonString = String(data: eventsData, encoding: .utf8)
                    self.calendarJSON.text = jsonString
                    self.calendarJSON.sizeToFit()
                }
            }
        }
    }
    ```

<span data-ttu-id="143cc-111">Ahora puede ejecutar la aplicación, iniciar sesión y puntear en el elemento de navegación **calendario** del menú.</span><span class="sxs-lookup"><span data-stu-id="143cc-111">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="143cc-112">Debería ver un volcado JSON de los eventos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="143cc-112">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="143cc-113">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="143cc-113">Display the results</span></span>

<span data-ttu-id="143cc-114">Ahora puede reemplazar el volcado de JSON con algo para mostrar los resultados de forma fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="143cc-114">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="143cc-115">En esta sección, modificará la `getEvents` función para que devuelva objetos con establecimiento inflexible de tipos `CalendarViewController` y lo modifique para usar una vista de tabla para representar los eventos.</span><span class="sxs-lookup"><span data-stu-id="143cc-115">In this section, you will modify the `getEvents` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

1. <span data-ttu-id="143cc-116">Abra **GraphManager. SWIFT**.</span><span class="sxs-lookup"><span data-stu-id="143cc-116">Open **GraphManager.swift**.</span></span> <span data-ttu-id="143cc-117">Reemplace la función `getEvents` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="143cc-117">Replace the existing `getEvents` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="1,17-38":::

1. <span data-ttu-id="143cc-118">Cree un nuevo archivo de **clase táctil de cacao** en el proyecto `CalendarTableViewCell.swift` **GraphTutorial** denominado.</span><span class="sxs-lookup"><span data-stu-id="143cc-118">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="143cc-119">Elija **UITableViewCell** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="143cc-119">Choose **UITableViewCell** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="143cc-120">Abra **CalendarTableViewCell. SWIFT** y agregue el código siguiente a la `CalendarTableViewCell` clase.</span><span class="sxs-lookup"><span data-stu-id="143cc-120">Open **CalendarTableViewCell.swift** and add the following code to the `CalendarTableViewCell` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. <span data-ttu-id="143cc-121">Abra **Main. Storyboard** y busque la **escena del calendario**.</span><span class="sxs-lookup"><span data-stu-id="143cc-121">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="143cc-122">Seleccione la **vista** de la **escena del calendario** y elimínela.</span><span class="sxs-lookup"><span data-stu-id="143cc-122">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Captura de pantalla de la vista en la escena del calendario](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="143cc-124">Agregue una **vista de tabla** de la **biblioteca** a la **escena del calendario**.</span><span class="sxs-lookup"><span data-stu-id="143cc-124">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="143cc-125">Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="143cc-125">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="143cc-126">Establezca **las celdas del prototipo** en **1**.</span><span class="sxs-lookup"><span data-stu-id="143cc-126">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="143cc-127">Use la **biblioteca** para agregar tres **etiquetas** a la celda del prototipo.</span><span class="sxs-lookup"><span data-stu-id="143cc-127">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="143cc-128">Seleccione la celda prototype y, a continuación, seleccione el **Inspector de identidad**.</span><span class="sxs-lookup"><span data-stu-id="143cc-128">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="143cc-129">Cambie la **clase** a **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="143cc-129">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="143cc-130">Seleccione el **Inspector de atributos** y establezca **identificador** en `EventCell`.</span><span class="sxs-lookup"><span data-stu-id="143cc-130">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="143cc-131">Con la **EventCell** seleccionada, seleccione el **Inspector de conexiones** y `durationLabel`Conéctese `organizerLabel`, `subjectLabel` y a las etiquetas que haya agregado a la celda en el guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="143cc-131">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="143cc-132">Establezca las propiedades y restricciones de las tres etiquetas como se indica a continuación.</span><span class="sxs-lookup"><span data-stu-id="143cc-132">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="143cc-133">**Etiqueta de asunto**</span><span class="sxs-lookup"><span data-stu-id="143cc-133">**Subject Label**</span></span>
        - <span data-ttu-id="143cc-134">Agregar restricción: espacio inicial para la vista de contenido margen inicial, valor: 0</span><span class="sxs-lookup"><span data-stu-id="143cc-134">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="143cc-135">Agregar restricción: espacio final para la vista de contenido margen final, valor: 0</span><span class="sxs-lookup"><span data-stu-id="143cc-135">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="143cc-136">Agregar restricción: espacio superior para la vista de contenido margen superior, valor: 0</span><span class="sxs-lookup"><span data-stu-id="143cc-136">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="143cc-137">**Etiqueta del organizador**</span><span class="sxs-lookup"><span data-stu-id="143cc-137">**Organizer Label**</span></span>
        - <span data-ttu-id="143cc-138">Fuente: sistema 12,0</span><span class="sxs-lookup"><span data-stu-id="143cc-138">Font: System 12.0</span></span>
        - <span data-ttu-id="143cc-139">Agregar restricción: espacio inicial para la vista de contenido margen inicial, valor: 0</span><span class="sxs-lookup"><span data-stu-id="143cc-139">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="143cc-140">Agregar restricción: espacio final para la vista de contenido margen final, valor: 0</span><span class="sxs-lookup"><span data-stu-id="143cc-140">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="143cc-141">Agregar restricción: espacio superior a etiqueta de asunto en la parte inferior, valor: estándar</span><span class="sxs-lookup"><span data-stu-id="143cc-141">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="143cc-142">**Etiqueta de duración**</span><span class="sxs-lookup"><span data-stu-id="143cc-142">**Duration Label**</span></span>
        - <span data-ttu-id="143cc-143">Fuente: sistema 12,0</span><span class="sxs-lookup"><span data-stu-id="143cc-143">Font: System 12.0</span></span>
        - <span data-ttu-id="143cc-144">Color: color gris oscuro</span><span class="sxs-lookup"><span data-stu-id="143cc-144">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="143cc-145">Agregar restricción: espacio inicial para la vista de contenido margen inicial, valor: 0</span><span class="sxs-lookup"><span data-stu-id="143cc-145">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="143cc-146">Agregar restricción: espacio final para la vista de contenido margen final, valor: 0</span><span class="sxs-lookup"><span data-stu-id="143cc-146">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="143cc-147">Agregar restricción: espacio superior para etiqueta de Organizer en la parte inferior, valor: estándar</span><span class="sxs-lookup"><span data-stu-id="143cc-147">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="143cc-148">Agregar restricción: espacio inferior para la vista de contenido margen inferior, valor: 8</span><span class="sxs-lookup"><span data-stu-id="143cc-148">Add constraint: Bottom space to Content View Bottom Margin, value: 8</span></span>

    ![Captura de pantalla del diseño de celda de prototipo](./images/prototype-cell-layout.png)

1. <span data-ttu-id="143cc-150">Abra **CalendarViewController. SWIFT** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="143cc-150">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. <span data-ttu-id="143cc-151">Ejecute la aplicación, inicie sesión y pulse la pestaña **calendario** . Debe ver la lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="143cc-151">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/calendar-list.png)
