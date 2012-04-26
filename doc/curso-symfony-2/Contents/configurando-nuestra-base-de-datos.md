# Configurando nuestra Base de Datos # {#configurando-nuestra-base-de-datos}

En este capítulo entraremos al mundo de Doctrine usándolo dentro de nuestro proyecto para así tener ya los datos dinámicamente almacenados en nuestra base de datos y poder manipularlos.

Como ya lo habíamos hablado en el capítulo 1, Doctrine es un ORM (Object-Relational Mapping). Cuando hablamos de relaciones en conceptos de base de datos, estamos refiriéndonos a las tablas diciendo entonces que existe una vinculación entra las tablas y objetos. Al usar un ORM mapeamos cada tabla con objetos dentro de nuestras aplicaciones, por ejemplo si tenemos una tabla de personas en la base de datos, tendremos un objeto Persona en la aplicación que conoce cuales son sus campos, tipos de datos, índices, etc. logrando con esto que la aplicación conozca el modelo de los datos desde un punto de vista orientado a objetos, es decir representado con Clases y Objetos.

Doctrine nos permitirá, conociendo nuestras tablas como hablamos anteriormente, crear las sentencias SQL por nosotros ya que toda la información necesaria para crear estos queries se encuentra “mapeada” en código PHP.

Como si esto fuera poco, la mayor de las ventajas de contar con un ORM será que nos permite como desarrolladores, abstraernos de que motor de base de datos estemos usando para el proyecto y nos permitirá, con solo un poco de configuración, cambiar toda nuestra aplicación por ejemplo de una base de datos MySQL a PostgreSQL o a cualquier otra soportada por el framework Doctrine.
Configuración de la base de datos

Para configurar los datos de la conexión del servidor de base de datos se deberán ingresar los valores en el archivo app\config\parameters.ini dentro de las variables ya definidas:

    database_driver = pdo_mysql
    database_host = localhost
    database_port =
    database_name = blog
    database_user = maestros
    database_password = clavesecreta

Estos datos serán usados por Doctrine para conectarse al servidor y trabajar con la base de datos. No hay necesidad de ingresar el puerto si se está usando el que figura por defecto para la base de datos que usemos, para nuestro caso MySQL.

Una vez que ya tenemos configurada nuestra base de datos, podemos usar el comando “console” para decirle a nuestro proyecto que se encargue de crear la base de datos en el servidor ejecutándolo de la siguiente manera:

C:\wamp\www\Symfony>php app\console doctrine:database:create
Created database for connection named <comment>blog</comment>

Esto nos retornará un mensaje diciéndonos, si los permisos estaban correctos, que la base de datos blog fue creada. Podemos revisar ahora nuestra base de datos por medio del phpMyAdmin y veremos que la base de datos ya existe.
Nota
En caso de necesitar borrar la base de datos podemos usar el comando “doctrine:database:create –force”, donde tendremos que pasarle el parámetro especial –force para confirmar que ejecute la acción.
Capturar a Evernote

El concepto que suele ser muy utilizado en frameworks de este tipo es ir creando la base de datos desde la aplicación, es decir, que la aplicación se encargue de crear la base de datos, las tablas, relaciones, índices y los datos de ejemplo. Este concepto lo iremos ahondando durante estos capítulos.
Creando las tablas y conociendo las Entidades

Como ejemplo usaremos dos tablas para nuestro manual simplemente para ver como trabajar con ellas usando el framework. Crearemos una tabla de Artículos y una de Comentarios para tener como ejemplo el concepto ya bien conocido de un blog.

A continuación tenemos el SQL de las tablas con sus campos:

CREATE TABLE IF NOT EXISTS `articles` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) NOT NULL,
  `author` varchar(255) NOT NULL,
  `content` longtext NOT NULL,
  `tags` varchar(255) NOT NULL,
  `created` date NOT NULL,
  `updated` date NOT NULL,
  `slug` varchar(255) NOT NULL,
  `category` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

