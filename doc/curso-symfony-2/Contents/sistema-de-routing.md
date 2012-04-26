# Sistema de Routing # {#sistema-de-routing}

Sistema de Routing

Una de las necesidades más comunes en el desarrollo de Sitios profesionales es implementar URLs amigables, así convertimos algo como /index.php?articulo=1 por algo más cómodo y agradable a la vista del usuario: /blog/introduccion_symfony2.htm.

El Routing de Symfony2 nos brinda un sistema de enrutado muy dinámico que deja concentrarnos en el desarrollo de los “caminos” hacia nuestra aplicación, además es bidireccional, lo cual nos permite cambiar la ruta /blog/introduccion_symfony2.htm por /noticia/introduccion_symfony2.htm editando sólo la definición de la ruta y así evitarnos la tarea de buscar todas las referencias internas hacia ella en nuestra aplicación para hacer el cambio.

El objetivo de éste capítulo es comprender el funcionamiento básico del sistema de Routing de Symfony2, crear e importar rutas, así como configuraciones básicas para permitir rutas flexibles. Nos enfocaremos en la configuración en formato YAML por ser simple y fácil de entender, recuerda que Symfony 2 es un Fw altamente configurable y que puedes utilizar como configuración: XML, PHP y Annotations.
Funcionamiento del Routing

En la arquitectura del Modelo MVC el encargado de manejar las peticiones Web es el Controlador (Controller), como cualquier aplicación consiste en varios Controllers y necesitamos un punto de acceso centralizado para distribuir las peticiones de nuestra aplicación a los Controllers indicados, a ese punto lo llamamos el Controlador frontal (Front Controller) que generalmente corresponde al archivo raíz de nuestra web, es decir el app.php o index.php (dependiendo de la configuración del servidor HTTP) y para ello necesitamos redirigir a éste todas las peticiones por medio de un .htaccess (en el caso de Apache):

# web/.htaccess

RewriteEngine On
RewriteCond %{REQUEST_FILEname} !-f
RewriteRule ^(.*)$ app.php [QSA,L]

En Symfony2 el Front Controller se encarga de cargar el kernel del Framework, el cual recibe nuestra petición HTTP (request) y pasa la URL al sistema de Routing, donde es procesada (comparada con las rutas definidas) y se ejecuta el Controller definido; este es el comportamiento básico del Routing: empatar URL y ejecutar los Controllers.

Código del Front Controller:

<?php
// web/app.php

require_once __DIR__.'/../app/bootstrap.php.cache';
require_once __DIR__.'/../app/AppKernel.php';
//require_once __DIR__.'/../app/AppCache.php';

use Symfony\Component\HttpFoundation\Request;

$kernel = new AppKernel('prod', false);
$kernel->loadClassCache();
//$kernel = new AppCache($kernel);
$kernel->handle(Request::createFromGlobals())->send();

Notarás que Symfony2 tiene en principio 2 Front Controllers: web/app.php y web/app_dev.php, esto se debe a que Symfony2 maneja por cada controlador frontal un “Entorno” (Environment) que le permite ajustar la configuración interna según la situación en que se encuentre nuestra aplicación, app.php para la Producción y app_dev.php para el desarrollo.

