<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="11a2a-101">En este ejercicio, incorporará Microsoft Graph a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="11a2a-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="11a2a-102">Para esta aplicación, usará el [SDK de Microsoft Graph para Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="11a2a-102">For this application, you will use the [Microsoft Graph SDK for Objective C](https://github.com/microsoftgraph/msgraph-sdk-objc) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="11a2a-103">Obtener eventos de calendario de Outlook</span><span class="sxs-lookup"><span data-stu-id="11a2a-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="11a2a-104">En esta sección, ampliará la `GraphManager` clase para agregar una función para obtener los eventos del usuario y actualizar `CalendarViewController` para usar estas funciones nuevas.</span><span class="sxs-lookup"><span data-stu-id="11a2a-104">In this section you will extend the `GraphManager` class to add a function to get the user's events and update `CalendarViewController` to use these new functions.</span></span>

1. <span data-ttu-id="11a2a-105">Abra **GraphManager. SWIFT** y agregue el siguiente método a la `GraphManager` clase.</span><span class="sxs-lookup"><span data-stu-id="11a2a-105">Open **GraphManager.swift** and add the following method to the `GraphManager` class.</span></span>

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
    > <span data-ttu-id="11a2a-106">Tenga en cuenta lo que `getEvents` hace el código.</span><span class="sxs-lookup"><span data-stu-id="11a2a-106">Consider what the code in `getEvents` is doing.</span></span>
    >
    > - <span data-ttu-id="11a2a-107">La dirección URL a la que se `/v1.0/me/events`llamará es.</span><span class="sxs-lookup"><span data-stu-id="11a2a-107">The URL that will be called is `/v1.0/me/events`.</span></span>
    > - <span data-ttu-id="11a2a-108">El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la aplicación usará realmente.</span><span class="sxs-lookup"><span data-stu-id="11a2a-108">The `select` query parameter limits the fields returned for each events to just those the app will actually use.</span></span>
    > - <span data-ttu-id="11a2a-109">El `orderby` parámetro de consulta ordena los resultados por la fecha y hora en que se crearon, con el elemento más reciente en primer lugar.</span><span class="sxs-lookup"><span data-stu-id="11a2a-109">The `orderby` query parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="11a2a-110">Abra **CalendarViewController. SWIFT** y reemplace todo su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="11a2a-110">Open **CalendarViewController.swift** and replace its entire contents with the following code.</span></span>

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

<span data-ttu-id="11a2a-111">Ahora puede ejecutar la aplicación, iniciar sesión y puntear en el elemento de navegación **calendario** del menú.</span><span class="sxs-lookup"><span data-stu-id="11a2a-111">You can now run the app, sign in, and tap the **Calendar** navigation item in the menu.</span></span> <span data-ttu-id="11a2a-112">Debería ver un volcado JSON de los eventos en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="11a2a-112">You should see a JSON dump of the events in the app.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="11a2a-113">Mostrar los resultados</span><span class="sxs-lookup"><span data-stu-id="11a2a-113">Display the results</span></span>

<span data-ttu-id="11a2a-114">Ahora puede reemplazar el volcado de JSON con algo para mostrar los resultados de forma fácil de uso.</span><span class="sxs-lookup"><span data-stu-id="11a2a-114">Now you can replace the JSON dump with something to display the results in a user-friendly manner.</span></span> <span data-ttu-id="11a2a-115">En esta sección, modificará la `getEvents` función para que devuelva objetos con establecimiento inflexible de tipos `CalendarViewController` y lo modifique para usar una vista de tabla para representar los eventos.</span><span class="sxs-lookup"><span data-stu-id="11a2a-115">In this section, you will modify the `getEvents` function to return strongly-typed objects, and modify `CalendarViewController` to use a table view to render the events.</span></span>

### <a name="update-getevents"></a><span data-ttu-id="11a2a-116">Actualizar getEvents</span><span class="sxs-lookup"><span data-stu-id="11a2a-116">Update getEvents</span></span>

1. <span data-ttu-id="11a2a-117">Abra **GraphManager. SWIFT**.</span><span class="sxs-lookup"><span data-stu-id="11a2a-117">Open **GraphManager.swift**.</span></span> <span data-ttu-id="11a2a-118">Cambie la `getEvents` declaración de función a lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="11a2a-118">Change the `getEvents` function declaration to the following.</span></span>

    ```Swift
    public func getEvents(completion: @escaping([MSGraphEvent]?, Error?) -> Void)
    ```

