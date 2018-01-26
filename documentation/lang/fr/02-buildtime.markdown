---
layout: documentation
title: The Build Time
configuration: true
---

# L'étape de Build #

Dans tout projet avec Propel, l'une des étapes primordiales est le "build". Durant cette étape de construction, le développeur va décrire la structure de la base de données attendue dans un fichier XML désigné comme "schema".

Propel se servira de ce schéma pour générer des classes PHP désignées comme "modèles de classes", dont les méthodes permettront des interactions avec une base de données relationnelle choisie : lire des données, les modifier, etc.

Le schéma XML peut aussi être utilisé pour générer du code SQL permettant de construire ou éditer la base de données.
A l'inverse, il est aussi possible de générer ce schéma XML en se servant d'une base de données existante comme support (voir le guide [Travailler avec une base de données existante](/documentation/cookbook/working-with-existing-databases.html) pour plus de détails).

Durant l'étape de build, le développeur définira aussi les paramètres de connexion à la base de données.

Pour illustrer les fonctionnalités du Build de Propel, ce chapitre prend en exemple la base de données qui pourrait être issue d'un magasin de livres. Elle serait constituée de 3 tables : la table `book` qui porte une clef étrangère (foreign key) vers les deux autres tables `author` et `publisher`.

Create a `bookstore` directory then setup Propel into it as covered in the previous
chapter. This will be the root of the bookstore project.

Il existe deux façons de faire la configuration du build : une dure (manuelle), et une facile (en lignes de commandes). Ce chapitre présente les deux méthodes, la dure étant la plus appropriée pour découvrir la structure de Propel et la configuration avancée de votre projet, tandis que la facile permet de simplifier l'usage.

## La méthode dure ##

### Décrire la base de données dans le schéma XML ###

