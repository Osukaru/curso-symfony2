# Creando páginas con Symfony 2 #

En el capítulo anterior hemos creado ya nuestro primer Bundle MDW\DemoBundle. Aquí crearemos todos los ejemplos para este manual y en este capítulo vamos a ver como crear las páginas con Symfony 2.
Pasos para crear nuestras páginas

Para crear un página tenemos que tener en cuenta tres pasos:

    Asignación de una ruta: Una dirección URL asignada a la página para que el controlador frontal la pueda acceder.
    Creación de una acción (action): La lógica necesaria para la página. Corresponde al Controlador en arquitectura MVC.
    Creación de la plantilla (template): La estructura de nuestra página. Corresponde a la Vista en arquitectura MVC.

Tomaremos el ejemplo que fue creado al generar nuestro MDWDemoBundle y lo estudiaremos para entender estos tres pasos.
1. Asignación de una ruta

Las rutas se refieren a la dirección URL que utilizará el controlador frontal para acceder a nuestra página. Dichas rutas se especifican en el archivo de configuración app/config/routing.yml. Este paso es sumamente importante ya que de lo contrario la página existirá pero no podrá ser accedida.

Dentro de este archivo podríamos crear las rutas directamente, pero para no mezclar las rutas de nuestro Bundle con las de otros, podemos crear las rutas en un archivo dentro de nuestra carpeta MDW\DemoBundle y de esta manera logramos independencia y portabilidad. Para hacer esto, tendremos que importar el archivo routing.yml de nuestro bundle dentro del app\config\routing.yml que básicamente es el archivo de rutas genérico para todo el proyecto.

Al crear nuestro Bundle con el script “console”, en el paso 7 explicado en el capítulo anterior es justamente lo que se hace automáticamente agregando el siguiente código de importación en el archivo app\config\routing.yml:

MDWDemoBundle:
  resource: "@MDWDemoBundle/Resources/config/routing.yml"
  prefix: /

La primera línea es simplemente un texto identificador para la importación, que por convención podríamos usar el mismo identificador de nuestro Bundle. Abajo, con espacios en blanco definimos la clave “resource” que apunta a un archivo externo y haciendo uso de nuestro identificador @MDWDemoBundle que apunta a src\MDW\DemoBundle, le decimos que use el archivo ubicado en nuestra carpeta /Resources/config/routing.yml.

La segunda clave a definir es el “prefix” que indica con / que a partir del controlador frontal se crearán nuestras rutas.
Nota
Si no conoces la forma de uso de los archivos YAML puedes ver un poco de información aquí.
Capturar a Evernote

Una vez que tenemos ya nuestro archivo importado lo abriremos y veremos el siguiente contenido de ejemplo que se creó al generar nuestro Bundle con el generador de Symfony:

MDWDemoBundle_homepage:
  pattern: /hello/{name}
  defaults: { _controller: MDWDemoBundle:Default:index }

El primer texto es nuevamente un texto identificador para esta ruta. Los identificadores no pueden repetirse con ninguna otra ruta del proyecto, en nuestro Bundle o en cualquier otro.

Abajo, en lugar de usar la clave resource para importar un archivo, definimos el “pattern” (patrón) que indica la dirección URL a usar para esta ruta. Dentro de una ruta cuando usamos llaves {name}, indicamos que será un parámetro. Como la ruta indica la dirección que el Controlador Frontal utilizará, estamos diciendo que podremos acceder a la página escribiendo http://localhost/Symfony/web/app_dev.php/hello/Jhon donde:

    http://localhost: Dirección del servidor.
    /app_dev.php: corresponde al controlador frontal, que podríamos utilizar el de desarrollo o producción.
    Finalmente /hello/Jhon: indica la ruta que acabamos de crear, donde {name} lo podremos reemplazar por el valor del parámetro que queramos.

La segunda clave obligatoria es “defaults” que utilizando llaves simulamos un array asociativo por lo que la clave “_controller” indica cual será el controlador que contendrá la lógica de la página. Para no escribir la ruta completa del archivo (\src\MDW\DemoBundle\Controller\DefaultController.php) utilizamos una forma abreviada o “dirección lógica” que está compuesta de tres partes: IdentificadorDelBundle:Controller:Action

    IdentificadorDelBundle: En este caso nuestro identificador es MDWDemolBundle.
    Controller: El nombre de la clase que contendrá los actions (acciones). Estas clases se encuentran en la carpeta “Controller” de nuestros Bundles.
    Action: Representado por un método de la clase arriba mencionada. Este método contendrá la lógica de negocios para nuestra página y se ejecutará antes de mostrar la página.