1. <span data-ttu-id="11a2a-119">Reemplace la línea `completion(eventsData, nil)` por el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="11a2a-119">Replace the `completion(eventsData, nil)` line with the following code.</span></span>

    ```Swift
    do {
        // Deserialize response as events collection
        let eventsCollection = try MSCollection(data: eventsData)
        var eventArray: [MSGraphEvent] = []

        eventsCollection.value.forEach({
            (rawEvent: Any) in
            // Convert JSON to a dictionary
            guard let eventDict = rawEvent as? [String: Any] else {
                return
            }

            // Deserialize event from the dictionary
            let event = MSGraphEvent(dictionary: eventDict)!
            eventArray.append(event)
        })

        // Return the array
        completion(eventArray, nil)
    } catch {
        completion(nil, error)
    }
    ```

### <a name="update-calendarviewcontroller"></a><span data-ttu-id="11a2a-120">Actualizar CalendarViewController</span><span class="sxs-lookup"><span data-stu-id="11a2a-120">Update CalendarViewController</span></span>

1. <span data-ttu-id="11a2a-121">Cree un nuevo archivo de **clase táctil de cacao** en el proyecto `CalendarTableViewCell.swift` **GraphTutorial** denominado.</span><span class="sxs-lookup"><span data-stu-id="11a2a-121">Create a new **Cocoa Touch Class** file in the **GraphTutorial** project named `CalendarTableViewCell.swift`.</span></span> <span data-ttu-id="11a2a-122">Elija **UITableViewCell** en el campo **subclase de** .</span><span class="sxs-lookup"><span data-stu-id="11a2a-122">Choose **UITableViewCell** in the **Subclass of** field.</span></span>
1. <span data-ttu-id="11a2a-123">Abra **CalendarTableViewCell. SWIFT** y agregue el código siguiente a la `CalendarTableViewCell` clase.</span><span class="sxs-lookup"><span data-stu-id="11a2a-123">Open **CalendarTableViewCell.swift** and add the following code to the `CalendarTableViewCell` class.</span></span>

    ```Swift
    @IBOutlet var subjectLabel: UILabel!
    @IBOutlet var organizerLabel: UILabel!
    @IBOutlet var durationLabel: UILabel!

    var subject: String? {
        didSet {
            subjectLabel.text = subject
        }
    }

    var organizer: String? {
        didSet {
            organizerLabel.text = organizer
        }
    }

    var duration: String? {
        didSet {
            durationLabel.text = duration
        }
    }
    ```

1. <span data-ttu-id="11a2a-124">Abra **Main. Storyboard** y busque la **escena del calendario**.</span><span class="sxs-lookup"><span data-stu-id="11a2a-124">Open **Main.storyboard** and locate the **Calendar Scene**.</span></span> <span data-ttu-id="11a2a-125">Seleccione la **vista** de la **escena del calendario** y elimínela.</span><span class="sxs-lookup"><span data-stu-id="11a2a-125">Select the **View** in the **Calendar Scene** and delete it.</span></span>

    ![Captura de pantalla de la vista en la escena del calendario](./images/view-in-calendar-scene.png)

