Learn Phoenix framework

Repositorio donde iré tomando notas según vaya siguiendo las guías de
http://www.phoenixframework.org/

# Instalar Phoenix

Una vez instalado Erlang y Elixir, podemos instalar Phoenix con el comando:

    mix archive.install https://github.com/phoenixframework/archives/raw/master/phoenix_new.ez

Por defecto, Phoenix usa PostgreSQL. Se puede encontrar una guía para instalarlo aquí:
https://wiki.postgresql.org/wiki/Detailed_installation_guides, o incluso mejor:
https://help.ubuntu.com/community/PostgreSQL

Para crear un proyecto Phoenix con lo mínimo:

    mix phoenix.new web --no-brunch --no-ecto

Phoenix usa brunch.io para gestionar recursos estáticos (JavaScript, CSS,...), compilarlos,
minificarlos,... Y para ello se debe tener instalado Node.js >= 5.0

# Comenzando

Con todas las herramientas listas (Erlan, Hex, Elixr, PostgreSQL, Node.js) podemos crear
nuestro primer proyecto Phoenix:

    mix phoenix.new <nombre del proyecto>

Crea la base de datos con:

    mix ecto.create

El comando anterior supone un usuario con permisos en PostgreSQL como `postgres/postgres`

Como paso final, arranquemos el servidor con:

    mix phoenix.server

# Añadiendo páginas

El comando anterior crea una estructura de directorios básica, común a todos los proyectos
Phoenix:

```
├── _build
├── config
├── deps
├── lib
├── priv
├── test
├── web
```

Donde el directorio `web` expandido se vería así:

```
├── channels
    └── user_socket.ex
├── controllers
│   └── page_controller.ex
├── models
├── static
│   ├── assets
│   |   ├── images
|   |   |   └── phoenix.png
|   |   └── favicon.ico
|   |   └── robots.txt
│   |   ├── vendor
├── templates
│   ├── layout
│   │   └── app.html.eex
│   └── page
│       └── index.html.eex
└── views
|   ├── error_helpers.ex
|   ├── error_view.ex
|   ├── layout_view.ex
|   └── page_view.ex
├── router.ex
├── gettext.ex
├── web.ex
```

Todos nuestros recursos estáticos, que no necesiten ser procesados, se guardarán en
`priv/static`. Los recursos estáticos que necesiten ser procesados (JavaScript y CSS
por ejemplo) irán en `web/static`. Al procesarlos, se moverán a `priv/static` minificados,
optimizados, transpilados, foobareados,...

Como en proyectos típicos de Elixir, el código irá en `lib`. El código dentro de `lib` no
será recompilado en cada petición, mientras que el código dentro de `web` se recompila en
cada una de ellas.

Eso está hecho a posta. En `web` contiene cualquier cosa cuyo estado tiene una vida útil
igual a una petición. `lib`, por el contrario, tiene código compartido y cualquier cosa
que necesite mantener el estado entre diferentes peticiones.

## Rutas (para el router)

Nada más crear el proyecto, se crea una ruta en el fichero de definición de rutas,
`web/router.ex`, con esta pinta:

    get "/", PageController, :index

