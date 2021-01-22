<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5c8b8-101">En este ejercicio incorporará Microsoft Graph en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="5c8b8-102">Para esta aplicación, usará el SDK de Microsoft Graph para [el Objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="5c8b8-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="5c8b8-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="5c8b8-104">En esta sección, ampliará la clase para agregar una función para obtener los eventos del usuario de la semana actual y actualizar para usar `GraphManager` `CalendarViewController` estas nuevas funciones.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-104">In this section you will extend the `GraphManager` class to add a function to get the user's events for the current week and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="5c8b8-105">Abra **GraphManager.swift** y agregue el siguiente método a la `GraphManager` clase.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

    ```Swift
    public func getCalendarView(viewStart: String,
                                viewEnd: String,
                                completion: @escaping(Data?, Error?) -> Void) {
        // GET /me/calendarview
        // Set start and end of the view
        let start = "startDateTime=\(viewStart)"
        let end = "endDateTime=\(viewEnd)"
        // Only return these fields in results
        let select = "$select=subject,organizer,start,end"
        // Sort results by when they were created, newest first
        let orderBy = "$orderby=start/dateTime"
        // Request at most 25 results
        let top = "$top=25"

        let eventsRequest = NSMutableURLRequest(url: URL(string: "\(MSGraphBaseURL)/me/calendarview?\(start)&\(end)&\(select)&\(orderBy)&\(top)")!)

        // Add the Prefer: outlook.timezone header to get start and end times
        // in user's time zone
        eventsRequest.addValue("outlook.timezone=\"\(self.userTimeZone)\"", forHTTPHeaderField: "Prefer")

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
    > <span data-ttu-id="5c8b8-106">Ten en cuenta lo que hace `getCalendarView` el código.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-106">Consider what the code in `getCalendarView` is doing.</span></span>
    >
    > - <span data-ttu-id="5c8b8-107">La dirección URL a la que se llamará es `/v1.0/me/calendarview` .</span><span class="sxs-lookup"><span data-stu-id="5c8b8-107">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
    >   - <span data-ttu-id="5c8b8-108">Los `startDateTime` parámetros de consulta y de consulta definen el inicio y el final de la vista de `endDateTime` calendario.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-108">The `startDateTime` and `endDateTime` query parameters define the start and end of the calendar view.</span></span>
    >   - <span data-ttu-id="5c8b8-109">El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-109">The `select` query parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    >   - <span data-ttu-id="5c8b8-110">El `orderby` parámetro de consulta ordena los resultados por hora de inicio.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-110">The `orderby` query parameter sorts the results by start time.</span></span>
    >   - <span data-ttu-id="5c8b8-111">El `top` parámetro de consulta solicita 25 resultados por página.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-111">The `top` query parameter requests 25 results per page.</span></span>
    >   - <span data-ttu-id="5c8b8-112">el encabezado hace que Microsoft Graph devuelva las horas de inicio y finalización de cada evento en la `Prefer: outlook.timezone` zona horaria del usuario.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-112">the `Prefer: outlook.timezone` header causes the Microsoft Graph to return the start and end times of each event in the user's time zone.</span></span>

1. <span data-ttu-id="5c8b8-113">Cree un nuevo **archivo Swift** en el **proyecto GraphTutorial** denominado **GraphToIana.swift**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-113">Create a new **Swift File** in the **GraphTutorial** project named **GraphToIana.swift**.</span></span> <span data-ttu-id="5c8b8-114">Agregue el siguiente código al archivo.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-114">Add the following code to the file.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.swift" id="GraphToIanaSnippet":::

    <span data-ttu-id="5c8b8-115">Esto hace una búsqueda simple para buscar un identificador de zona horaria IANA basado en el nombre de zona horaria devuelto por Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-115">This does a simple lookup to find an IANA time zone identifier based on the time zone name returned by Microsoft Graph.</span></span>

1. <span data-ttu-id="5c8b8-116">Abra **CalendarViewController.swift** y reemplace todo su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-116">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

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

            // Calculate the start and end of the current week
            let timeZone = GraphToIana.getIanaIdentifier(graphIdentifer: GraphManager.instance.userTimeZone)
            let now = Date()
            var calendar = Calendar(identifier: .gregorian)
            calendar.timeZone = TimeZone(identifier: timeZone)!

            let startOfWeek = calendar.dateComponents([.calendar, .yearForWeekOfYear, .weekOfYear], from: now).date!
            let endOfWeek = calendar.date(byAdding: .day, value: 7, to: startOfWeek)!

            // Convert start and end to ISO 8601 strings
            let isoFormatter = ISO8601DateFormatter()
            let viewStart = isoFormatter.string(from: startOfWeek)
            let viewEnd = isoFormatter.string(from: endOfWeek)

            GraphManager.instance.getCalendarView(viewStart: viewStart, viewEnd: viewEnd) {
                (data: Data?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

                    // TEMPORARY
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

<span data-ttu-id="5c8b8-117">Ahora puedes ejecutar la aplicación, iniciar sesión y pulsar el elemento **de** navegación Calendario en el menú.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-117">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="5c8b8-118">Debería ver un volcado JSON de los eventos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-118">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="5c8b8-119">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="5c8b8-119">Display the results</span></span>

<span data-ttu-id="5c8b8-120">Ahora puede reemplazar el volcado json por algo para mostrar los resultados de forma fácil de usar.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-120">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="5c8b8-121">En esta sección, modificará la función para devolver objetos fuertemente especificados y modificará para usar una vista de tabla `getCalendarView` `CalendarViewController` para representar los eventos.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-121">In this section, you will modify the `getCalendarView` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

1. <span data-ttu-id="5c8b8-122">Abra **GraphManager.swift**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-122">Open **GraphManager.swift**.</span></span> <span data-ttu-id="5c8b8-123">Reemplace la función `getCalendarView` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-123">Replace the existing `getCalendarView` function with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="3,28-49":::

1. <span data-ttu-id="5c8b8-124">Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewController.swift` .</span><span class="sxs-lookup"><span data-stu-id="5c8b8-124">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewController.swift`.</span></span> <span data-ttu-id="5c8b8-125">Elija **UITableViewController** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-125">Choose **UITableViewController** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="5c8b8-126">Abra **CalendarTableViewController.swift** y reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-126">Open **CalendarTableViewController.swift** and replace its contents with the following.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.swift" id="CalendarTableViewControllerSnippet":::

1. <span data-ttu-id="5c8b8-127">Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewCell.swift` .</span><span class="sxs-lookup"><span data-stu-id="5c8b8-127">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="5c8b8-128">Elija **UITableViewCell** en la **subclase del** campo.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-128">Choose **UITableViewCell** in the **Subclass of** field.</span></span>