CREATE TABLE IF NOT EXISTS `comments` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `author` varchar(255) NOT NULL,
  `content` longtext NOT NULL,
  `reply_to` int(11) NOT NULL,
  `article_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `IDX_A6E8F47C7294869C` (`article_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

Para crear estas tablas en la base de datos primeramente lo haremos en código PHP para que Doctrine conozca perfectamente las tablas para poder interactuar con ellas. Es aquí donde conocemos el concepto de las Entidades (Entities). Una entidad es la representación “orientada a objetos” de nuestras tablas, es decir que crearemos una clase por cada una de nuestras tablas. Para no tener que escribirlas a mano, Symfony nos provee un generador de Entidades que es invocado con nuestro comando console de la siguiente manera:

C:\wamp\www\Symfony>php app\console doctrine:generate:entity

Al darle enter nos preguntará el nombre que usaremos para nuestra entidad y hay que entender que sigue el siguiente formato: IdentificadorDelBundle:NombreEntidad. Para nuestro caso, y como ya vimos en los capítulos anteriores, nuestro identificador del Bundle es MDWDemoBundle y primero crearemos la entidad para nuestra tabla de artículos por lo tanto escribiremos lo siguiente:

The Entity shortcut name: MDWDemoBundle:Articles

Al darle enter nos preguntará que formato queremos usar para agregar los metadatos que mapearan a la tabla proponiéndonos [annotation]. Aceptaremos la propuesta presionando Enter

Configuration format (yml, xml, php, or annotation) [annotation]:

Luego ya nos empezará a preguntar cuales serán las columnas de nuestra tabla y por cada columna el tipo de dato que arriba nos deja como ejemplo cuales podemos elegir. En caso de que sea un tipo de datos que necesite longitud también nos la pedirá y cuando ya no queramos ingresar columnas simplemente daremos un enter dejando vacío el valor.
Nota
No será necesario ingresar el campo id de la tabla ya que Symfony lo creará automáticamente con la idea de ser usado como clave primaria autonumérica.
Capturar a Evernote

Una vez finalizada la carga de los campos, nos preguntará si queremos crear un repositorio vacío a lo que contestaremos SI para luego, como último paso preguntarnos si confirmamos la creación automática de la Entidad a lo que también diremos que SI:

Instead of starting with a blank entity, you can add some fields now.
Note that the primary key will be added automatically (named id).

Available types: array, object, boolean, integer, smallint,
bigint, string, text, datetime, datetimetz, date, time, decimal, float.

New field name (press <return> to stop adding fields): title
Field type [string]:
Field length [255]:

New field name (press <return> to stop adding fields): author
Field type [string]:
Field length [255]:

New field name (press <return> to stop adding fields): content
Field type [string]: text

New field name (press <return> to stop adding fields): tags
Field type [string]:
Field length [255]:

New field name (press <return> to stop adding fields): created
Field type [string]: date

New field name (press <return> to stop adding fields): updated
Field type [string]: date

New field name (press <return> to stop adding fields): slug
Field type [string]:
Field length [255]:

New field name (press <return> to stop adding fields): category
Field type [string]:
Field length [255]:

New field name (press <return> to stop adding fields):

Do you want to generate an empty repository class [no]? yes

Summary before generation

You are going to generate a "MDWDemoBundle:Articles" Doctrine2 entity
using the "annotation" format.

Do you confirm generation [yes]?

Entity generation

Generating the entity code: OK

You can now start using the generated code!

Una vez finalizado el generador, tendremos dos archivos creados dentro de la carpeta src\MDW\DemoBundle\Entity ya que como prefijo de nuestro nombre de Entidad usamos el identificador de nuestro Bundle (MDWDemoBundle). El primer archivo será nuestra entidad Articles.php y el otro será el repositorio de esa entidad ArticlesRepository.php del cual hablaremos en el siguiente capítulo.

