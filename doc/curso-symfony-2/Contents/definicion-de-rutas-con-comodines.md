# Definición de rutas con comodines #

No sólo de rutas estáticas se compone una aplicación web, usualmente se necesitan pasar parámetros variables por GET, es decir, por la URL y es aquí en donde el Routing saca lo mejor que tiene. Un marcador de posición o comodín es un segmento de la ruta variable, como por ejemplo: /blog/articulo_x dónde articulo_x es una parte variable que representa la página a consultar, en Symfony2 estos comodines se definen entre llaves “{}”:

# src/MDW/DemoBundle/Resources/config/routing.yml
blog_mostrar:
    pattern:   /blog/{slug}
    defaults:  { _controller: MDWDemoBundle:Blog:show }  

# src/MDW/DemoBundle/Resources/config/routing.yml
blog_mostrar:
    pattern:   /blog/{slug}
    defaults:  { _controller: MDWDemoBundle:Blog:show }

La parte {slug} representa nuestro comodín y como es variable cualquier URL que coincida con la expresión: /blog/* empatará con dicha ruta, además el mismo comodín (slug) va a coincidir con el parámetro slug que definamos en el Controlador, por ejemplo, en una ruta /blog/el_articulo_de_symfony en el controlador la variable (parámetro) $slug contendrá “el_articulo_del_symfony”.

De forma predeterminada los comodines son requeridos, si tuvieses, por ejemplo, una ruta de paginación como esta: /blog/page/1 tendrías que definir un comodín para el número de página, pero estarías obligado siempre en agregar “1”, esto se resuelve añadiendo un valor por defecto:

# src/MDW/DemoBundle/Resources/config/routing.yml
blog_index:
    pattern: /blog/page/{page}
    defaults: { _controller: MDWDemoBundle:Blog:index, page: 1 }

De esta forma añadimos el valor por defecto del comodín page: 1 como parámetro del atributo defaults.
Agregando Requisitos a la ruta

Hasta ahora tenemos dos rutas: /blog/{slug} y /blog/page/{page}, la segunda es poco práctica ¿que pasaría si la simplificamos como /blog/{slug} y /blog/{page}?, que ambos patrones asemejan con /blog/* y el Routing solo tomará en cuenta la primera ruta coincidente. La solución a este problema es añadir requerimientos, como por ejemplo definir que el comodín {page} acepte solo números que es lo que realmente lo diferencia del {slug}:

# src/MDW/DemoBundle/Resources/config/routing.yml
blog_index:
    pattern: /blog/{page}
    defaults: { _controller: MDWDemoBundle:Blog:index, page:  1 }
    requirements:
        page:  d+
blog_mostrar:
    pattern:   /blog/{slug}
    defaults:  { _controller: MDWDemoBundle:Blog:show }

Nótese que con el nuevo parámetro requirements: puedes definir expresiones regulares para cada comodín, en el caso anterior al añadirle d+ a {page} le indicamos al Routing que dicho parámetro debe coincidir con la expresión regular, lo que te permite definir el requerimiento de acuerdo a tus necesidades.

Es importante aclarar que debes declarar tus rutas en un orden lógico, debido a que el Routing toma en cuenta la primera ruta que coincide, por ejemplo, si defines la ruta blog_mostrar antes de blog_index no funcionará porque como blog_mostrar no tiene requerimientos, cualquier número coincide perfectamente con {slug} y llegar a {page} nunca será posible de esa forma.

Además de especificar requisitos para cada comodín, existen otros de mucha ayuda:

    _method: [GET | POST ] como lo indica su nombre, permite establecer como restricción que la ruta solo coincida si la petición fue POST o GET, muy útil cuando nuestra acción del controlador consista en manipulación de datos o envío de forms.

    _format: es un parámetro especial que permite definir el content-type de la Respuesta (Response), puede ser utilizado en el patrón como el comodín {_format} y de esta forma pasarlo al Controlador.

Con todo esto ya somos capaces de crear rutas dinámicas con Symfony2, pero el sistema de Routing es tan flexible que permite crear rutas avanzadas, veamos:

# src/MDW/DemoBundle/Resources/config/routing.yml
blog_articulos:
    pattern:  /articulos/{culture}/{year}/{titulo}.{_format}
    defaults: { _controller: MDWDemoBundle:Blog:articulos, _format: html }
    requirements:
        culture:  en|es
        _format:  html|rss
        year:     d{4}+

En este ejemplo podemos apreciar que {culture} debe coincidir con en o es, además {year} es un número de cuatro dígitos y el {_format} por defecto es html, así que las siguientes rutas empatan con nuestra anterior definición:

    /articulos/es/2000/patron_mvc
    /articulos/es/2000/patron_mvc.html
    /articulos/es/2000/patron_mvc.rss

Generando Rutas

Como anteriormente indicamos, el Routing es un sistema bidireccional, en el cual nos permite generar URLs desde las mismas definiciones de rutas, el objetivo es obvio: si deseas cambiar el patrón de la ruta lo haces y ya. No necesitas buscar los link’s internos hacia esa ruta dentro de tu aplicación si utilizas el generador de rutas de Symfony2.

La forma más común de utilizar el generador de rutas es desde nuestras plantillas (Vistas/Views) y para ello solo necesitamos acceder al objeto “router”, ejemplos:

Con TWIG como gestor de plantillas usamos path para obtener la url relativa

<a href="{{ path('blog_mostrar', { 'slug': 'mi-articulo' }) }}">
    lee el artículo.
</a>

O url si queremos una url absoluta:

<a href="{{ url('blog_mostrar', { 'slug': 'mi-articulo' }) }}">
    lee el artículo.
</a>

Con PHP como gestor de plantillas vemos que en realidad accedemos al helper router para obtener la url relativa:

<a href="<?php echo $view['router']->generate('blog_mostrar', array('slug' => 'mi-articulo')) ?>">
    lee el artículo
</a>

En el caso de querer una url absoluta solo debemos especificar true en el tercer parámetro de la función generate:

<a href="<?php echo $view['router']->generate('blog_mostrar', array('slug' =>'mi-articulo'), true) ?>;">
    lee el artículo
</a>

Así de sencillo. Si en nuestras vistas usamos el generador de rutas no tendremos que preocuparnos por la eventualidad de cambiar las URLs de nuestra aplicación, además de que también podemos acceder desde nuestros controladores gracias a la inyección de dependencias(DI) donde obtendremos el servicio “router”:

// src/MDW/DemoBundle/Controller/DefaultController.php
class BlogController extends Controller
{
    public function showAction($slug)
    {
      // ...
      $url = $this-&amp;amp;gt;get('router')-&amp;amp;gt;generate('blog_mostrar', array('slug' =&amp;amp;gt; 'mi-articulo'));
      // Atajo si extiendes la clase Controller:
      $url = $this-&amp;amp;gt;generateUrl('blog_mostrar', array('slug' =&amp;amp;gt; 'mi-articulo'));
    }
}

Esto es todo lo básico que debes saber para generar rutas dinámicas y concisas en Symfony2.
Resumen Final

Como podemos observar el sistema de rutas de Symfony2 no sólo nos permite definir caminos claros y concisos para nuestros usuarios, sino que también nos hace más fácil su implementación, con lo cual al cambiar el patrón de una ruta no tenemos que buscar todas las llamadas de la misma en la aplicación.

Además aprendimos a definir rutas condicionadas, entendimos la importancia del orden de declaración de las mismas y vimos como utilizar el sistema de routing bidireccional en nuestras vistas y controladores.

En el próximo capítulo profundizaremos sobre los controladores en Symfony2, aprenderemos a como definirlos y adaptarlos a nuestras necesidades.