1. <span data-ttu-id="5c8b8-129">Abre **CalendarTableViewCell.swift y** agrega las siguientes propiedades a la `CalendarTableViewCell` clase.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-129">Open **CalendarTableViewCell.swift** and add the following properties to the `CalendarTableViewCell` class.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. <span data-ttu-id="5c8b8-130">Abra **Main.storyboard** y busque la **escena del calendario.**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-130">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="5c8b8-131">Elimine la vista de desplazamiento de la vista raíz.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-131">Delete the scroll view from the root view.</span></span>
1. <span data-ttu-id="5c8b8-132">Mediante la **biblioteca,** agregue una **barra de navegación** en la parte superior de la vista.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-132">Using the **Library**, add a **Navigation Bar** to the top of the view.</span></span>
1. <span data-ttu-id="5c8b8-133">Haga doble clic en **el título** de la barra de navegación y actualíquelo a `Calendar` .</span><span class="sxs-lookup"><span data-stu-id="5c8b8-133">Double-click the **Title** in the navigation bar and update it to `Calendar`.</span></span>
1. <span data-ttu-id="5c8b8-134">Mediante la **biblioteca,** agregue un elemento **de botón** de barra a la derecha de la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-134">Using the **Library**, add a **Bar Button Item** to the right-hand side of the navigation bar.</span></span>
1. <span data-ttu-id="5c8b8-135">Seleccione el botón nueva barra y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-135">Select the new bar button, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="5c8b8-136">Cambiar **imagen** a **más**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-136">Change **Image** to **plus**.</span></span>
1. <span data-ttu-id="5c8b8-137">Agregue una **vista de contenedor** de la biblioteca **a** la vista debajo de la barra de navegación.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-137">Add a **Container View** from the **Library** to the view under the navigation bar.</span></span> <span data-ttu-id="5c8b8-138">Cambie el tamaño de la vista de contenedor para que tome todo el espacio restante en la vista.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-138">Resize the container view to take all of the remaining space in the view.</span></span>
1. <span data-ttu-id="5c8b8-139">Establezca las restricciones en la barra de navegación y la vista de contenedor como se muestra a continuación.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-139">Set constraints on the navigation bar and container view as follows.</span></span>

    - <span data-ttu-id="5c8b8-140">**Barra de navegación**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-140">**Navigation Bar**</span></span>
        - <span data-ttu-id="5c8b8-141">Agregar restricción: Alto, valor: 44</span><span class="sxs-lookup"><span data-stu-id="5c8b8-141">Add constraint: Height, value: 44</span></span>
        - <span data-ttu-id="5c8b8-142">Agregar restricción: Espacio inicial a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-142">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="5c8b8-143">Agregar restricción: Espacio final a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-143">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="5c8b8-144">Agregar restricción: Espacio superior a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-144">Add constraint: Top space to Safe Area, value: 0</span></span>
    - <span data-ttu-id="5c8b8-145">**Vista de contenedor**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-145">**Container View**</span></span>
        - <span data-ttu-id="5c8b8-146">Agregar restricción: Espacio inicial a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-146">Add constraint: Leading space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="5c8b8-147">Agregar restricción: Espacio final a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-147">Add constraint: Trailing space to Safe Area, value: 0</span></span>
        - <span data-ttu-id="5c8b8-148">Agregar restricción: Espacio superior en la parte inferior de la barra de navegación, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-148">Add constraint: Top space to Navigation Bar Bottom, value: 0</span></span>
        - <span data-ttu-id="5c8b8-149">Agregar restricción: espacio inferior a Área segura, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-149">Add constraint: Bottom space to Safe Area, value: 0</span></span>