Si revisamos el contenido de estos archivos podremos ver que es una Clase con el nombre Articles y que contiene como propiedades los datos que hemos cargado de las columnas de la tabla incluyendo sus métodos getters y setters. También podremos ver que la información que mapea los datos de la tabla se encuentra como parte de bloques de comentarios sobre cada propiedad usando el concepto de Anotaciones, concepto muy conocido en el lenguaje Java. Esta información será utilizada en runtime para conocer datos sobre la entidad. Comentemos algunos más importantes:

    @ORM\Entity: Indica que esta tabla se comportará como una entidad, es decir que mapeará una tabla de la base de datos.
    @ORM\Table: Doctrine usará el nombre de nuestra entidad para crear una tabla en la base de datos con el mismo nombre y esta anotación nos permitirá especificar un nombre diferente si agregamos el parametro name: @ORM\Table(name=”nombre_tabla”).
    @ORM\Column: Indica que esta propiedad mapea una columna de la tabla y como argumentos recibe datos de la misma como por ejemplo el tipo de dato y el largo en el caso de ser string. Doctrine se encargará de crear estas columnas con el tipo de dato correspondiente para cada motor de base de datos
    @ORM\Id: Indica que esta propiedad/columna será la Clave Primaria de la tabla
    @ORM\GeneratedValue: Indica que este campo numérico se irá autoincrementando usando la estrategia que pasemos por parámetro. En este caso AUTO hará que elija la mejor forma según el motor de base de datos que usemos, por ejemplo si usamos MySQL creará un indice autonumérico mientras que si es PostgreSQL usará un dato de tipo SERIAL.

Ahora creemos la entidad para la tabla de comentarios

C:\wamp\www\Symfony>php app\console doctrine:generate:entity

  Welcome to the Doctrine2 entity generator

This command helps you generate Doctrine2 entities.

First, you need to give the entity name you want to generate.
You must use the shortcut notation like AcmeBlogBundle:Post.

The Entity shortcut name: MDWDemoBundle:Comments

Determine the format to use for the mapping information.

Configuration format (yml, xml, php, or annotation) [annotation]:

Instead of starting with a blank entity, you can add some fields now.
Note that the primary key will be added automatically (named id).

Available types: array, object, boolean, integer, smallint,
bigint, string, text, datetime, datetimetz, date, time, decimal, float.

New field name (press <return> to stop adding fields): author
Field type [string]:
Field length [255]:

New field name (press <return> to stop adding fields): content
Field type [string]: text

New field name (press <return> to stop adding fields): reply_to
Field type [string]: integer

New field name (press <return> to stop adding fields):

Una vez que tengamos estas 2 entidades creadas podemos probar crear las tablas en la base de datos con el siguiente comando:

C:\wamp\www\Symfony>php app\console doctrine:schema:create

Ahora revisando la base de datos con el phpMyAdmin podremos ver como ambas tablas fueron creadas por nuestro proyecto y nos daremos cuenta como Doctrine, por medio de nuestras Entidades, conoce perfectamente como crearlas.
Modificando la estructura de las tablas

Ahora que ya tenemos nuestras Entidades tenemos que crear la relación que existe entre ellas. Para este caso decimos que un Articulo puede llegar a tener varios comentarios relacionados, mientras que un comentario pertenece a un artículo específico, por lo que en la tabla de comentarios deberíamos agregar una Clave Foránea apuntando a la tabla de artículos. Para esto, agregaremos la siguiente propiedad con sus respectivos getter y setter a nuestra entidad Comment:

    /**
     * @ORM\ManyToOne(targetEntity="Articles", inversedBy="comments")
     * @ORM\JoinColumn(name="article_id", referencedColumnName="id")
     * @return integer
     */
    private $article;
    public function setArticle(\Mdw\BlogBundle\Entity\Articles $article)
    {
        $this->article = $article;
    }

    public function getArticle()
    {
        return $this->article;
    }

Con este código estamos definiendo una relación entre ambas tablas y nuevamente las anotaciones permiten a Doctrine especificar como se define la FK:

    @ORM\ManyToOne: Esta anotación le dice que desde esta tabla de comentarios existe una relación de muchos a uno con la entidad Articles
    @ORM\JoinColumn: Especifica las columnas que se usarán para hacer el join. Localmente se usará una campo article_id y en la tabla de referencia se usará la propiedad id