1. <span data-ttu-id="11a2a-127">Agregue una **vista de tabla** de la **biblioteca** a la **escena del calendario**.</span><span class="sxs-lookup"><span data-stu-id="11a2a-127">Add a **Table View** from the **Library** to the **Calendar Scene**.</span></span>
1. <span data-ttu-id="11a2a-128">Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos**.</span><span class="sxs-lookup"><span data-stu-id="11a2a-128">Select the table view, then select the **Attributes Inspector**.</span></span> <span data-ttu-id="11a2a-129">Establezca **las celdas del prototipo** en **1**.</span><span class="sxs-lookup"><span data-stu-id="11a2a-129">Set **Prototype Cells** to **1**.</span></span>
1. <span data-ttu-id="11a2a-130">Use la **biblioteca** para agregar tres **etiquetas** a la celda del prototipo.</span><span class="sxs-lookup"><span data-stu-id="11a2a-130">Use the **Library** to add three **Labels** to the prototype cell.</span></span>
1. <span data-ttu-id="11a2a-131">Seleccione la celda prototype y, a continuación, seleccione el **Inspector de identidad**.</span><span class="sxs-lookup"><span data-stu-id="11a2a-131">Select the prototype cell, then select the **Identity Inspector**.</span></span> <span data-ttu-id="11a2a-132">Cambie la **clase** a **CalendarTableViewCell**.</span><span class="sxs-lookup"><span data-stu-id="11a2a-132">Change **Class** to **CalendarTableViewCell**.</span></span>
1. <span data-ttu-id="11a2a-133">Seleccione el **Inspector de atributos** y establezca **identificador** en `EventCell`.</span><span class="sxs-lookup"><span data-stu-id="11a2a-133">Select the **Attributes Inspector** and set **Identifier** to `EventCell`.</span></span>
1. <span data-ttu-id="11a2a-134">Con la **EventCell** seleccionada, seleccione el **Inspector de conexiones** y `durationLabel`Conéctese `organizerLabel`, `subjectLabel` y a las etiquetas que haya agregado a la celda en el guión gráfico.</span><span class="sxs-lookup"><span data-stu-id="11a2a-134">With the **EventCell** selected, select the **Connections Inspector** and connect `durationLabel`, `organizerLabel`, and `subjectLabel` to the labels you added to the cell on the storyboard.</span></span>

    ![Captura de pantalla del diseño de celda de prototipo](./images/prototype-cell-layout.png)

1. <span data-ttu-id="11a2a-136">Abra **CalendarViewController. SWIFT** y reemplace su contenido por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="11a2a-136">Open **CalendarViewController.swift** and replace its contents with the following code.</span></span>

    ```Swift
    import UIKit
    import MSGraphClientModels

    class CalendarViewController: UITableViewController {

        private let tableCellIdentifier = "EventCell"
        private let spinner = SpinnerViewController()
        private var events: [MSGraphEvent]?

        override func viewDidLoad() {
            super.viewDidLoad()

            // Do any additional setup after loading the view.
            self.spinner.start(container: self)

            GraphManager.instance.getEvents {
                (eventArray: [MSGraphEvent]?, error: Error?) in
                DispatchQueue.main.async {
                    self.spinner.stop()

                    guard let events = eventArray, error == nil else {
                        // Show the error
                        let alert = UIAlertController(title: "Error getting events",
                                                      message: error.debugDescription,
                                                      preferredStyle: .alert)

                        alert.addAction(UIAlertAction(title: "OK", style: .default, handler: nil))
                        self.present(alert, animated: true)
                        return
                    }

                    self.events = events
                    self.tableView.reloadData()
                }
            }
        }

        // Number of sections, always 1
        override func numberOfSections(in tableView: UITableView) -> Int {
            return 1
        }

        // Return the number of events in the table
        override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return events?.count ?? 0
        }

        override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            let cell = tableView.dequeueReusableCell(withIdentifier: tableCellIdentifier, for: indexPath) as! CalendarTableViewCell

            // Get the event that corresponds to the row
            let event = events?[indexPath.row]

            // Configure the cell
            cell.subject = event?.subject
            cell.organizer = event?.organizer?.emailAddress?.name

            // Build a duration string
            let duration = "\(self.formatGraphDateTime(dateTime: event?.start)) to \(self.formatGraphDateTime(dateTime: event?.end))"
            cell.duration = duration

            return cell
        }

        private func formatGraphDateTime(dateTime: MSGraphDateTimeTimeZone?) -> String {
            guard let graphDateTime = dateTime else {
                return ""
            }

            // Create a formatter to parse Graph's date format
            let isoFormatter = DateFormatter()
            isoFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss.SSSSSSS"

            // Specify the timezone
            isoFormatter.timeZone = TimeZone(identifier: graphDateTime.timeZone!)

            let date = isoFormatter.date(from: graphDateTime.dateTime)

            // Output like 5/5/2019, 2:00 PM
            let dateFormatter = DateFormatter()
            dateFormatter.dateStyle = .short
            dateFormatter.timeStyle = .short

            return dateFormatter.string(from: date!)
        }
    }
    ```

1. <span data-ttu-id="11a2a-137">Ejecute la aplicación, inicie sesión y pulse la pestaña **calendario** . Debe ver la lista de eventos.</span><span class="sxs-lookup"><span data-stu-id="11a2a-137">Run the app, sign in, and tap the **Calendar** tab. You should see the list of events.</span></span>

    ![Captura de pantalla de la tabla de eventos](./images/calendar-list.png)