1. <span data-ttu-id="5c8b8-150">Busca el segundo controlador de vista agregado al guión gráfico cuando agregaste la vista de contenedor.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-150">Locate the second view controller added to the storyboard when you added the container view.</span></span> <span data-ttu-id="5c8b8-151">Se conecta a la escena **de calendario** mediante un segue de inserción.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-151">It is connected to the **Calendar Scene** by an embed segue.</span></span> <span data-ttu-id="5c8b8-152">Seleccione este controlador y use el **Inspector de identidad** para cambiar la **clase** **a CalendarTableViewController**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-152">Select this controller and use the **Identity Inspector** to change **Class** to **CalendarTableViewController**.</span></span>
1. <span data-ttu-id="5c8b8-153">Elimine la **vista del** controlador de vista de tabla **de calendario.**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-153">Delete the **View** from the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="5c8b8-154">Agregue una **vista de tabla** de la biblioteca **al** controlador de vista de tabla **de calendario.**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-154">Add a **Table View** from the **Library** to the **Calendar Table View Controller**.</span></span>
1. <span data-ttu-id="5c8b8-155">Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos.**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-155">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="5c8b8-156">Establecer **celdas prototipo** en **1**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-156">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="5c8b8-157">Arrastre el borde inferior de la celda prototipo para darle un área más grande con la que trabajar.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-157">Drag the bottom edge of the prototype cell to give you a larger area to work with.</span></span>
1. <span data-ttu-id="5c8b8-158">Use la **biblioteca para** agregar tres **etiquetas a** la celda prototipo.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-158">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="5c8b8-159">Seleccione la celda prototipo y, a continuación, seleccione **el Inspector de identidad.**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-159">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="5c8b8-160">Cambiar **clase** a **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-160">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="5c8b8-161">Seleccione el **Inspector de atributos** y establezca El **identificador** en `EventCell` .</span><span class="sxs-lookup"><span data-stu-id="5c8b8-161">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="5c8b8-162">Con **eventCell** seleccionado, seleccione el **Inspector de** conexiones y conéctese, y a las etiquetas que agregó a la celda en `durationLabel` el `organizerLabel` `subjectLabel` guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-162">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>
1. <span data-ttu-id="5c8b8-163">Establezca las propiedades y restricciones de las tres etiquetas de la siguiente manera.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-163">Set the properties and constraints on the three labels as follows.</span></span>

    - <span data-ttu-id="5c8b8-164">**Etiqueta de asunto**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-164">**Subject Label**</span></span>
        - <span data-ttu-id="5c8b8-165">Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-165">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="5c8b8-166">Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-166">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="5c8b8-167">Agregar restricción: Espacio superior al margen superior de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-167">Add constraint: Top space to Content View Top Margin, value: 0</span></span>
    - <span data-ttu-id="5c8b8-168">**Etiqueta del organizador**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-168">**Organizer Label**</span></span>
        - <span data-ttu-id="5c8b8-169">Fuente: System 12.0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-169">Font: System 12.0</span></span>
        - <span data-ttu-id="5c8b8-170">Agregar restricción: Alto, valor: 15</span><span class="sxs-lookup"><span data-stu-id="5c8b8-170">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="5c8b8-171">Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-171">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="5c8b8-172">Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-172">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="5c8b8-173">Agregar restricción: Espacio superior a La parte inferior de la etiqueta de asunto, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="5c8b8-173">Add constraint: Top space to Subject Label Bottom, value: Standard</span></span>
    - <span data-ttu-id="5c8b8-174">**Etiqueta de duración**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-174">**Duration Label**</span></span>
        - <span data-ttu-id="5c8b8-175">Fuente: System 12.0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-175">Font: System 12.0</span></span>
        - <span data-ttu-id="5c8b8-176">Color: Color gris oscuro</span><span class="sxs-lookup"><span data-stu-id="5c8b8-176">Color: Dark Gray Color</span></span>
        - <span data-ttu-id="5c8b8-177">Agregar restricción: Alto, valor: 15</span><span class="sxs-lookup"><span data-stu-id="5c8b8-177">Add constraint: Height, value: 15</span></span>
        - <span data-ttu-id="5c8b8-178">Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-178">Add constraint: Leading space to Content View Leading Margin, value: 0</span></span>
        - <span data-ttu-id="5c8b8-179">Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-179">Add constraint: Trailing space to Content View Trailing Margin, value: 0</span></span>
        - <span data-ttu-id="5c8b8-180">Agregar restricción: Espacio superior en la parte inferior de la etiqueta del organizador, valor: Estándar</span><span class="sxs-lookup"><span data-stu-id="5c8b8-180">Add constraint: Top space to Organizer Label Bottom, value: Standard</span></span>
        - <span data-ttu-id="5c8b8-181">Agregar restricción: espacio inferior al margen inferior de la vista de contenido, valor: 0</span><span class="sxs-lookup"><span data-stu-id="5c8b8-181">Add constraint: Bottom space to Content View Bottom Margin, value: 0</span></span>

1. <span data-ttu-id="5c8b8-182">Seleccione **eventCell** y, a continuación, seleccione **el Inspector de tamaño**.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-182">Select the **EventCell**, then select the **Size Inspector**.</span></span> <span data-ttu-id="5c8b8-183">Habilitar **Automático para** alto de **fila.**</span><span class="sxs-lookup"><span data-stu-id="5c8b8-183">Enable **Automatic** for **Row Height**.</span></span>

    ![Captura de pantalla de los controladores de vista de tabla de calendario y calendario](images/calendar-table-storyboard.png)

1. <span data-ttu-id="5c8b8-185">Abra **CalendarViewController.swift** y reemplace su contenido por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-185">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. <span data-ttu-id="5c8b8-186">Ejecuta la aplicación, inicia sesión y pulsa en la **pestaña** Calendario. Debería ver la lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="5c8b8-186">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](images/calendar-list.png)
