# Trabajando con el Controlador en Symfony2 #

En el capítulo anterior aprendimos a crear las rutas hacia nuestros controladores, como algunos ya saben el “Controlador” (o Controller) es la piedra angular de la arquitectura MVC, es lo que nos permite enlazar el modelo de la vista y básicamente donde se construye la aplicación.
Pero, ¿Que hace exactamente un controlador?

Bajo la filosofía de Symfony2 el concepto es simple y preciso: “El controlador recibe la petición (Request) y se encarga de crear y devolver una respuesta (Response)”; a simple vista pensarás “¡el controlador hace más que eso!” sí, pero a Symfony2 sólo le importa eso, lo que implementes dentro de él depende de tu lógica de aplicación y negocio*, al final un controlador puede:

    Renderizar una vista.
    Devolver un contenido tipo XML, JSON o simplemente HTML.
    Redirigir la petición (HTTP 3xx).
    Enviar mails, consultar el modelo, manejar sesiones u otro servicio.

Te darás cuenta de que todas las funciones anteriores terminan siendo o devolviendo una “Respuesta” al cliente, que es la base fundamental de una aplicación basada en el ciclo Web (Petición -> acción -> Respuesta). En este capítulo nos concentraremos en las funciones básicas que puedes hacer en el controlador, su estructura básica y las funciones más comunes.
Nota
*Las buenas prácticas indican que la lógica de negocios debe residir en el modelo (modelos gordos, controladores flacos).
Capturar a Evernote
Definiendo el controlador

Los controladores en Symfony2 se declaran como archivos con el sufijo Controller “MicontroladorController.php” en el directorio Mibundle/Controller dentro de nuestro Bundle, quedando así nuestro ejemplo: “src/MDW/DemoBundle/Controller/DemoController.php”, veamos el código:

<?php
// src/MDW/DemoBundle/Controller/DefaultController.php
namespace MDW\DemoBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;

class DefaultController extends Controller
{
    public function indexAction()
    {
        //$response = new \Symfony\Component\HttpFoundation\Response('Hola mundo!!!');
        return new Response('Hola mundo!!!');
    }
}

Vemos que el controlador es una clase que extiende de la clase base Controller (Symfony\Bundle\FrameworkBundle\Controller\Controller), pero en realidad esta clase es un paquete de controladores, ¡sí, es confuso!: nuestros verdaderos controladores son en realidad las funciones (acciones) que contiene esta clase, cuyo sufijo siempre sea Action.

En la mayoría de los FW el Controlador contiene varias Acciones y si nos damos cuenta es en las Acciones en donde aplicamos la verdadera lógica del controlador, consideralo como un cliché; además Symfony2 no requiere que la clase controlador extienda de Controller, la clase base Controller simplemente nos provee de atajos útiles hacia las herramientas del núcleo del framework, como el DI (inyección de dependencias), por eso se recomienda su uso.