Entendiendo los pasos para crear nuestras páginas, vemos que nuestra ruta de ejemplo apunta a MDWDemoBundle:Default:index, lo que indica que la acción a ejecutarse al ingresar a nuestra ruta se encuentra en MDWDemoBundle en una clase DefaultController programado en un método indexAction.

Nota: Hablaremos más sobre el sistema de rutas de Symfony en el capítulo 4
2. Creación de un controlador

Así como vimos en la sección anterior, una ruta apunta a un controlador usando la clave defaults

defaults: { _controller: MDWDemoBundle:Default:index }

Si abrimos el archivo mencionado veremos lo siguiente:

//-- \src\MDW\DemoBundle\Controller\DefaultController.php
public function indexAction($name)
{
    return $this->render('MDWDemoBundle:Default:index.html.twig', array('name' => $name));
}

Vemos que nuestro método recibe el parámetro $name que debe coincidir con el escrito en nuestra ruta /hello/{name}.

No hay lógica para esta página ya que es un simple ejemplo, por lo que simplemente termina haciendo un return del resultado de un método render() que se encarga de llamar a la plantilla (paso siguiente) y pasarle datos por medio del segundo argumento, un array asociativo. En caso de que no necesitemos enviar variables a la vista simplemente enviamos un array vacío.

Como primer argumento del método render(), enviamos el “nombre lógico” del template MDWDemoBundle:Default:index.html.twig que indica donde está la plantilla. Las plantillas se encuentran organizadas dentro de la carpeta “Resources\views” de nuestros Bundles, en este caso src\MDW\DemoBundle\Resources\views\Default\index.html.twig.
3. Creación de la plantilla

Para la plantilla estamos utilizando el framework Twig y si abrimos el archivo mencionado vemos que simplemente utilizamos la variable $name como si existiera, esto lo podemos hacer porque al momento de llamar a la plantilla desde el controlador enviamos esta variable para que exista mágicamente usando el método render(): array(‘name’ => $name)

Si abrimos el archivo src\MDW\DemoBundle\Resources\views\Default\index.html.twig vemos que solo contiene como ejemplo: Hello {{ name }}!

Como habíamos hablado en el capítulo 1, Twig permite separar el código PHP del HTML por lo que por ejemplo en lugar de escribir:

Hello <?php echo htmlentities($name) ?>!

podemos simplemente poner:

Hello {{ name }}!

donde {{ $var }} significa que la variable que este dentro será impresa como si utilizaramos un echo.
Creemos nuestra primera página de ejemplo

Siguiendo las mismas instrucciones vistas en los puntos anteriores, crearemos una página de ejemplo para mostrar un listado de artículos de un blog en una tabla. Supondremos que los artículos son extraídos de una base de datos pero como todavía no hemos llegado a hablar de Doctrine los obtendremos de un array.
Ejemplo 1

Lo primero que tenemos que pensar es una dirección URL para acceder a la página, luego crearemos el action que se procesará al ejecutarse la petición de la URL y finalmente usaremos los datos devueltos por nuestro action dentro del template para mostrar la tabla de artículos.
Paso 1

Dentro del archivo src\MDW\DemoBundle\Resources\config\routing.yml de nuestro Bundle de ejemplo crearemos la ruta correspondiente agregando el siguiente código:

articulos:
  pattern: /articulos
  defaults: { _controller: MDWDemoBundle:Default:articulos }

Como habíamos mencionado la primera línea corresponde al identificador de la ruta al que llamaremos “articulos”. Este nombre será usado para mostrar los links a nuestras páginas pero esto lo veremos con más detalles más adelante.

A continuación definimos el pattern que será lo que agregaremos a la dirección del controlador frontal. Para este caso definimos que si nuestra dirección de acceso al controlador frontal es “http://localhost/Symfony/web/app_dev.php” agregaremos “/articulos” para acceder a nuestra página teniendo como URL: http://localhost/Symfony/web/app_dev.php/articulos.

Por último definimos el action que se ejecutará al llamar a esta ruta: MDWDemoBundle:Default:articulos lo que apuntaría al método articulosAction() de la clase DefaultController.php que existe dentro de nuestro MDWDemoBundle.
Paso 2

Ahora que ya tenemos nuestra ruta apuntando a nuestro action nos encargaremos de crear el método correspondiente que será ejecutado y que contendrá la lógica del negocio, que para este caso es muy simple. Dentro de nuestra clase \src\MDW\DemoBundle\Controller\DefaultController.php agregaremos el método (action) correspondiente que deberá llamarse articulosAction():

