# Manejando las vistas con Twig en Symfony2 #

Dentro del patrón o arquitectura MVC la vista es la encargada de proporcionar la verdadera “interfaz” a nuestros usuarios, para ello en Symfony2 podemos usar el servicio de Templating (plantillas) y como ya sabrás Twig es el motor por defecto, aunque si lo prefieres puedes usar PHP.
¿Por qué Twig?

Porque las plantillas en Twig son muy fáciles de hacer y resultan muy intuitivas en el caso de que contemos con maquetadores o Diseñadores Frontend, además de todo ello su sintaxis corta y concisa es muy similar (por no decir idéntica) a la de otros famosos FW como django, Jinja, Ruby OnRails y Smarty; además Twig implementa un novedoso mecanismo de herencia de plantillas y no tendrás que preocuparte por el peso que conlleva el interpretar todo ello, debido a que Twig cachea en auténticas clases PHP todo el contenido de las mismas, para acelerar el rendimiento de nuestra aplicación.

En este capítulo nos concentraremos en las nociones básicas de Twig, verás lo sencillo que es interactuar con él.
Lo básico de Twig

{# comentario #}
{{ mostrar_algo }}
{% hacer algo %}

Con Twig mostrar el contenido de una variable es tan simple como usar las dobles llaves {{ variable }}, sin necesidad de echo ni etiquetas de apertura de PHP (<?php ?>), además de eso Twig es un lenguaje de plantillas, lo que nos permite hacer condicionales y estructuras de control muy intuitivas y funcionales:

<ul>
  {% for usuario in usuarios %}
    <li>{{ usuario.nombreusuario | upper }}</li>
  {% else %}
    <li><em>no hay usuarios</em></li>
  {% endfor %}
</ul>

¿for .. else?: sí, esto en realidad no existe en PHP, como puedes notar Twig internamente hace una comprobación de la colección (u array) antes de iterarlo, lo que hace a la plantilla más fácil de escribir y es intuitivo para maquetadores.
{{ usuario.nombreusuario | upper }} Twig dispone de una serie de filtros, los cuales puedes anidar hacia la derecha con el operador “pipe” (|) de esta forma con el filtro upper nos imprime el valor de la variable usuario.nombreusuario en mayúsculas, al final del capítulo mostraremos los filtros más comunes.
Nomenclatura y Ubicación de Plantillas

Symfony 2 dispone de dos principales lugares para contener a las plantillas:

    En nuestros Bundles: src/Vendor/MyBundle/Resources/views
    En la Aplicación: app/Resources/views y para reemplazo: app/Resources/VendorMyBundle/views

De esta forma el servicio de templating de Symfony busca primero las plantillas en la Aplicación y luego en el mismo Bundle, permitiéndonos un mecanismo simple para reemplazar plantillas en Bundles de terceros.

Como se indicó en el capítulo anterior, cuando renderizamos las vistas (es decir plantillas) desde el Controlador utilizamos un patrón definido:
[Bundle]:[Controller]:template[._format].template_engine

El parámetro template representa el nombre de archivo de nuestra plantilla (como norma se recomienda el mismo nombre de la Acción del Controlador), el parámetro _format es requerido y se usa para poder diferenciar el formato real que representará la plantilla y el último parámetro template_engine que representa la extensión real de nuestra plantilla le indica al servicio de templating el motor a utilizar, así que un archivo de plantillas quedaría de la siguiente forma:

show.html.twig o show.rss.twig

Con respecto a la primera parte los parámetros [Bundle]:[Controller] nos permiten ubicar la plantilla dentro de un Bundle, como puedes notar Controller indica que existe otra carpeta (views/Controller), en donde buscara el template, si se omite buscará nuestro template en views, ejemplo:

MDWDemoBundle:Blog:show.html.twig =

    src/MDW/DemoBundle/Resources/views/Blog/show.html.twig
    app/Resources/MDWDemoBundle/views/Blog/show.html.twig

MDWDemoBundle::show.html.twig =

    src/MDW/DemoBundle/Resources/views/show.html.twig
    app/Resources/MDWDemoBundle/views/show.html.twig

Nótese que de omitirse el Controller se busca en la raíz de views, pero también podemos omitir el Bundle, de ésta forma indicamos que queremos una plantilla global de nuestra Aplicación, con el cual no dependería del Bundle:

:Blog:show.html.twig = app/Resources/views/Blog/show.html.twig
::show.html.twig = app/Resources/views/show.html.twig

De esta forma indicamos con precisión de donde queremos nuestras plantillas y no sólo desde los Controladores, éste patrón también se aplica dentro de las mismas plantillas tanto para incluir otras como para el mecanismo de herencia de plantillas que veremos más adelante.
Herencia de Plantillas

Este mecanismo está pensado para simular el mismo comportamiento de la POO en nuestras plantillas, de modo que tengamos “plantillas base” que contienen los elementos comunes como el Layout, de esta forma las plantillas pueden heredar de las Plantillas Base, proporcionando un Decorado de plantillas multi-nivel; Twig provee esto por medio de Bloques (block) y utilizando el método extends para heredar la plantilla, vemos un ejemplo básico de una plantilla que herede de otra:

Plantilla base por defecto de Symfony2:

{# app/Resources/views/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title>{% block title %}Welcome!{% endblock %}</title>
        {% block stylesheets %}{% endblock %}
        <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
    </head>
    <body>
        {% block body %}{% endblock %}
        {% block javascripts %}{% endblock %}
    </body>
</html>

Plantilla layout de nuestro Bundle:

{# src/MDW/DemoBundle/Resources/views/layout.html.twig #}
{% extends '::base.html.twig' %}
{% block stylesheets %}
    {{ parent() }}
    <link href="{{ asset('bundles/mdwdemo/css/main.css') }}" type="text/css" rel="stylesheet" />
{% endblock %}
{% block body %}
    <div id="layout">

        <div id="header">
            <h1>Encabezado</h1>
        </div>
        <div id="content" class="clean" >
            <div id="content_left">
                {% block content %}
                {% endblock %}
            </div>
            <div id="content_right">
                columna
            </div>
        </div>
        <div id="footer">
            <p>Pie {{"now"|date("m/d/Y")}}</p>
        </div>
    </div>
{% endblock %}

Como puedes notar en la Plantilla base (base.html.twig) se define la estructura básica de todo documento HTML (en este caso HTML5) y dentro de la misma se definen los bloques title, stylesheets, body y javascript, de ese modo la plantilla que herede de ella puede reemplazar cada uno de estos bloques facilitando el decorado, en la plantilla layout.html.twig de nuestro Bundle se aprecia:

    {% extends ‘::base.html.twig’ %} extiende o hereda (herencia simple) de la plantilla base, fíjate que la nomenclatura es igual a la expresada en el enunciado anterior.
    {% block stylesheets %} redefinir el bloque stylesheets indica que se reemplaza para incluir una hoja de estilos propia de nuestro bundle, y dentro del bloque notarás {{ parent() }} esto permite que el contenido original del mismo bloque en la plantilla base sea agregado, con el cual podremos conservar las hojas CSS declaradas en la plantilla base.

Además podemos definir nuevos bloques como {% block content %}, de esta forma en las plantillas que extiendan de ésta puedan reemplazar solo el bloque estratégico, esto con el fin de introducir el modelo de Herencia a tres niveles propuesto por Symfony2 que brinda una manera flexible de independizar nuestro layout del Bundle del de la Aplicación, con ello nuestras plantillas finales tendrían este aspecto:

{# src/MDW/DemoBundle/Resources/views/Blog/show.html.twig #}
{% extends 'MDWDemoBundle::layout.html.twig' %}
{% block content %}
    El artíclo solicitado es: {{articulo}}
{% endblock %}

De está manera al renderizar la plantilla show.html.twig ésta a su vez extiende del layout.html.twig del Bundle que extiende del base.html.twig de la Aplicación, y en la plantilla solo necesitamos reemplazar el Bloque correspondiente al contenido.
Reutilizando Plantillas

Una de las necesidades más comunes que se nos pueden presentar es la de fragmentar el código común y repetitivo en nuestras plantillas (conocido como partials), con el objetivo de reutilizarlo en más de una plantilla, para ello en Symfony2 podemos crear tales fragmentos en archivos de plantillas separados, y utilizando el helper include de twig podemos incluir dinámicamente el contenido de dicha plantilla, además de permitirnos pasar datos a la misma:

{% include 'MDWDemoBundle:Blog:articuloDetalles.html.twig' with {'articulo': articulo} %}

Como puedes notar el patrón de acceso a la plantilla es el mismo que utilizas al renderizar cualquier plantilla, además puedes añadir un array de variables pasadas a la misma después de la clausula with.
Reutilizando Controladores

En algunos casos en nuestras plantillas incluidas necesitamos acceder a datos del modelo que de otra forma sólo seria posible si desde el controlador añadimos la consulta hacia el modelo (por ejemplo un barner con los 3 títulos de artículos más recientes), por lo cual tendríamos un doble trabajo al volver a pasar las variables generadas al include de dicha plantilla, una mejor solución es crear otro controlador que realice dicha tarea por separado y renderizarlo o incrustarlo directamente desde la plantilla (conocido en Symfony1 como los components), para ello utilizamos el helper render:

{% render "MDWDemoBundle:Articulo:articulosRecientes" with {'max': 3} %}

De esta forma puedes crear un controlador articulosRecientes que realice la correspondiente consulta al modelo y pasar variables específicas como la cantidad que quieras consultar, así no es necesario incluir consultas adicionales en tu controlador principal, conservando la semántica de cada controlador y un código más limpio y ordenado, además de que dicho controlador No requiere que le definas una ruta.
Incorporando Activos (Assets)

Toda aplicación web contiene un conjunto de Activos (Assets) que son básicamente archivos Javascript, Hojas de estilo CSS, Imágenes y demás; generalmente estos archivos deben de publicarse dentro del árbol de publicación del sitio para que sean accesibles por el navegador, pero el hecho de disponer de “Url Amigables en la aplicación” nos obliga a generar rutas hacia éstos desde la raíz del sitio anteponiendo un slash (/), por ejemplo “/images/logo.gif”, eso si la aplicación está disponible desde la raíz del servidor, en el caso de tenerla en una carpeta compartida hay que añadirla: “/miaplicacion/web/images/logo.gif” lo que resulta en un problema, afortunadamente el Helper asset nos permite hacer la aplicación más portable, con ello solo necesitamos:

<img src="{{ asset('images/logo.gif') }}" alt="mi logo!" />

De esta forma no importa en donde viva tu aplicación, el Helper asset de Symfony 2 se encarga de generar la URL correcta para que no tengas que preocuparte por ello.
Listado de filtros Twig y Helpers más comunes
RAW y Escapado de variables (XSS)

De forma predeterminada Twig escapa todos los caracteres especiales HTML de las variables, lo que permite una protección frente a XSS, de igual forma si tenemos un contenido que disponga de código HTML seguro, podremos evitar este mecanismo con el filtro raw:

{{ variable | raw }} {# evita el escapado de variables #}

{# fuerza el escapado de variables (opción por defecto en Symfony2) #}
{{ variable | escape }}

default (valores por defecto) y detectando variables no declaradas

Si una variable no es pasada a la plantilla twig devolverá una excepción, esto es un inconveniente para cuando necesitemos reutilizar plantillas en las cuales no todas las variables necesiten ser pasadas, afortunadamente en estos casos podemos definir un valor por defecto:

{{ variable | default('valor por defecto') }}

Pero en oraciones no necesitamos que twig nos devuelva un valor por defecto, sino saber si la variable fue declarada o no, para interactuar en un condicional por ejemplo, allí usamos is defined:

{% if variable is defined %}
    {# aplicar operaciones si no se ha declarado la variable #}
{% endif %}

En el caso de querer comprobar si la variable está declarada pero tiene un valor null usamos variable is null
Capitalización, Mayúsculas y Minúsculas

Con estos sencillos filtros podremos capitalizar o convertir a mayúsculas / minúsculas una variable cadena:

{{ variable | capitalize }} {# capitaliza el primer carácter de la cadena #}
{{ variable | lower }} {# convierte a minúsculas la cadena #}
{{ variable | upper }} {# convierte a mayúsculas la cadena #}
{{ variable | title }} {# capitaliza cada palabra de la cadena #}

y por si fuera poco, podremos aplicar el filtro a un gran bloque de código HTML anidándolo dentro de un bloque filter de twig:

{% filter upper %}
    Todo el texto de aquí será convertido a mayúsculas
{% endfilter %}

date (Formateando fechas)

El filtro date es una forma rápida de aplicar formato a nuestras variables con fechas y lo mejor de todo es que internamente aplica las mismas convenciones de la función date() nativa de PHP, además como parámetro adicional podemos establecer la zona horaria:

{{ variable | date("m/d/Y", "Europe/Paris") }}

En ciertas ocasiones necesitamos simplemente la fecha/hora actual, por lo que no es necesario declarar en el controlador una variable y asignarle el timestamp actual, colocando como fecha “now” en twig es realmente simple:

{# "now" nos devuelve la fecha/hora actual #}
{{ "now" | date("m/d/Y") }}

number_format (formato numérico)

Twig realmente nos ofrece con este filtro un atajo a la función nativa number_format de PHP:

{{ 2500.333 | number_format(2, ',', '.') }}

~(Concatenación en twig) y declarando variables desde plantilla

En ciertas ocasiones necesitamos concatenar una cadena estática con una o más variables, para aplicarlo como parámetro de un filtro, o para asignarlo a una variable declarada desde plantilla para reutilizarla, un ejemplo práctico puede ser crear una variable con el “share url” para compartir una entrada de blog en varias redes sociales, así evitamos el repetir constantemente el llamado al helper path:

{% set url_share = 'http://maycolalvarez.com' ~ path('blog_article', {
    'year'  : (article.created|date('Y')),
    'month' : (article.created|date('m')),
    'slug'  : article.slug })
%}
<!-- Coloca esta etiqueta donde quieras que se muestre el botón +1. -->
<g:plusone size="medium" href="{{ url_share }}"></g:plusone>
<a href="https://twitter.com/share" class="twitter-share-button" data-url="{{ url_share }}" data-lang="es">Twittear</a>
<div class="fb-like" data-href="{{ url_share }}" data-send="false" data-layout="button_count" data-width="100" data-show-faces="true"></div>

Como puedes notar con set variable = podemos declarar una variable, asignamos una cadena estática entre comillas simples y concatenamos el resultado del helper path con el operador (~), con ello podemos usar {{ url_share }} en la url de cada botón de red social, en el ejemplo Google+, Twitter y Facebook (consulte su api para más detalles).
Resumen Final

Como pudimos apreciar, manejar las Vistas con Twig nos permite tener código mucho más limpio e intuitivo,  no sólo abstrae la verdadera lógica de una vista, sino que nos proporciona de herramientas útiles, no sólo diseñadores sino a programadores también como la herencia de plantillas lo cual separa de forma limpia el layout, además de poder incluir plantillas y modularizar el contenido de las vistas y llamar controladores para obtener datos del modelo sin romper el esquema MVC.

Con esto culminamos el capítulo de la vista, reitero que puedes usar PHP como motor de plantillas si lo deseas, pero si elijes quedarte con twig no olvides revisar su documentación (http://twig.sensiolabs.org/documentation) para extender tus conocimientos y mejorar tu desempeño.
