---
typora-copy-images-to: assets
---

# 2. Object Relational Mapping

<iframe width="960" height="540" src="https://www.youtube.com/embed/kgH-9r4_3kc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Un ORM (Object Relational Mapping) es un framework encargado de tratar con una base de datos relacional (conectar con ella, realizar operaciones de consulta, inserción, etc.), de forma que, de cara a la aplicación, se convierten a objetos todos los elementos que se extraigan de la base de datos, y viceversa (los objetos de la aplicación se transforman en registros de la base de datos, llegado el caso).

De esta forma, el ORM se encargará de realizar esta conversión o mapeo automáticamente por nosotros. Definiendo una serie de reglas, indicaremos qué tablas de la base de datos relacional se corresponden con qué clases de nuestro modelo, y qué campos de cada tabla se corresponden con qué atributos de cada clase. A partir de ahí, el ORM se encargará de extraer la información de la base de datos y crear los objetos correspondientes, o de convertir los objetos con sus atributos en registros de la base de datos, con sus correspondientes columnas.

La principal ventaja de utilizar un ORM como Doctrine es aislar la aplicación del gestor de base de datos que hayamos elegido (MySQL, Oracle, PostgreSQL...) ya que a nivel de aplicación trabajaremos con objetos, y será Doctrine quien se encargue de conectar con la base de datos elegida, y transformar los objetos para adaptarlos a la misma.

## 2.1 Configuración básica de Doctrine

Para poder utilizar Doctrine, tenemos que indicar cómo conectar al servidor de base de datos que vayamos a utilizar. Estos parámetros de conexión se pueden configurar en el archivo `.env` de nuestro proyecto. Este es un archivo donde se definen ciertas variables propias de entorno, que luego se procesan y se convierten en variables reales. En nuestro caso, definimos una llamada `DATABASE_URL`, con una `URL` donde se especifican tanto la dirección y puerto de conexión a la base de datos, como el `login` y `password` necesarios para acceder, y el nombre de la base de datos a la que conectar. Por ejemplo, para una base de datos MySQL, la estructura general será ésta:

```
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
```

En nuestro caso, será

```
DATABASE_URL=mysql://root:sa@127.0.0.1:3306/contactos
```

En el caso de que la base de datos aún no exista, Doctrine puede crearla por nosotros.
Para ello, basta con escribir el siguiente comando:

```
php bin/console doctrine:database:create
```
Automáticamente, se tomará el nombre de la base de datos de la variable de entorno anterior, se conectará al servidor y se creará (sin tablas, de momento).

## 2.2 Creación de entidades

Las entidades son las clases que van a componer el modelo de datos de nuestra aplicación. Por ejemplo, para nuestra aplicación de contactos, necesitaremos una entidad/clase llamada `Contacto` que almacene los datos concretos de cada contacto (código, nombre, teléfono y e­mail).
Para crear una entidad, empleamos el siguiente comando desde el terminal (dentro de la carpeta principal de nuestro proyecto Symfony):

```
php bin/console make:entity
```

Se iniciará un asistente que nos irá pidiendo información para construir la entidad:

* Nombre de la clase o entidad
* Propiedades o atributos de la clase, para cada uno, pedirá el nombre (si directamente pulsamos Intro dejará de pedirnos más datos), el tipo de dato, la longitud o tamaño del campo, si admite nulos...



<script id="asciicast-bc8O8iC56qHQ33JnAuvjC1EBT" src="https://asciinema.org/a/bc8O8iC56qHQ33JnAuvjC1EBT.js" async data-size="medium"></script>

Como resultado, se generará una clase `Contacto` dentro de la carpeta `src/Entity`. El código queda como sigue:

