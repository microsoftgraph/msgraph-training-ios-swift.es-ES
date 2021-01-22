<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio incorporará Microsoft Graph en la aplicación. Para esta aplicación, usará el SDK de Microsoft Graph para [el Objetivo C](https://github.com/microsoftgraph/msgraph-sdk-objc) para realizar llamadas a Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obtener eventos de calendario de Outlook

En esta sección, ampliará la clase para agregar una función para obtener los eventos del usuario de la semana actual y actualizar para usar `GraphManager` `CalendarViewController` estas nuevas funciones.

1. Abra **GraphManager.swift** y agregue el siguiente método a la `GraphManager` clase.

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
    > Ten en cuenta lo que hace `getCalendarView` el código.
    >
    > - La dirección URL a la que se llamará es `/v1.0/me/calendarview` .
    >   - Los `startDateTime` parámetros de consulta y de consulta definen el inicio y el final de la vista de `endDateTime` calendario.
    >   - El `select` parámetro de consulta limita los campos devueltos para cada evento a solo aquellos que la vista usará realmente.
    >   - El `orderby` parámetro de consulta ordena los resultados por hora de inicio.
    >   - El `top` parámetro de consulta solicita 25 resultados por página.
    >   - el encabezado hace que Microsoft Graph devuelva las horas de inicio y finalización de cada evento en la `Prefer: outlook.timezone` zona horaria del usuario.

1. Cree un nuevo **archivo Swift** en el **proyecto GraphTutorial** denominado **GraphToIana.swift**. Agregue el siguiente código al archivo.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphToIana.swift" id="GraphToIanaSnippet":::

    Esto hace una búsqueda simple para buscar un identificador de zona horaria IANA basado en el nombre de zona horaria devuelto por Microsoft Graph.

1. Abra **CalendarViewController.swift** y reemplace todo su contenido por el siguiente código.

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

Ahora puedes ejecutar la aplicación, iniciar sesión y pulsar el elemento **de** navegación Calendario en el menú. Debería ver un volcado JSON de los eventos en la aplicación.

## <a name="display-the-results"></a>Mostrar los resultados

Ahora puede reemplazar el volcado json por algo para mostrar los resultados de forma fácil de usar. En esta sección, modificará la función para devolver objetos fuertemente especificados y modificará para usar una vista de tabla `getCalendarView` `CalendarViewController` para representar los eventos.

1. Abra **GraphManager.swift**. Reemplace la función `getCalendarView` existente por lo siguiente.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/GraphManager.swift" id="GetEventsSnippet" highlight="3,28-49":::

1. Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewController.swift` . Elija **UITableViewController** en la **subclase del** campo.

1. Abra **CalendarTableViewController.swift** y reemplace su contenido por lo siguiente.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewController.swift" id="CalendarTableViewControllerSnippet":::

1. Cree un nuevo **archivo de clase Cocoa Touch** en el proyecto **GraphTutorial** denominado `CalendarTableViewCell.swift` . Elija **UITableViewCell** en la **subclase del** campo.

1. Abre **CalendarTableViewCell.swift y** agrega las siguientes propiedades a la `CalendarTableViewCell` clase.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarTableViewCell.swift" id="PropertiesSnippet":::

1. Abra **Main.storyboard** y busque la **escena del calendario.** Elimine la vista de desplazamiento de la vista raíz.
1. Mediante la **biblioteca,** agregue una **barra de navegación** en la parte superior de la vista.
1. Haga doble clic en **el título** de la barra de navegación y actualíquelo a `Calendar` .
1. Mediante la **biblioteca,** agregue un elemento **de botón** de barra a la derecha de la barra de navegación.
1. Seleccione el botón nueva barra y, a continuación, seleccione el **Inspector de atributos**. Cambiar **imagen** a **más**.
1. Agregue una **vista de contenedor** de la biblioteca **a** la vista debajo de la barra de navegación. Cambie el tamaño de la vista de contenedor para que tome todo el espacio restante en la vista.
1. Establezca las restricciones en la barra de navegación y la vista de contenedor como se muestra a continuación.

    - **Barra de navegación**
        - Agregar restricción: Alto, valor: 44
        - Agregar restricción: Espacio inicial a Área segura, valor: 0
        - Agregar restricción: Espacio final a Área segura, valor: 0
        - Agregar restricción: Espacio superior a Área segura, valor: 0
    - **Vista de contenedor**
        - Agregar restricción: Espacio inicial a Área segura, valor: 0
        - Agregar restricción: Espacio final a Área segura, valor: 0
        - Agregar restricción: Espacio superior en la parte inferior de la barra de navegación, valor: 0
        - Agregar restricción: espacio inferior a Área segura, valor: 0

1. Busca el segundo controlador de vista agregado al guión gráfico cuando agregaste la vista de contenedor. Se conecta a la escena **de calendario** mediante un segue de inserción. Seleccione este controlador y use el **Inspector de identidad** para cambiar la **clase** **a CalendarTableViewController**.
1. Elimine la **vista del** controlador de vista de tabla **de calendario.**
1. Agregue una **vista de tabla** de la biblioteca **al** controlador de vista de tabla **de calendario.**
1. Seleccione la vista de tabla y, a continuación, seleccione el **Inspector de atributos.** Establecer **celdas prototipo** en **1**.
1. Arrastre el borde inferior de la celda prototipo para darle un área más grande con la que trabajar.
1. Use la **biblioteca para** agregar tres **etiquetas a** la celda prototipo.
1. Seleccione la celda prototipo y, a continuación, seleccione **el Inspector de identidad.** Cambiar **clase** a **CalendarTableViewCell**.
1. Seleccione el **Inspector de atributos** y establezca El **identificador** en `EventCell` .
1. Con **eventCell** seleccionado, seleccione el **Inspector de** conexiones y conéctese, y a las etiquetas que agregó a la celda en `durationLabel` el `organizerLabel` `subjectLabel` guión gráfico.
1. Establezca las propiedades y restricciones de las tres etiquetas de la siguiente manera.

    - **Etiqueta de asunto**
        - Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0
        - Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0
        - Agregar restricción: Espacio superior al margen superior de la vista de contenido, valor: 0
    - **Etiqueta del organizador**
        - Fuente: System 12.0
        - Agregar restricción: Alto, valor: 15
        - Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0
        - Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0
        - Agregar restricción: Espacio superior a La parte inferior de la etiqueta de asunto, valor: Estándar
    - **Etiqueta de duración**
        - Fuente: System 12.0
        - Color: Color gris oscuro
        - Agregar restricción: Alto, valor: 15
        - Agregar restricción: espacio inicial al margen inicial de la vista de contenido, valor: 0
        - Agregar restricción: espacio final al margen final de la vista de contenido, valor: 0
        - Agregar restricción: Espacio superior en la parte inferior de la etiqueta del organizador, valor: Estándar
        - Agregar restricción: espacio inferior al margen inferior de la vista de contenido, valor: 0

1. Seleccione **eventCell** y, a continuación, seleccione **el Inspector de tamaño**. Habilitar **Automático para** alto de **fila.**

    ![Captura de pantalla de los controladores de vista de tabla de calendario y calendario](images/calendar-table-storyboard.png)

1. Abra **CalendarViewController.swift** y reemplace su contenido por el siguiente código.

    :::code language="swift" source="../demo/GraphTutorial/GraphTutorial/CalendarViewController.swift" id="CalendarViewSnippet":::

1. Ejecuta la aplicación, inicia sesión y pulsa en la **pestaña** Calendario. Debería ver la lista de eventos.

    ![Captura de pantalla de la tabla de eventos](images/calendar-list.png)