public function articulosAction()
{
    //-- Simulamos obtener los datos de la base de datos cargando los artículos a un array
    $articulos = array(
        array('id' => 1, 'title' => 'Articulo numero 1', 'created' => '2011-01-01'),
        array('id' => 2, 'title' => 'Articulo numero 2', 'created' => '2011-01-01'),
        array('id' => 3, 'title' => 'Articulo numero 3', 'created' => '2011-01-01'),
    );
    return $this->render('MDWDemoBundle:Default:articulos.html.twig', array('articulos' => $articulos));
}

Una vez que tenemos los datos dentro de nuestro array, por medio del método ->render() llamamos a nuestro template con el primer argumento y con el segundo pasamos los datos que deberán existir dentro del mismo.
Paso 3

El último paso será usar la información que nuestro action nos provee para generar nuestra vista y mostrarla al usuario. Para esto, crearemos un archivo dentro de la carpeta \Resources\views\Default de nuestro bundle con el nombre articulos.html.twig y con la ayuda de Twig recorreremos nuestro array de artículos para mostrar nuestra tabla de una manera muy sencilla:

<h1>Listado de Articulos</h1>
<table border="1">
    <tr>
        <th>ID</th>
        <th>Titulo</th>
        <th>Fecha de Creacion</th>
    </tr>
    {% for articulo in articulos %}
    <tr>
        <td>{{articulo.id}}</td>
        <td>{{articulo.title}}</td>
        <td>{{articulo.created}}</td>
    </tr>
    {% endfor %}
</table>

Twig nos provee mucha habilidad para manipular los datos sin escribir código PHP pudiendo acceder a las claves de nuestro array por medio de articulo.id o articulo['id'] indistintamente.

Con esto ya tenemos una tabla que muestra artículos haciéndolo en tres pasos bien concisos y donde cada uno tiene su responsabilidad. Para ingresar a la página podemos escribir esta URL: http://localhost/Symfony/web/app_dev.php/articulos
Ejemplo 2

Ahora que ya hemos visto como mostrar los datos de nuestro array supongamos que queremos otra página que reciba como parámetro GET un id de artículo y nos muestre sus datos. Para esto nuevamente sigamos los mismo pasos:
Paso 1

Para este caso crearemos una ruta llamada “articulo” que recibirá un parámetro {id} y hará que se ejecute el action MDWDemoBundle:Default:articulo

articulo:
  pattern: /articulo/{id}
  defaults: { _controller: MDWDemoBundle:Default:articulo }

Paso 2

Para simular la base de datos volveremos a tener nuestro array y buscaremos el id recibido. Como recibimos un parámetro GET podemos recibirlo directamente como argumento de nuestro método:

public function articuloAction($id)
{
    //-- Simulamos obtener los datos de la base de datos cargando los artículos a un array
    $articulos = array(
        array('id' => 1, 'title' => 'Articulo numero 1', 'created' => '2011-01-01'),
        array('id' => 2, 'title' => 'Articulo numero 2', 'created' => '2011-01-01'),
        array('id' => 3, 'title' => 'Articulo numero 3', 'created' => '2011-01-01'),
    );

    //-- Buscamos dentro del array el ID solicitado
    $articuloSeleccionado = null;
    foreach($articulos as $articulo)
    {
        if($articulo['id'] == $id)
        {
            $articuloSeleccionado = $articulo;
            break;
        }
    }

    //-- Invocamos a nuestra nueva plantilla, pasando los datos
    return $this->render('MDWDemoBundle:Default:articulo.html.twig', array('articulo' => $articuloSeleccionado));
 }

Paso 3

Por último mostramos los datos de nuestro artículo devuelto por nuestro controlador:

<h1>Articulo con ID {{articulo.id}}</h1>
<ul>
    <li>Titulo: {{articulo.title}}</li>
    <li>Fecha de creacion: {{articulo.created}}</li>
</ul>

Para acceder a este ejemplo podemos escribir la URL: http://localhost/Symfony/web/app_dev.php/articulo/1. Donde el parámetro “1″ sería el id del artículo que queremos ver.
Resumen Final

En este capítulo hemos visto los tres pasos básicos para crear nuestras páginas y hemos llegado a la conclusión de que primeramente se accede a una URL definida por nuestras rutas en el archivo routing.yml, luego el sistema de ruteo deriva la invocación a un método de un controlador llamado action en donde se procesan los datos y se llama a la vista pasándole como variables los datos que necesitan ser mostrados. Una vez en la vista usaremos el framework Twig para facilitar la visualización de la información.

En los siguientes 3 capítulos hablaremos un poco más detalladamente sobre la potencia del sistema de ruteo de Symfony2 y del framework Twig.