Con esto hemos modelado la FK desde el punto de vista de la entidad Comment, iremos a la entidad Articles a escribir la relación desde su punto de vista agregando el siguiente código

    /**
     * @ORM\OneToMany(targetEntity="Comments", mappedBy="article")
     */
    private $comments;
    public function __construct()
    {
        $this->comments = new \Doctrine\Common\Collections\ArrayCollection();
    }
    public function addComments(\Mdw\BlogBundle\Entity\Comments $comments)
    {
        $this->comments[] = $comments;
    }

    public function getComments()
    {
        return $this->comments;
    }

Agregando la propiedad $comments estamos creando la referencia a la otra tabla ya que un artículo puede tener varios comentarios usamos la anotación inversa a la que vimos anteriormente @ORM\OneToMany y podremos ver que agregamos un constructor que inicializa la propiedad con un objeto del tipo ArrayCollection, que nos permitirá que un artículo contenga varios comentarios para así poder obtenerlos todos a través del método getComments().

Con estas modificaciones realizadas volvamos a generar nuestras tablas, pero como ya hemos creado ambas tablas, ejecutemos primeramente para borrarlas el siguiente comando:

C:\wamp\www\Symfony>php app\console doctrine:schema:drop --force

Para luego volver a crearlas usando el comando ya conocido:

C:\wamp\www\Symfony>php app\console doctrine:schema:create

La otra manera de actualizar nuestras tablas, en caso de no poder o no querer borrarlas es usando el comando de update donde podemos ver la gran potencia que nos provee Doctrine enviando solo el SQL necesario para actualizar las tablas

C:\wamp\www\Symfony>php app\console doctrine:schema:update --force

Nota
En cualquiera de los tres casos doctrine:schema:create, doctrine:schema:drop y doctrine:eschema:update podemos usar el parámetro especial “–dump-squl” para que en lugar de ejecutar el SQL necesario solo nos lo muestre en la pantalla para poder controlarlo:
Capturar a Evernote

C:\wamp\www\Symfony>php app\console doctrine:schema:create --dump-sql
ATTENTION: This operation should not be executed in a production environment.

CREATE TABLE Articles (id INT AUTO_INCREMENT NOT NULL, 
title VARCHAR(255) NOT NULL, author VARCHAR(255) NOT NULL,
content LONGTEXT NOT NULL, tags VARCHAR(255) NOT NULL,
created DATE NOT NULL, updated DATE NOT NULL,
slug VARCHAR(255) NOT NULL, category VARCHAR(255) NOT NULL,
PRIMARY KEY(id)) ENGINE = InnoDB;

CREATE TABLE Comments (id INT AUTO_INCREMENT NOT NULL,
author VARCHAR(255) NOT NULL, content LONGTEXT NOT NULL,
reply_to INT NOT NULL, PRIMARY KEY(id)) ENGINE = InnoDB

C:\wamp\www\Symfony>php app\console doctrine:schema:update --dump-sql

ALTER TABLE comments ADD article_id INT DEFAULT NULL;
ALTER TABLE comments ADD CONSTRAINT FK_A6E8F47C7294869C FOREIGN KEY (article_id)
 REFERENCES Articles(id);
CREATE INDEX IDX_A6E8F47C7294869C ON comments (article_id)

C:\wamp\www\Symfony>php app\console doctrine:schema:drop --dump-sql

ALTER TABLE comments DROP FOREIGN KEY FK_A6E8F47C7294869C;
DROP TABLE articles;
DROP TABLE comments

Resumen Final

Como vemos, el framework Doctrine, al tener los datos de la base de datos y de las tablas, nos proporciona un soporte muy potente para trabajar con ellas y eso que solo hemos visto la parte de creación, borrado y modificación de tablas. En los siguientes capítulos trabajaremos manipulando los datos de las tablas y le proporcionaremos aún más información a nuestras entidades para ayudarnos a validar los datos ingresados en nuestros campos.