Como primera norma tenemos que asignar el “namespace” a nuestro controlador (línea #3), para asegurar que el mecanismo de autoload pueda encontrar nuestro controlador y con ello el Routing (entre otros) pueda hallarlo eficientemente, luego vemos una serie de declaraciones use que nos sirven para importar los diferentes espacios de nombres y clases que necesitamos para trabajar.

Namespaces: Es sabido que para algunos este cambio es brusco, pero tiene su objetivo claro el cual otros FW siguen y dentro de poco otros seguirán, entre ellos evitar conflictos con los nombres de clases, pero bueno ¿te frustra escribir tanto código?: tranquilo, que si usas un IDE inteligente como Netbeans 7 con solo hacer nombre a la clase puede generarte automáticamente la cadena de referencia completa (como puedes notar en la línea comentada #12), claro está que utilizar use es más elegante y forma parte de las buenas prácticas.

Al final vemos que simplemente creamos nuestra respuesta (objeto Response) y lo devolvemos, eso es lo básico de nuestro controlador, más adelante expondremos los detalles.
Renderizando (generando) Vistas

Prácticamente es la tarea más común en cada controlador en la cual hacemos la llamada al servicio de Templating, Symfony2 por defecto utiliza Twig y los detalles se expondrán en el próximo capitulo, por ahora nos concentraremos en la forma de renderizar nuestras vistas, para eso añadimos la siguiente acción en nuestro BlogController que nos corresponderá a la ruta “blog_index” definida en el capítulo anterior:

// src/MDW/DemoBundle/Controller/BlogController.php
// ...
public function showAction($slug)
{
    $articulo = $slug; // (suponiendo que obtenemos el artículo del modelo con slug)
    return $this->render('MDWDemoBundle:Blog:show.html.twig', array(
        'articulo' => $articulo
    ));
}
// ...

La función $this->render() acepta 2 parámetros: el primero es un patrón para encontrar la plantilla [Bundle]:[Controller]:template.[_format].template_engine y el segundo el array para pasar variables a la plantilla, devolviendo un objeto Response().

Como apreciarás en el patrón, Symfony busca la correspondiente plantilla en el directorio MyBundle/Resoruces/views, según su propia estructura interna detallada en el próximo capítulo de la Vista. Como indicamos en el capítulo anterior sobre Routing, los parámetros de nuestro Controlador coinciden con los comodines definidos en la ruta con el cual obtendremos eficientemente estos datos.

En realidad la función $this->render() es un atajo de nuestra clase base Controller que renderiza la plantilla y crea el Response, al final tenemos estas diferentes formas para hacer el mismo proceso:

Otras formas de renderizar plantillas:

// 2da forma- atajo que obtiene directamente el contenido renderizado:
$content = $this->renderView('MDWDemoBundle:Blog:show.html.twig', array('articulo' => $articulo));
return new Response($content);

// 3ra forma- obteniendo el servicio templating por DI (inyección de dependencias):
$templating = $this->get('templating');
$content = $templating->render('MDWDemoBundle:Blog:show.html.twig', array('articulo' => $articulo));
return new Response($content);

Como puedes apreciar, los atajos de la clase base Controller son de mucha ayuda, y si en dado caso necesitas mayor flexibilidad puedes optar por la 2da y 3ra forma. ;-)
Obteniendo datos de la Petición (Request)

Ya sabes que los parámetros del Controlador coinciden con los comodines definidos en las Rutas, pero existe otra forma de acceder a las variables POST o GET pasadas a nuestra petición, y como es obvio lo proporciona el objeto Request(), existen varias formas de obtener el objeto Request, incluso como en Symfony1 en Symfony2 lo puedes pasar como parámetro del controlador, como el sistema de Routing empata éstos con los comodines el orden no importa:

// src/MDW/DemoBundle/Controller/BlogController.php
// ...
use Symfony\Component\HttpFoundation\Request; //es necesario para importar la clase
//...
public function showAction(Request $peticion, $slug)
{
    $articulo = $peticion->get('slug'); // otra forma para obtener comodines, GET o POST
    $metodo = $peticion->getMethod(); //obtenemos si la petición fue por GET o POST
    return $this->render('MDWDemoBundle:Blog:show.html.twig', array(
        'articulo' => $articulo
    ));
}

Vemos que con el objeto Request obtenemos por completo los datos de nuestra petición, además la clase base Controller dispone de un atajo para obtener el Request sin necesidad de pasarlo como parámetro y de esa forma nos ahorraremos la importación por use:

$peticion = $this->getRequest();
//lo que es lo mismo que con DI:
$peticion = $this->get('request');

Redirecciones

En diversas oportunidades nos vemos obligados en hacer una redirección, ya sea por haber procesado un POST, o dependiendo de la lógica que apliquemos en el Controlador, si en dado caso tengamos que redirigir a HTTP 404 por un registro en el modelo no encontrado, etc.

public function indexAction()
{
    return $this->redirect($this->generateUrl('homepage'));
    //return $this->redirect($this->generateUrl('homepage'), 301);
}

Como notarás la función $this->redirect() acepta como primer parámetro una string Url (la función $this->generateUrl() nos permite generar la url desde una ruta definida en Routing) y por defecto realiza una redirección HTTP 302 (temporal), en el segundo parámetro puedes modificarla para hacer una 301 (permanente).

En el caso de redirigir a HTTP 404, podremos causar una Excepción de esta forma que Symfony2 intercepta internamente:

public function indexAction()
{
    $producto = false;// suponiendo que nuestro modelo devuelva false al no encontrarlo
    if (!$producto) {
        throw $this->createNotFoundException('No existe el producto');
    }
    return $this->render(...);
}

En ocasiones necesitarás pasar la petición a otro controlador internamente sin necesidad de redirecciones HTTP, en este caso el Forwarding es tu alternativa:

public function indexAction($name)
{
    $respuesta = $this->forward('MDWDemoBundle:Blog:index', array(
        'slug'  => $name
    ));
    // puedes modificar directamente la respuesta devuelta por el controlador anterior.
    return $respuesta;
}

Como podrás notar, en el primer parámetro de la función $this->forward() acepta el mismo formato que utilizamos en el Routing para hallar el Controlador, y como segundo parámetro un array con los parámetros de dicho Controlador, cabe mencionar que el mismo ha de ser un controlador normal y no es necesario que tenga una ruta en Routing definida.
Resumen Final

Como apreciamos en éste capítulo, los Controladores en Symfony2  requieren el sufijo Controller para ser hallados por el FW, además de resaltar que la verdadera lógica del controlador reside en sus Acciones las cuales son las funciones internas de los mismos declaradas con el sufijo Action, además de ello vimos las funcionalidades básicas del controlador, como lo son acceder a las variables de la petición “Request”, Redireccionar, pasar a otros controladores sin redirecciones (Forwarding) y el renderizado de las vistas.

En el próximo capitulo “La Vista y Twig” veremos la flexibilidad que nos permite este maravilloso sistema de plantillas, además de enumerar las operaciones básicas de la Vista en Symfony2 y como aprovechar el sistema de Herencia de plantillas para lograr una estructura de layouts dinámicos y reutilizables.