Propel génère ses classes PHP based on a _relational_ description of your data
model. Ce "schéma" utilise XML pour décrire les tables, leurs colonnes et les relations entre chaque (clefs étrangères, ...). The
schema syntax closely follows the actual structure of the database (you have
to describe any excluded tables inside your
[configuration file](/documentation/reference/configuration-file.html#exclude-tables)).

#### Database Connection Name ####

Créer un fichier `schema.xml` dans le nouveau dossier `bookstore/`

The root tag of the XML schema is the `<database>` tag:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <!-- table definitions go here -->
</database>
```

L'attribut `name` définit le nom de la connexion que Propel utilisera pour les tables de ce schéma. Il n'est pas nécessaire que ce nom de connexion soit le nom réel de la base de données, Propel permettant d'associer un nom de connexion à des données de connexion à une base de données (nom d'user, mot de passe et nom de la base de données).

The `defaultIdMethod` attribute indicates that the tables in this schema use the database's "native" auto-increment/sequence features to handle id columns that are set to auto-increment.

> **Tip** You can define several schemas for a single project. Just make sure
> that each of the schema filenames end with `schema.xml`.

#### Les tables et leurs colonnes ####

Within the `<database>` tag, Propel expects a `<table>` tag for each table:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <table name="book" phpName="Book">
    <!-- column and foreign key definitions go here -->
  </table>
  <table name="author" phpName="Author">
    <!-- column and foreign key definitions go here -->
  </table>
  <table name="publisher" phpName="Publisher">
    <!-- column and foreign key definitions go here -->
  </table>
</database>
```

This time, the `name` attributes are the real table names. The `phpName` is the name that Propel will use for the generated PHP class. By default, Propel uses a CamelCase version of the table name as its phpName - that means that you could omit the `phpName` attribute in the example above.

Within each set of `<table>` tags, define the columns that belong to that table:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <table name="book" phpName="Book">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="title" type="varchar" size="255" required="true" />
    <column name="isbn" type="varchar" size="24" required="true" phpName="ISBN"/>
    <column name="publisher_id" type="integer" required="true"/>
    <column name="author_id" type="integer" required="true"/>
  </table>
  <table name="author" phpName="Author">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="first_name" type="varchar" size="128" required="true"/>
    <column name="last_name" type="varchar" size="128" required="true"/>
  </table>
  <table name="publisher" phpName="Publisher">
   <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true" />
   <column name="name" type="varchar" size="128" required="true" />
  </table>
</database>
```

Each column has a `name` (the one used by the database), and an optional `phpName` attribute. Once again, the Propel default behavior is to use a CamelCase version of the `name` as `phpName` when not specified.

Each column also requires a `type`. The XML schema is database agnostic, so the
column types and attributes are probably not exactly the same as the ones you
use in your own database. But Propel knows how to map the schema types with SQL
types for many database vendors. Check the [schema reference][schema] for more
details on each column type.

As for the other column attributes, `required`, `primaryKey`, and `autoIncrement`, they mean exactly what their names imply.

> **Tip** If you specify a `namespace` attribute in a `<table>` element, the
> generated PHP classes for this table will use this namespace.

#### Foreign Keys ####

A table can have several `<foreign-key>` tags, describing foreign keys to foreign tables. Each `<foreign-key>` tag consists of one or more mappings between a local column and a foreign column.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<database name="bookstore" defaultIdMethod="native">
  <table name="book" phpName="Book">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="title" type="varchar" size="255" required="true" />
    <column name="isbn" type="varchar" size="24" required="true" phpName="ISBN"/>
    <column name="publisher_id" type="integer" required="true"/>
    <column name="author_id" type="integer" required="true"/>
    <foreign-key foreignTable="publisher" phpName="Publisher" refPhpName="Book">
      <reference local="publisher_id" foreign="id"/>
    </foreign-key>
    <foreign-key foreignTable="author">
      <reference local="author_id" foreign="id"/>
    </foreign-key>
  </table>
  <table name="author" phpName="Author">
    <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true"/>
    <column name="first_name" type="varchar" size="128" required="true"/>
    <column name="last_name" type="varchar" size="128" required="true"/>
  </table>
  <table name="publisher" phpName="Publisher">
   <column name="id" type="integer" required="true" primaryKey="true" autoIncrement="true" />
   <column name="name" type="varchar" size="128" required="true" />
  </table>
</database>
```

Une clef étrangère (aussi appelée "foreign key") représente une relation. Tout comme une table ou une colonne, une relation possède un `phpName`. Par défaut, Propel utilise le `phpName` de la table étrangère comme `phpName` de la relation. Le `refPhpName` définit le nom de la relation as seen from the foreign table.

There are many more attributes and elements available to describe a datamodel.
Propel's documentation provides a complete [reference of the schema syntax][schema],
together with a [DTD][DTD] and a [XSD][XSD] schema for its validation.

### Building The Model ###

#### Setting Up Configuration ####

The build process is highly customizable. Whether you need the generated classes to inherit one of your classes rather than Propel's base classes, or to enable/disable some methods in the generated classes, pretty much every customization is possible. Of course, Propel provides sensible defaults, so that you actually need to define only two settings for the build process to start: the RDBMS you are going to use, and a name for your project.

Propel expects the configuration to be stored in a file called `propel.ext`, where `.ext` stands for "one of the supported extensions" and stored at the same level as the `schema.xml`, or in a subdirectory named `conf` or `config`. See the [configuration chapter](10-configuration.html) of the documentation, for a detailed description.
Here's a minimal example configuration file for MySQL, for each supported format

<div class="conftabs">
<ul>
<li><a href="#tabyaml">propel.yaml</a></li>
<li><a href="#tabphp">propel.php</a></li>
<li><a href="#tabjson">propel.json</a></li>
<li><a href="#tabini">propel.ini</a></li>
<li><a href="#tabxml">propel.xml</a></li>
</ul>
<div id="tabyaml">
{% highlight yaml %}
propel:
  database:
      connections:
          bookstore:
              adapter: mysql
              classname: Propel\Runtime\Connection\ConnectionWrapper
              dsn: "mysql:host=localhost;dbname=my_db_name"
              user: my_db_user
              password: s3cr3t
              attributes:
  runtime:
      defaultConnection: bookstore
      connections:
          - bookstore
  generator:
      defaultConnection: bookstore
      connections:
          - bookstore
{% endhighlight %}
</div>
<div id="tabphp">
{% highlight php %}
<?php

return [
    'propel' => [
        'database' => [
            'connections' => [
                'bookstore' => [
                    'adapter'    => 'mysql',
                    'classname'  => 'Propel\Runtime\Connection\ConnectionWrapper',
                    'dsn'        => 'mysql:host=localhost;dbname=my_db_name',
                    'user'       => 'my_db_user',
                    'password'   => 's3cr3t',
                    'attributes' => []
                ]
            ]
        ],
        'runtime' => [
            'defaultConnection' => 'bookstore',
            'connections' => ['bookstore']
        ],
        'generator' => [
            'defaultConnection' => 'bookstore',
            'connections' => ['bookstore']
        ]
    ]
];
{% endhighlight %}
</div>
<div id="tabjson">
{% highlight json %}
{
    "propel": {
        "database": {
            "connections": {
                "bookstore": {
                    "adapter": "mysql",
                    "classname": "Propel\Runtime\Connection\ConnectionWrapper",
                    "dsn": "mysql:host=localhost;dbname=my_db_name",
                    "user": "my_db_user",
                    "password": "s3cr3t",
                    "attributes": []
                }
            }
        },
        "runtime": {
            "defaultConnection": "bookstore",
            "connections": ["bookstore"]
        },
        "generator": {
            "defaultConnection": "bookstore",
            "connections": ["bookstore"]
        }
    }
}
{% endhighlight %}
</div>
<div id="tabini">
{% highlight ini %}
[propel]
;
; Database section
;
database.connections.bookstore.adapter    = mysql
database.connections.bookstore.classname  = Propel\Runtime\Connection\ConnectionWrapper
database.connections.bookstore.dsn        = mysql:host=localhost;dbname=my_db_name
database.connections.bookstore.user       = my_db_user
database.connections.bookstore.password   = s3cr3t
database.connections.bookstore.attributes =

;
; Runtime section
;
runtime.defaultConnection = bookstore
runtime.connections[0]    = bookstore

;
; Generator section
;
generator.defaultConnection = bookstore
generator.connections[0] = bookstore
{% endhighlight %}
</div>
<div id="tabxml">
{% highlight xml %}
<?xml version="1.0" encoding="ISO-8859-1" standalone="no"?>
<config>
    <propel>
        <database>
            <connections>
                <connection id="bookstore">
                    <adapter>mysql</adapter>
                    <classname>Propel\Runtime\Connection\ConnectionWrapper</classname>
                    <dsn>mysql:host=localhost;dbname=my_db_name</dsn>
                    <user>my_db_user</user>
                    <password>s3cr3t</password>
                    <attributes></attributes>
                </connection>
            </connections>
        </database>
        <runtime>
            <defaultConnection>bookstore</defaultConnection>
            <connection>bookstore</connection>
        </runtime>
        <generator>
            <defaultConnection>bookstore</defaultConnection>
            <connection>bookstore</connection>
        </generator>
    </propel>
</config>
{% endhighlight %}
</div>
</div>

Use your own database vendor driver, chosen among pgsql, mysql, sqlite, mssql
and oracle.

You can learn more about the available build settings and their possible values in
the [configuration reference](/documentation/reference/configuration-file.html).

#### Setup UTF-8 ####

If you want to save anything in UTF-8 in your database then depending on your database you have to set some extra configuration values.

<div class="conftabs">
<ul>
<li><a href="#tabMysql">MySQL</a></li>
<li><a href="#tabPgsql">PostgreSQL</a></li>
<li><a href="#tabSqlite">SQLite</a></li>
<li><a href="#tabMssql">MSSQL</a></li>
<li><a href="#tabOracle">Oracle</a></li>
</ul>
<div id="tabMysql">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: mysql
        settings:
          charset: utf8mb4
          queries:
            utf8: "SET NAMES utf8mb4 COLLATE utf8mb4_unicode_ci, COLLATION_CONNECTION = utf8mb4_unicode_ci, COLLATION_DATABASE = utf8mb4_unicode_ci, COLLATION_SERVER = utf8mb4_unicode_ci"
{% endhighlight %}
</div>
<div id="tabPgsql">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: pgsql
        settings:
          charset: utf8
          queries:
            utf8: "SET NAMES 'UTF8'"
{% endhighlight %}
</div>
<div id="tabSqlite">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: sqlite
        settings:
          charset: utf8
{% endhighlight %}
</div>
<div id="tabMssql">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: mssql
        settings:
          charset: UTF-8
{% endhighlight %}
</div>
<div id="tabOracle">
{% highlight yaml %}
propel:
  database:
    connections:
      default:
        adapter: oracle
        settings:
          charset: UTF8
{% endhighlight %}
</div>
</div>

#### Utiliser le script `propel` pour construire le fichier SQL ####

Comme expliqué dans le chapitre précédent, Propel dispose d'un script permettant de réaliser différentes actions, dont notamment la génération de schémas.

Dans un terminal, se placer dans le dossier contenant les schémas de la base de données (`schema.xml` et `propel.ext`), puis lancer la commande suivante pour générer un fichier SQL :

```bash
$ propel sql:build
```

#### Générer des classes modèles ####

Maintenant que la base de données est prête, générons les fichiers modèles. Ces fichiers seront simplement des classes permettant une interaction facilitée avec les différentes tables de la base de données. Pour lancer la génération de ces fichiers, faire :

```bash
$ propel model:build
```

Propel génèrera un nouveau dossier `generated-classes` contenant tout le nécessaire pour interagir avec les tables.

Ainsi, pour chaque table de la base de données, Propel crée 3 classes :

* a _model_ class (e.g. `Book`), which represents a row in the database;
* a _tablemap_ class (e.g. `Map\BookTableMap`), offering static constants and methods mostly for compatibility with previous Propel versions;
* a _query_ class (e.g. `BookQuery`), used to operate on a table to retrieve and update rows

Propel uses the `phpName` attribute of each table as the base for the PHP class names.

Chacune de ces classes est vide, mais hérite d'une classe portant le même nom et située dans le dossier `generated-classes/Base` :

```php
<?php

/**
 * Skeleton subclass for representing a row from the 'book' table.
 */
class Book extends BaseBook
{

}
```

These empty classes are called _stub_ classes. This is where you will add your own model code. These classes are generated only once by Propel ; on the other hand, the _base_ classes they extend are overwritten every time you call the `model:build` command, and that happens a lot in the course of a project, because the schema evolves with your needs.


Propel also generates one `TableMap` class for each table under the `Map/`
directory. You will probably never use the map classes directly, but Propel needs
them to get metadata information about the table structure at runtime.

> **Tip** Never add any code of your own to the classes generated by Propel in the
> `Map/` directory; this code would be lost next time you call the `model:build`
> command.

Basically, all that means is that despite the fact that Propel generates _five_ classes for each table, you should only care about two of them: the model class and the query class.


> **Attention** Après la génération des classes, il est impératif d'en rajouter le dossier à l'autoload. Ainsi, avec Composer par exemple, il faudra les inclure de la manière suivante :
>
> ```json
> {
>   ...
>   "autoload": {
>     "classmap": ["generated-classes/"]
>   }
> }
> ```
>
Puis, exécuter la commande `composer dump-autoload`. See also
> [namespaces recipe](../cookbook/namespaces.html) if you prefer to autoload
> your model classes following PSR-0.

### Runtime Connection Settings ###

The database and PHP classes are now ready to be used. But they don't know yet
how to communicate with each other at runtime. You must tell Propel which database
connection settings should be used to finish the setup.

For performance reasons, Propel prefers to use a PHP version of the connection
settings rather than read it from the configuration file every time. So you must
use the `propel` script one last time to build the PHP version of the runtime
configuration:

```bash
$ propel config:convert
```

The `config:convert` command reads the `runtime` section of the configuration file
and generates the relative PHP script. The resulting file can be found under
`generated-conf/config.php`.

You can now setup Propel with the following script:

```php
<?php

// setup the autoloading
require_once '/path/to/vendor/autoload.php';

// setup Propel
require_once '/generated-conf/config.php';
```

It's also a good practice to add a logger to the service container, so that Propel
can log warnings and errors. You can do so by adding the following code to the
setup script:

```php
<?php

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$defaultLogger = new Logger('defaultLogger');
$defaultLogger->pushHandler(new StreamHandler('/var/log/propel.log', Logger::WARNING));

$serviceContainer->setLogger('defaultLogger', $defaultLogger);
```

> **Tip** You may wish to write the setup code in a standalone script that is
included at the beginning of your PHP files.

Now you are ready to start using your model classes!

#### Créer la base de données à partir du schéma généré ####

Now that your project is fully set up, you have to create the generated schema
in your database.

Before inserting it, you should create a database, let's say we want to call it
`bookstore`. If you are using MySQL for instance, just run:

```bash
$ mysqladmin -u root -p create bookstore
```

Insérer ensuite les commandes SQL dans la base de données :

```bash
$ propel sql:insert
```

You should normally have your tables created. Propel will also generate a
`generated-sql` folder containning the SQL files of your schema ; useful if you
are using a SCM, you can so compare the different versions of your schema.

Each time you will update your schema, you should run `sql:build` and `sql:insert`.

Depending on which RDBMS you are using, it may be normal to see some errors
(e.g. "unable to DROP...") when you first run this command. This is because some
databases have no way of checking to see whether a database object exists before
attempting to DROP it (MySQL is a notable exception). It is safe to disregard
these errors, and you can always run the script a second time to make sure that
the errors are no longer present.

> **Attention** Le fichier `schema.sql` EFFACERA TOUTES LES TABLES EXISTANTES avant d'en créer (cela vide donc la base de données avant de la recréer).

## La méthode facile ##

Pour faciliter l'étape de Build, Propel dispose d'une commande `init` qui vous guidera durant ce processus. Dans un terminal, taper :

```bash
$ propel init
```

Propel vous demandera les informations d'accès à la base de données, ainsi que les identifiants d'utilisateur.

Puis, Propel vous demandera si vous désirez importer une base de données existante dans votre projet, et où stocker les différents fichiers propres à Propel tels que le schéma

Enfin, précisez le format du fichier de coniguration. Par défaut, YAML est proposé, mais vous pouvez choisir le format à votre préférence, la documentation expliquant chacun d'entre eux.

The command will also output a summary of your current configuration that you can
modify if you want to. Propel est maintenant prêt à être utilisé !

---
<span class="next">[Suivant: Le CRUD de base &rarr;](03-basic-crud.html)</span>

[schema]: /documentation/reference/schema.html
[DTD]: https://github.com/propelorm/Propel2/blob/master/resources/dtd/database.dtd
[XSD]: https://github.com/propelorm/Propel2/blob/master/resources/xsd/database.xsd