Esa ruta la podemos interpretar así: cuando el usuario visita la página `/` (http://localhost:4000/),
la petición será procesada por el controlador `HelloPhoenix.PageController` (que lo podemos
encontrar en `web/controllers/page_controller.ex`) y su función `index`. Se puede profundizar
en el enrutamiento en http://www.phoenixframework.org/docs/routing

Ahora, vamos a añadir nuestra primera ruta (luego crearemos el controlador):

    get "/hello", HelloController, :index

## Controladores

Un controlador es un módulo Elixir, y una acción es una función dentro de ese módulo.

Nuestro controlador para responder a la ruta que hemos definido anteriormente lo crearemos
en `web/controllers/hello_controller.ex`:

```
defmodule HelloPhoenix.HelloController do
  use HelloPhoenix.Web, :controller

  def index(conn, _params) do
    render conn, "index.html"
  end
end
```

Lo importante de la acción `index` es 

    render conn, "index.html"

que indica a Phoenix a buscar una plantilla llamada `index.html.eex` y renderizarla.
Phoenix buscará la plantilla en un directorio bajo el nombre del controlador:
`web/templates/hello`.

## Una nueva (pero vacía) Vista

Las vistas tienen dos trabajos:

1. Renderizar las plantillas
2. Hacer de capa de presentación para proveer de los datos necesarios a las plantillas para
que se rendericen correctamente. Las vistas tendrían cierta lógica, para hacer que las
plantillas fueran lo más sencillas posible

Nuestra primera Vista no necesita nada de especial, pero necesitamos una vista si
queremos renderizar una plantilla. La mínima vista sería:

```
defmodule HelloPhoenix.HelloView do
  use HelloPhoenix.Web, :view
end
```

Por cierto, los nombres de las vistas y los controladores deben coincidir: `<nombre>Controller` y
`<nombre>View`. Y la ruta donde guardar la vista seguro que la adivinaís: `web/views/hello_view.ex`

## Por fin, la Plantilla

Phoenix usa un motor de plantillas llamado `EEx`. Y las plantillas se guardan en el
directorio `web/templates`. Por razones de organización, crearemos un directorio con
el nombre de nuestro controlador, y crearemos nuestra plantilla en:
`web/templates/hello/index.html.eex`:

```
<div class="jumbotron">
  <h2>Hello World, from Phoenix!</h2>
</div>
```

Ahora que tenemos todas las piezas en su lugar (ruta, controlador, vista y plantilla) ya
podemos visitar nuestra URL: http://localhost:4000/hello

# Otra página

## La ruta

Añadimos la ruta: `get "/hello/:messenger", HelloController, :show`. De esta forma, un
mapa con la clave `"messenger"` (una cadena) será pasado a la función `show` del controlador. El valor
para esa clave dependerá de la URL que visite el usuario: `/hello/Dubi`, `/hello/Einar`,
darían valores distintos: `Dubi` y `Einar`.

## La acción

La acción invocará a la vista pasándole el parámetro recibido:

    def show(conn, %{"messenger" => msgr}) do
        render conn, "show.html", messenger: msgr
    end

Del parámetro `params` de la acción extraemos el `messenger` mediante pattern matching.
A la vista le estamos pasando una lista de parámetros, donde `:messenger` tendrá el
valor extraído del mapa de parámetros.

## La plantilla

La plantilla irá definida en `web/templates/hello/show.html.eex`. `hello` por ser el
nombre del controlador, `show.html.eex` por ser el nombre de la plantilla que el
controlador quiere renderizar.

# Enrutado

Una documentación algo exhaustiva sobre enrutado en Phoenix se puede encontrar en la
URL: http://www.phoenixframework.org/docs/routing

La definición de rutas suelen estar en el fichero `web/router.ex`. El router tiene dos
funcionalidades principales: las `pipelines` y los `scopes`. Los veremos más adelante.

En los `scopes` es donde se definen las rutas, por ejemplo:

    get "/", PageController, :index

`get` es una macro que se expande por una definición de la función `match/3`:

    def match("/", PageController, :index) do

Así, cada ruta que definamos con `get` será una nueva definición de `match/3`, que
se evalúa como cualquier otra función de Elixir con varias definiciones. Por orden,
de arriba a abajo, pattern matching para decidir qué definición ejecutar, solo una
definición se ejecuta,...

Con el comando `mix phoenix.routes` podemos ver una lista de las rutas que
están definidas.

La macro `resources` crea varias definiciones de `match/3`. De hecho, crea las
definiciones típicas para gestionar recursos REST. Por ejemplo,
`resources "/users", UserController` expandería a:

    user_path  GET     /users           HelloPhoenix.UserController :index
    user_path  GET     /users/:id/edit  HelloPhoenix.UserController :edit
    user_path  GET     /users/new       HelloPhoenix.UserController :new
    user_path  GET     /users/:id       HelloPhoenix.UserController :show
    user_path  POST    /users           HelloPhoenix.UserController :create
    user_path  PATCH   /users/:id       HelloPhoenix.UserController :update
               PUT     /users/:id       HelloPhoenix.UserController :update
    user_path  DELETE  /users/:id       HelloPhoenix.UserController :delete

## Path helpers: http://www.phoenixframework.org/docs/routing#section-path-helpers

Dinámicamente, se añaden ciertas funciones al módulo `HelloPhoenix.Router.Helper`.
Por ejemplo, si nuestro controlador se llama `page`, tendremos disponibles las
funciones `page_path` y `page_url`:

    iex> HelloPhoenix.Router.Helpers.page_path(HelloPhoenix.Endpoint, :index)
    "/" 
    iex(3)> user_url(Endpoint, :index)
    "http://localhost:4000/users"

`page_path` nos dice el path para una acción. `page_url` nos dice la URL completa.

Esto puede ser muy útil para construir enlaces en las plantillas. Por ejemplo,
para enlazar de un controlador a otro, o de una lista a un detalle de un
elemento.

## Otros conceptos

- Recursos anidados: http://www.phoenixframework.org/docs/routing#section-nested-resources
- Rutas con su propio scope: http://www.phoenixframework.org/docs/routing#section-scoped-routes

## Pipelines: 

Las pipelines (http://www.phoenixframework.org/docs/routing#section-pipelines)
son un grupo de *plugs* que se ejecutarán uno detrás de otro y a las que se han
dado un orden.

Un [plug](http://www.phoenixframework.org/docs/overview#section-plug) es como
un middleware o algo así, son tareas que se ejecutan para transformar una
petición del usuario: retornan ficheros estáticos, escriben logs, hacen
comprobaciones de seguridad,...

# Rutas para canales

Los [canales](http://www.phoenixframework.org/docs/channels) gestionan mensajes
entrantes y salientes emitidos a través de un socket y que tratan sobre un
determinado tema. Las
[rutas para los canales](http://www.phoenixframework.org/docs/routing#section-channel-routes)
relacionan peticiones por socket y tema, para que sean gestionados por el canal
adecuado.

Los manejadores de socket se montan sobre el *endpoint*, que se define en
`lib/hello_phoenix/endpoint.ex`:

    defmodule HelloPhoenix.Endpoint do
      use Phoenix.Endpoint

      socket "/socket", HelloPhoenix.UserSocket
      ...
    end

Cada socket, puede gestionar varios canales. Por ejemplo,
`web/channels/user_socket.ex` sería parecido a:

    defmodule HelloPhoenix.UserSocket do
      use Phoenix.Socket

      channel "rooms:*", HelloPhoenix.RoomChannel, via: [Phoenix.Transports.WebSocket]
      channel "foods:*", HelloPhoenix.FoodChannel
    end

Cada canal puede usar varios transportes: WebSockets o/y Long-Polling. O se
puede restringir con `via:`, como antes.

# Canales: http://www.phoenixframework.org/docs/channels

## Partes móviles

- Manejadores de sockets: básicamente gestionan la conexión. Suele haber una sola conexión que se multiplexa entre varios canales. Un ejemplo es `web/channels/user_socket.ex`, que se crea al crear un nuevo proyecto Phoenix.
- Rutas para canales: se definen en los manejadores de sockets. Hacen coincidir tópicos con módulos donde se implementan los canales.

    channel "room:*", HelloPhoenix.RoomChannel

- Canales: gestiona eventos de clientes. Son como Controladores, pero los Canales son bidireccionales y la conexión persiste más allá de una sola petición. Cada Canal implementará alguno de los callbacks `join/3`, `terminate/2`, `handle_in/3` o `handle_out/3`.
- PubSub: gestionan los mensajes a pasar por los canales. Normalmente, no les prestaremos mucha atención. Los Canales los utilizan por debajo para el paso de mensajes.
- Mensajes: son definidos en un struct con las siguientes claves: `topic`, `event`, `payload` y `ref`.
- Tópicos: son identificadores en forma de cadena
- Transportes
- Adaptadores de transportes
- Librerías de clientes: Phoenix viene con una libraría para JavaScript, pero hay clientes para Android, iOS,...

## Creando una aplicación de chat

La configuración comienza en el endpoint. Al crear una aplicación Phoenix, el endpoint lo podemos encontrar en `lib/hello_phoenix/endpoint.ex`. Buscamos una línea donde se defina cómo se va a manejar un socket:

```
defmodule HelloPhoenix.Endpoint do
  use Phoenix.Endpoint, otp_app: :hello_phoenix

  socket "/socket", HelloPhoenix.UserSocket
  # ...
end
```

El siguiente paso es localizar el módulo `HelloPhoenix.UserSocket`, en `web/channels/user_socket`, donde deberemos descomentar la línea que configura el identificador para el canal `room:*`:


```
defmodule HelloPhoenix.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "room:*", HelloPhoenix.RoomChannel
  #...
```

Cualquier cliente que mande un mensaje con `romm:<lo que sea>` como tópico, será manejado por `RoomChannel`.

### Uniéndose (autorizando) a un canal

Para aceptar o denegar conexiones a un canal, debemos implementar `join/3` en `RoomChannel`:


```
defmodule HelloPhoenix.RoomChannel do
  use Phoenix.Channel

  def join("room:lobby", _message, socket) do
    {:ok, socket}
  end
  def join("room:" <> _private_room_id, _params, _socket) do
    {:error, %{reason: "unauthorized"}}
  end
end
```

Estamos aceptando mensajes dirigidos a `room:lobby` mientras que estamos rechazando mensajes a cualquier otra *habitación*.

### Un cliente JavaScript

Si usamos [Brunch](http://www.phoenixframework.org/docs/static-assets) (se usa por defecto), tendremos un cliente básico de WebSockets usando la implementación que viene en Phoenix. Podemos consultar el código JavaScript en `web/static/js/socket.js`.

```
//...
socket.connect()

// Now that you are connected, you can join channels with a topic:
let channel = socket.channel("room:lobby", {})
channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

export default socket
```

Tenemos que asegurarnos que ese fichero se importa en nuestra aplicación. Para ello, descomentar la línea que hace refrencia al fichero en `web/static/js/app.js`.

En este punto, si visitamos http://localhost:4000, deberíamos ver un mensaje de éxito por la consola del navegador. Deberíamos haber sido capaces de conectarnos al WebSocket.

Ahora, en `web/templates/page/index.html.eex` añadiremos un bloque donde mostrar los mensajes recibidos y una caja de texto donde escribirlos.

```
<div id="messages"></div>
<input id="chat-input" type="text"></input>
```

Ahora, añadamos algún listener de eventos en `web/static/js/socket.js` para poder enviar mensajes.

```
//...
let channel           = socket.channel("room:lobby", {})
let chatInput         = document.querySelector("#chat-input")
let messagesContainer = document.querySelector("#messages")

chatInput.addEventListener("keypress", event => {
  if(event.keyCode === 13){
    channel.push("new_msg", {body: chatInput.value})
    chatInput.value = ""
  }
})

channel.join()
  .receive("ok", resp => { console.log("Joined successfully", resp) })
  .receive("error", resp => { console.log("Unable to join", resp) })

export default socket
```

El código anterior escucha eventos `keypress`, y cuando pulsamos `Enter` envía un mensaje por el canal, con `channel.push`. El nombre del evento, del mensaje que enviamos es `new_msg`, y debemos enviar un objeto con un `body`.

Ahora, escucharemos mensajes y los escribiremos en el bloque adecuado:

```
channel.on("new_msg", payload => {
  let messageItem = document.createElement("li");
  messageItem.innerText = `[${Date()}] ${payload.body}`
  messagesContainer.appendChild(messageItem)
})
```

### Mensajes entrantes en el servidor

Los mensajes entrantes se manejan en `handle_in/3`, de `RoomChannel`. Podemos hacer pattern matching de los nombres de los eventos, en nuestro caso `new_msg`. En nuestro caso, notificaremos a todos los clientes conectados a `room:lobby` mediante `broadcast!/3`.

```
def handle_in("new_msg", %{"body" => body}, socket) do
  broadcast! socket, "new_msg", %{body: body}
  {:noreply, socket}
end
```

`broadcast!/3` notifica a todos los clientes del canal llamando a `handle_out/3` de cada uno de ellos. De esta forma podemos controlar qué y cómo salen los mensajes a cada cliente. `handle_out/3` envía los mensajes a los clientes haciendo `push` al socket con el mensaje en particular.


```
def handle_out("new_msg", payload, socket) do
  push socket, "new_msg", payload
  {:noreply, socket}
end
```




