De ahora en adelante utilizaremos el Front Controller de desarrollo web/app_dev.php debido a que desactiva el mecanismo de caché interna de Symfony, permitiéndonos comprobar los cambios que hagamos sobre la configuración de nuestra aplicación, en nuestro navegador accederíamos así: /app_dev.php (es decir: http://localhost/Symfony/web/app_dev.php en tu navegador).
Definiendo e Importando Rutas

Como ya sabes una ruta es una asociación o mapa de un patrón URL hacia un Controlador, las mismas se definen en el archivo routing de nuestra aplicación, el cual se encuentra en app/config y puede estar definido en tres formatos: YAML, XML o PHP; Symfony 2 utiliza por defecto YAML (routing.yml) pero puede cambiarse. Por ejemplo, tenemos una ruta (suponiendo un Bundle MDWDemoBundle con DefaultController previamente definido):

# app/config/routing.yml
path_hello_world:
    pattern:  /hello
    defaults: { _controller: MDWDemoBundle:Default:index }

Analicemos cada componente:

path_hello_world: corresponde al nombre de la ruta, por ahora te parecerá irrelevante, pero es el requisito indispensable para generar las URLs internas en tu sitio que se verá más adelante, el nombre debe ser único, corto y conciso.

pattern: /hello el atributo pattern define el patrón de la ruta, lo que le permite al Routing empatar las peticiones, si el patrón fuese solo “/” representaría nuestra página de inicio, más adelante se verá su uso avanzado y los comodines.

defaults: { _controller: MDWDemoBundle:Default:index } dentro de defaults tenemos el parámetro especial _controller donde se define el “Controlador”, nótese que sigue un patrón definido Bundle:Controller:Action, esto le permite al Routing hallar el controlador especificado, automáticamente Routing resuelve la ubicación del Bundle, el controlador y la acción, sin necesidad de definir los sufijos “Controller” y “Action” correspondientes al controlador y acción.

Con la ruta anterior accederíamos al siguiente Controlador y Acción:

<?php
// src/MDW/DemoBundle/Controller/DefaultController.php
namespace MDW\DemoBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Symfony\Component\HttpFoundation\Response;

class DefaultController extends Controller
{
    public function indexAction()
    {
        $response = new Response('Hola mundo!!!');
        return $response;
    }
}

Así que ésta es la definición básica de una ruta, con ello puedo acceder desde mi navegador a /app_dev.php/hello.

Pero según la filosofía de los Bundles en Symfony2 se debe hacer el código lo más portable posible y si defino mis rutas en la aplicación, serían parte de mi aplicación y no de mi “Bundle”, por lo cual debería definirlas en el Bundle e importarlas hacia mi aplicación, de esa forma cuando tenga la necesidad de crear otra aplicación en la cual necesite cierto Bundle, sólo tengo que importar las rutas definidas en dicho Bundle para utilizarlas en mi aplicación, con lo cual mi Bundle es verdaderamente desacoplado y portable; para ello traslado mis rutas hacia el Bundle en su propio archivo de rutas: src/MDW/DemoBundle/Resources/config/routing.yml y en mi archivo de rutas de la aplicación lo importo:

# app/config/routing.yml
MDWDemoBundle:
    resource: "@MDWDemoBundle/Resources/config/routing.yml"
    prefix:   /

Como podemos ver la estructura ha cambiado, en el atributo resource: podemos definir la ruta completa hacia nuestro archivo de rutas del Bundle, en este caso utilizamos la forma especial @NombreBundle lo cual le indica al Routing que internamente resuelva la ruta hacia nuestro Bundle, haciendo la tarea muy cómoda.

También tenemos el atributo prefix: ¡que nos permite definir un prefijo!, con ello podemos hacer muchas cosas como diferenciar patrones similares en Bundles diferentes anteponiendo un prefijo, por ejemplo, si colocamos prefix: /blogger las URLs del Bundle quedarían así: app_dev.php/blogger/hello o /blogger/hello motivo por el cual el prefijo predeterminado es “/” es decir: sin prefijo.

A partir de aquí las rutas están definidas en el Bundle, y en el routing de nuestra aplicación se importan, haciendo nuestro “Bundle” más portable.
Rutas por defecto en el entorno de Desarrollo

Si revisamos bien el archivo config_dev.yml, utilizado para el entorno de desarrollo notamos que el archivo de rutas principalmente importado es routing_dev.yml, en el cual no sólo se registran las rutas del perfilador y otras que necesitas al momento de probar en el entorno de desarrollo, sino también una serie de rutas hacia el AcmeDemoBundle, que no es más que un Bundle de pruebas que no necesitas realmente en tu aplicación.

Como el AcmeDemoBundle sólo se registra en el entorno de Desarrollo, no afectará para nada tu aplicación cuando la ejecutes normalmente desde el entorno de Producción, pero debido a que el routing.yml normal es importado al final por éste, si intentas definir una ruta hacia “/” o alguna que coincida con dicho AcmeDemoBundle, notarás que al acceder desde el entorno de Desarrollo tomará prioridad AcmeDemoBundle y no el tuyo, afortunadamente la solución es muy sencilla donde simplemente comentas o eliminas las 3 rutas que pertenecen al AcmeDemoBundle, luego, si lo deseas eliminas dicho Bundle:

# app/config/routing_dev.yml

# COMENTAMOS o ELIMINAMOS estas 3 rutas: ------------------------

#_welcome:
#    pattern:  /
#    defaults: { _controller: AcmeDemoBundle:Welcome:index }

#_demo_secured:
#    resource: "@AcmeDemoBundle/Controller/SecuredController.php"
#    type:     annotation

#_demo:
#    resource: "@AcmeDemoBundle/Controller/DemoController.php"
#    type:     annotation
#    prefix:   /demo

_assetic:
    resource: .
    type:     assetic

_wdt:
    resource: "@WebProfilerBundle/Resources/config/routing/wdt.xml"
    prefix:   /_wdt

_profiler:
    resource: "@WebProfilerBundle/Resources/config/routing/profiler.xml"
    prefix:   /_profiler

_configurator:
    resource: "@SensioDistributionBundle/Resources/config/routing/webconfigurator.xml"
    prefix:   /_configurator

_main:
    resource: routing.yml

Resumen Final

En éste capítulo profundizamos en el comportamiento esencial del Sistema de Routing, en dónde un Controlador Frontal despacha las peticiones a los controladores reales de nuestra aplicación, además comprendimos la importancia en la importación de las rutas, para así obtener el código correspondiente a nuestros Bundles encapsulados en los mismos, fomentando un diseño perfectamente desacoplable, adicionalmente vimos como desactivar el AcmeDemoBundle desde nuestro entorno de Desarrollo para así permitirnos navegar nuestra aplicación por completo desde éste y así obtener las ventajas del perfilador para depurar nuestros proyectos.

En el próximo capitulo aprenderemos a como definir rutas a la medida, aprovechar las ventajas del sistema bidireccional y con ello rendir al máximo con el desarrollo de rutas amigables, limpias y precisas.