```php
<?php

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="App\Repository\ContactoRepository")
 */
class Contacto
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=255)
     */
    private $nombre;

    /**
     * @ORM\Column(type="string", length=15)
     */
    private $telefono;

    /**
     * @ORM\Column(type="string", length=255)
     */
    private $email;
    
    public function getId()
    {
        return $this->id;
    }

    public function getNombre(): ?string
    {
        return $this->nombre;
    }

    public function setNombre(string $nombre): self
    {
        $this->nombre = $nombre;

        return $this;
    }

    public function getTelefono(): ?string
    {
        return $this->telefono;
    }

    public function setTelefono(string $telefono): self
    {
        $this->telefono = $telefono;

        return $this;
    }

    public function getEmail(): ?string
    {
        return $this->email;
    }

    public function setEmail(string $email): self
    {
        $this->email = $email;

        return $this;
    }
}
```

Como podemos observar, el campo `codigo` que usábamos en nuestra base de datos de prueba lo hemos reemplazado por un `id` autonumérico que se genera automáticamente como clave principal de la clase. Por lo tanto, sólo hemos tenido que especificar el `nombre`, `teléfono` y `e­mail`, de tipo **string**.

En cuanto a los tipos de datos que podemos especificar, si pulsamos `?` e `Intro` cuando vayamos a especificar el tipo de dato, veremos un listado completo de los tipos disponibles (también lo podéis consultar [aquí](https://www.doctrine-project.org/projects/doctrine-orm/en/2.6/reference/basic-mapping.html#doctrine-mapping-types)). Lo habitual será trabajar con cadenas de texto de una longitud determinada (`string`), textos ilimitados (`text`), enteros (`integer`), booleanos (`boolean`), reales (`float`), fechas (`date`, `time` o `datetime`, dependiendo de lo que queramos almacenar)...

## 2.3 Generación del esquema

Una vez hemos definida la entidad, podemos generar la correspondiente tabla en la base de datos. Para ello, escribimos este comando:

```
php bin/console make:migration
```

Lo que hace este comando es cotejar los cambios entre nuestro modelo de entidades y el esquema de la base de datos, y generar un archivo PHP que se encargará de volcar esos cambios a la base de datos. Por consola se nos informará de dónde está este archivo para que lo comprobemos (estará en la carpeta `src/Migrations`), y si todo es correcto, ejecutando este otro comando se reflejarán los cambios en la base de datos:

```
php bin/console doctrine:migration:migrate
```

![1549382168090](assets/1549382168090.png)



## 2.4 Editar entidades

¿Qué pasa si, tras crear una entidad, queremos modificar su estructura? Podemos editar la clase de la entidad manualmente para añadir, modificar o borrar campos, pero también podemos volver a ejecutar el comando `make:entity`, indicar el mismo nombre de clase que queremos modificar, y especificar los nuevos campos que queramos añadir (en el caso de que lo que queramos sea añadir campos).
Después de definir los cambios en la(s) entidad(es) deseada(s), deberemos generar una nueva migración con los comandos vistos en el subapartado anterior.

## 2.5 Establecer claves primarias

Por defecto, hemos visto que Doctrine agrega un campo id a las entidades, que es autonumérico y actúa como clave primaria. En el caso de que no queramos que sea así, y prefiramos elegir otro campo no autonumérico como clave primaria, debemos seguir estos pasos:

* Eliminar el atributo id y su getter correspondiente de la entidad

* Añadir la siguiente anotación al atributo que hayamos elegido como clave primaria:

  ```php
  /**
  * @ORM\Id()
  * ...
  */
  private $nombreCampo;
  ```

En el caso de que sea una clave primaria compuesta por más de un campo, deberemos añadir esta anotación en cada campo que forme parte de la clave primaria.

## 2.6 Operaciones contra la base de datos

<iframe width="960" height="540" src="https://www.youtube.com/embed/NO7dJb44rcM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Ahora que ya hemos visto cómo definir entidades simples, veamos cómo realizar operaciones con ellas, tales como inserciones, borrados, modificaciones y consultas. Para realizar estas operaciones, nos valdremos de un objeto muy importante en Doctrine, su `entity manager`, a través del cual haremos las inserciones, borrados, etc. También utilizaremos el repositorio de la entidad correspondiente, para realizar las búsquedas.

### 2.6.1 Insertar objetos

Si queremos añadir objetos nuevos a nuestra base de datos, basta con que creemos un objeto de la entidad correspondiente en el método oportuno, y llamemos al método `persist` y `flush` del `entity manager` de Doctrine.
Por ejemplo, para probar, vamos a crear un controlador en nuestra clase `ContactoController` asociado a una ruta `/contacto/insertar`, que de momento será de pruebas hasta que hagamos un formulario de inserción. Dentro de este método, creamos los objetos `Contacto` a partir del array que hemos creado anteriormente, obtenemos el `entity manager` de Doctrine y persistimos el objeto:

![image-20220109165702135](assets/image-20220109165702135.png)

> **NOTA**: es **importante** que ubiquemos este controlador **antes controlador `ficha`**, ya que de lo contrario se disparará este último al escribir la URL `/contacto/insertar`.

Si accedemos desde el navegador a la ruta http://127.0.0.1:8080/contacto/insertar, podremos ver el resultado en la tabla contacto de nuestra base de datos:

![1549382698610](assets/1549382698610.png)



Es **importante** recalcar que la llamada a `persist` por sí sola no actualiza la base de datos, sino que indica que se quiere persistir el objeto indicado. Es la llamada a `flush` la que hace efectiva esa persistencia.

### 2.6.2 Obtener objetos

Los objetos siempre se obtienen de un repositorio, pero no hace falta especificarlo como hacíamos hasta ahora sino que lo hace automáticamente Doctrine:

```php
 $repositorio = $doctrine->getRepository(Contacto::class);
```

donde lo único que varía es el nombre de la clase.

A la hora de obtener objetos de una tabla, existen diferentes métodos que podemos emplear. Por ejemplo:

* El método `find` localiza el objeto por la clave primaria (normalmente el id) que se le pasa como parámetro. Así buscaríamos el contacto con id 1:

  ```php
  $contacto = $repositorio->find(1);
  ```

* El método `findOneBy` localiza un objeto que cumpla los criterios de búsqueda pasados como parámetro. Así buscaríamos el contacto cuyo teléfono sea “900110011”:

  ```php
  $contacto = $repositorio->findOneBy(["telefono" => "54565859"]);
  ```

  En el caso de querer definir más criterios de búsqueda, se pasarían uno tras otro en el array, separados por comas.

* El método `findBy` localiza todos los objetos que cumplan los criterios de búsqueda pasados como parámetro. Esta instrucción es como la anterior, pero devuelve un array de contactos con todos los resultados coincidentes:

  ```php
  $contactos = $repositorio->findBy(["telefono" => "54565859"]);
  ```

* El método `findAll` (sin parámetros), obtiene todos los objetos de la colección.

  ```php
  $contactos = $repositorio->findAll();
  ```

Todos estos métodos se obtienen a partir de un repositorio de la clase, que viene a ser algo así como un asistente que nos ayuda a obtener objetos que pertenezcan a esa clase.

Veamos un ejemplo con nuestra clase `ContactoController`: vamos a modificar nuestro método ficha para que, en lugar de buscar en la base de datos de prueba que hemos venido empleando en sesiones anteriores, busque por id en la base de datos real. Para ello, obtenemos el repositorio de nuestra clase `Contacto` y buscamos (`find`) el contacto con el id que hemos recibido como parámetro:

![image-20220109165736024](assets/image-20220109165736024.png)

#### 2.6.2.1 Consultas más avanzadas

Con los métodos de consulta anteriores podemos realizar consultas que se limitan a comprobar si uno o varios campos de un objeto son iguales a unos criterios de búsqueda determinados. Pero, ¿cómo podríamos, por ejemplo, buscar los contactos cuyo nombre contenga un cierto texto, o los libros de más de 100 páginas? Para este tipo de consultas, necesitamos ampliar el repositorio de nuestra entidad.

Por ejemplo, para nuestra entidad `Contacto`, imaginemos que queremos buscar los contactos cuyo nombre concierta un cierto texto. Para conseguir esto, necesitamos editar el repositorio de la entidad, que está en `src/Repository/ContactoRepository.php`. Este archivo contiene comentados un par de métodos de prueba que podríamos definir para ampliar las capacidades de la entidad.

En nuestro caso, vamos a añadir un método que se encargará de obtener los contactos cuyo nombre contenga un texto determinado que le pasemos como parámetro:

![image-20220109165749027](assets/image-20220109165749027.png)

Empleamos el **query builder** de Doctrine para construir la consulta con esa sintaxis específica. En primer lugar, definimos un elemento (alias) que hemos llamado `c` (de `Contacto`) que usaremos para referenciar las propiedades de los contactos, por ejemplo, en la cláusula `where`. Lo que viene a hacer este código es buscar aquellos contactos `c` cuyo nombre sea como el parámetro `text`, y a continuación especifica que dicho parámetro `text` es igual al parámetro que recibimos en el método, encerrado entre símbolos `'%'`, para indicar que da igual lo que haya delante o detrás del texto.

Ahora, ya podríamos utilizar este método desde donde lo necesitemos. Por ejemplo, podemos modificar el método buscar de `ContactoController` para que busque contactos por nombre empleando este nuevo método:

![image-20220109165805283](assets/image-20220109165805283.png)

Si, por ejemplo, quisiéramos buscar por una propiedad numérica (por ejemplo, personas cuya edad sea mayor que una dada), usaríamos una sintaxis como esta (también muy similar a SQL):
```php
$qb = $this->createQueryBuilder('p')
->andWhere('p.edad > :edad')
->setParameter('edad', $edad)
->getQuery();
```

Alternativamente, también podemos emplear un lenguaje llamado `DQL` (Doctrine Query Language) para realizar la consulta anterior:

```php
public function findByName($text): array
{

    $entityManager = $this->getEntityManager();
    $query = $entityManager->createQuery(
        'SELECT c FROM App\Entity\Contacto c WHERE c.nombre LIKE :text'
    )->setParameter('text', '%' . $text . '%');

    return $query->execute();        
}
```

Y, como tercera vía, también podemos emplear SQL estándar, pero en este caso lo que obtendríamos ya no sería un array de objetos, sino un array de registros, como si empleáramos la librería mysqli de PHP para acceder a la base de datos.

Aquí tenéis enlaces para consultar información adicional tanto de [Query Builder](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html) como del lenguaje [DQL](https://www.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html).

### 2.6.3 Actualizar objetos

Para actualizar un objeto en una base de datos, debemos seguir tres pasos:

* Obtener el objeto de la base de datos (típicamente haciendo un `find` por su clave primaria)
* Modificar los datos necesarios con los respectivos `setters` del objeto
* Hacer un `flush` para actualizar los cambios en la base de datos.

Si, por ejemplo, quisiéramos actualizar los datos de un contacto haríamos esto:

![image-20220109165827535](assets/image-20220109165827535.png)

### 2.6.4 Borrar objetos

El borrado de objetos es similar a la actualización: debemos obtener el objeto también, pero después llamamos al método `remove` para borrarlo, y finalmente a `flush`. 

Por ejemplo:

![image-20220109165850835](assets/image-20220109165850835.png)

Nuevamente, tanto en la actualización como en el borrado, el método `flush` puede provocar una **excepción** si la operación no ha podido llevarse a cabo. Debemos tenerlo en cuenta para capturarla y generar la respuesta oportuna.

## 2.7 Relaciones entre entidades

<iframe width="960" height="540" src="https://www.youtube.com/embed/z7RhQp9KpsE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Hasta ahora las operaciones que hemos hecho se han centrado en una única tabla o entidad (la entidad/tabla de `contactos`). Veamos ahora cómo podemos trabajar con más de una `tabla/entidad` que estén relacionadas entre sí.

Existen dos tipos principales de relaciones entre entidades:

* **Muchos a uno**: en este tipo se englobarían las relaciones “uno a muchos”, “muchos a uno” y “uno a uno”, ya que en cualquiera de los tres casos, la relación se refleja añadiendo una clave ajena en una de las dos entidades que referencie a la otra.
* **Muchos a muchos**: en este tipo de relaciones, se necesita de una tabla adicional para reflejar la relación entre las entidades.

Vamos a definir una relación muchos a uno en nuestra base de datos de contactos. Para ello, vamos a crear primero una entidad llamada `Provincia`, que sólo contenga un `id` autogenerado y un `nombre` (string):

```
php bin/console make:entity
```

 Y definimos los campos:

<script id="asciicast-seD4fxqXUY9wSCkjJEUk5sqfj" src="https://asciinema.org/a/seD4fxqXUY9wSCkjJEUk5sqfj.js" async data-size="medium"></script>

Tras generar la nueva entidad, creamos la correspondiente tabla en la base de datos a través de la migración.

```
php bin/console make:migration
php bin/console doctrine:migration:migrate
```

Ahora, vamos a hacer que los contactos tengan una provincia asociada. Para ello, editamos la entidad `Contacto` y le añadimos un nuevo campo, llamado `provincia`, que será de tipo relación muchos a uno (un contacto pertenecerá a una provincia, y una provincia puede tener muchos contactos).

<script id="asciicast-EIgPamUamfQOm4N6xS2vIzXGP" src="https://asciinema.org/a/EIgPamUamfQOm4N6xS2vIzXGP.js" async></script>

Como puede verse, a la hora de elegir el tipo de campo, indicamos que es una relación (`relation`), en cuyo caso nos pide indicar a qué entidad está vinculada (`Provincia`, en este caso), y qué tipo de relación es (`ManyToOne` en nuestro caso, pero podemos elegir cualquiera de las otras tres opciones `OneToMany`, `OneToOne` o `ManyToMany`). También podemos comprobar que el asistente nos pregunta si queremos añadir un campo en la otra entidad para que la relación sea bidireccional (es decir, para que desde un objeto de cualquiera de las dos entidades podamos consultar el/los objeto(s) asociado(s) de la otra. En este caso indicamos que **no** para simplificar el código.

Una vez creada la relación vamos a realizar la migración:

```
php bin/console make:migration
php bin/console doctrine:migration:migrate
```

Ya tendremos el nuevo campo añadido en nuestra entidad `Contacto` y a la tabla contacto de la base de datos:

```php
//src/Entity/Contacto
/**
* @ORM\ManyToOne(targetEntity="App\Entity\Provincia")
* @ORM\JoinColumn(nullable=false)
*/
private $provincia;
```

![1549386995547](assets/1549386995547.png)

### 2.7.1 Trabajar con entidades relacionadas

Ahora que ya sabemos relacionar entidades entre sí, ¿cómo podemos insertar una entidad que depende de otra, o acceder a los datos de una entidad desde la otra?

#### 2.7.1.1 Inserción de entidades relacionadas

Por ejemplo, si quisiéramos insertar un contacto asignándole una provincia:

* Si la provincia no existe, creamos un objeto de tipo `Provincia`, y después otro de tipo `Contacto`, estableciendo como provincia el objeto `Provincia` recién creado:
![image-20220109165914273](assets/image-20220109165914273.png)
  
* Ahora modificamos la plantilla `ficha_contacto.html.twig`
![image-20220109165941038](assets/image-20220109165941038.png)
  
  Añadiendo la provincia mediante`contacto.provincia.nombre`
  
* Si la provincia sí existe, la buscamos en la base de datos (con algún método `find` o similar) y después creamos el objeto `Contacto` y le asignamos ese objeto `Provincia`:
![image-20220109170043888](assets/image-20220109170043888.png)
  
  

#### 2.7.1.2 Búsqueda de entidades relacionadas

En el caso de que hagamos una búsqueda de una entidad que está relacionada con otra, el acceso a esa otra entidad es inmediato desde la primera. Por ejemplo, si quisiéramos saber el nombre de la provincia del contacto con código 1, haríamos algo así:

```php
$repositorio = $this->getDoctrine()->getRepository(Contacto::class);
$contacto = $repositorio->find(1);
$nombreProvincia = $contacto->getProvincia()->getNombre();
```

