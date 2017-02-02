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







