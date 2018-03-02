---
layout: documentation
title: Migrations
---

# Les Migrations #

Au cours de la vie d'un projet, le Modèle de données reste rarement le même : de nouvelles tables apparaissent, et des tables existantes demandent parfois des modifications (changement d'une colonne, ajout d'un index, d'une clef étrangère...). Mettre à jour la structure de la base de données tout en préservant les données existantes est une préoccupation commune. Propel met à disposition des outils permettant facilement la _migration_ d'une structure de base de données et de données.

>**Astuce** Les migrations avec Propel sont supportées sous MySQL, SQLite et PostgreSQL.

## Feuille de route pour une Migration ##

Le processus pour une migration avec Propel est très simple :

1. Editer le schéma XML pour modifier le modèle de la base de données
2. Appeler la commande `diff` de Propel pour créer une classe de migration dans le dossier `generated-migrations` qui contiendra les instructions SQL permettant la modification de la structure de la base de donnnées
3. Passer en revue la classe de migration ainsi générée, et ajuster le code si nécessaire
4. Exécuter la migration en utilisant la commande `migrate` de Propel.

Prenons un cas concret... Pour le site d'un libraire, un développeur crée un schéma xml avec une unique table `book` :

```xml
<database name="bookstore" defaultIdMethod="native">
  <table name="book" description="Book Table">
    <column name="id" type="integer" primaryKey="true" autoIncrement="true" />    <column name="title" type="varchar" required="true" primaryString="true" />
    <column name="isbn" required="true" type="varchar" size="24" phpName="ISBN" />
  </table>
</database>
```

Puis, le développeur utilise la commande `diff` de Propel pour comparer la structure de la base de données existante et ce schéma xml :

```bash
$ propel diff

[propel-sql-diff] Reading databases structure...
[propel-sql-diff] Database is empty
[propel-sql-diff] Loading XML schema files...
[propel-sql-diff] 1 tables found in 1 schema file.
[propel-sql-diff] Comparing models...
[propel-sql-diff] Structure of database was modified: 1 added table
[propel-sql-diff] "PropelMigration_1286483354.php" file successfully created in /path/to/project/build/migrations
[propel-sql-diff]   Please review the generated SQL statements, and add data migration code if necessary.
[propel-sql-diff]   Once the migration class is valid, call the "migrate" task to execute it.
```

Il est fortement conseillé de relire le fichier de migration généré dans `generated-migration/PropelMigration_[...]` pour vérifier les commandes SQL préparées. Il contient une classe disposant de deux méthodes, `getUpSQL()` et `getDownSQL()`, permettant respectivement de faire la migration pour adapter la structure de la base de données au schéma, ou d'annuler la migration effectuée.


```php
<?php
/**
 * Data object containing the SQL and PHP code to migrate the database
 * up to version 1286483354.
 * Generated on 2010-10-07 22:29:14 by francois
 */
class PropelMigration_1286483354
{

	public function getUpSQL()
	{
		return array('bookstore' => '
CREATE TABLE `book`
(
	`id` INTEGER NOT NULL AUTO_INCREMENT,
	`title` VARCHAR(255) NOT NULL,
	`isbn` VARCHAR(24) NOT NULL,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT=\'Book Table\';
',
);
	}

	public function getDownSQL()
	{
		return array('bookstore' => '
DROP TABLE IF EXISTS `book`;
',
);
	}
}
```

>**Astuce** Sur un projet utilisant un système de gestion de versions, il est important de commiter les classes de migration dans le repository. Ainsi, les autres développeurs récupérant le projet pourront appliquer les mêmes migrations et avoir une base de données à un stade identique.

Pour appliquer la création de la table `book` dans la base de données, le développeur lance ensuite la commande `migrate` :

```bash
$ propel migrate

[propel-migration] Executing migration PropelMigration_1286483354 up
[propel-migration] 1 of 1 SQL statements executed successfully on datasource "bookstore"
[propel-migration] Migration complete. No further migration to execute.
```

La table `book` est alors ajoutée à la base de données, et peut accueillir des données.

Après quelques jours, le développeur décide d'ajouter une nouvelle table `author`, vers laquelle pointe une clef étrangère (= foreign key) portée par la table `book`. Le schéma XML est modifié ainsi par ses soins :

```xml
<database name="bookstore" defaultIdMethod="native">
  <table name="book" description="Book Table">
    <column name="id" type="integer" primaryKey="true" autoIncrement="true" />
    <column name="title" type="varchar" required="true" primaryString="true" />
    <column name="isbn" required="true" type="varchar" size="24" phpName="ISBN" />
    <column name="author_id" type="integer" />
    <foreign-key foreignTable="author" onDelete="setnull" onUpdate="cascade">
      <reference local="author_id" foreign="id" />
    </foreign-key>
  </table>
  <table name="author">
    <column name="id" type="integer" primaryKey="true" autoIncrement="true" />
    <column name="first_name" type="varchar" />
    <column name="last_name" type="varchar" />
  </table>
</database>
```

Pour appliquer cette mise à jour à la structure de la base de données, le procédé est identique :

```bash
$ propel diff

[propel-sql-diff] Reading databases structure...
[propel-sql-diff] 1 tables imported from databases.
[propel-sql-diff] Loading XML schema files...
[propel-sql-diff] 2 tables found in 1 schema file.
[propel-sql-diff] Comparing models...
[propel-sql-diff] Structure of database was modified: 1 added table, 1 modified table
[propel-sql-diff] "PropelMigration_1286484196.php" file successfully created in /path/to/project/build/migrations
[propel-sql-diff]   Please review the generated SQL statements, and add data migration code if necessary.
[propel-sql-diff]   Once the migration class is valid, call the "migrate" task to execute it.

$ propel migrate

[propel-migration] Executing migration PropelMigration_1286484196 up
[propel-migration] 4 of 4 SQL statements executed successfully on datasource "bookstore"
[propel-migration] Migration complete. No further migration to execute.
```

Propel a exécuté le code `PropelMigration_1286484196::getUpSQL()`, qui modifie la structure de la table `book` _sans supprimer les données qu'elle contient_:

```sql
ALTER TABLE `book` ADD
(
	`author_id` INTEGER
);

CREATE INDEX `book_FI_1` ON `book` (`author_id`);

ALTER TABLE `book` ADD CONSTRAINT `book_FK_1`
	FOREIGN KEY (`author_id`)
	REFERENCES `author` (`id`)
	ON UPDATE CASCADE
	ON DELETE SET NULL;

CREATE TABLE `author`
(
	`id` INTEGER NOT NULL AUTO_INCREMENT,
	`first_name` VARCHAR(255),
	`last_name` VARCHAR(255),
	PRIMARY KEY (`id`)
) ENGINE=InnoDB;
```

>**Astuce** `diff` et `migrate` sont souvent utilisées successivement ; il est possible d'exécuter les deux à la fois en précisant en premier argument de la commande `propel` le chemin vers le projet en cours :

```bash
$ propel . diff migrate
```

## Les tâches de migration ##

Les deux commandes de migrations de base sont `diff` et `migrate` - vous les connaissez déjà. `diff` crée une classe de migration, tandis que `migrate` exécute ces migrations. Mais il y a aussi trois autres tâches de migration que vous trouverez certainement bien utiles.

### Migration Up ou Down, et One At A Time ###

Dans l'exemple précédent, deux migrations étaient exécutées. Mais le développeur voudrait maintenant inverser la dernière qu'il a faite. La tâche `migration:down` propose justement cette option : cela permet d'annuler une seule migration : la dernière effectuée.

```
$ propel migration:down

[propel-migration-down] Executing migration PropelMigration_1286484196 down
[propel-migration-down] 4 of 4 SQL statements executed successfully on datasource "bookstore"
[propel-migration-down] Reverse migration complete. 1 more migrations available for reverse.
```

Notez que le fichier de migration `PropelMigration_1286484196` est alors exécuté en _down_, et non plus en _up_ comme lors de la précédente migration. Il est possible d'appeler plusieurs fois la commande pour annuler plusieurs migrations succesives et rendre à la structure de la base de données son état d'origine :

```bash
$ propel migration:down

[propel-migration-down] Executing migration PropelMigration_1286483354 down
[propel-migration-down] 1 of 1 SQL statements executed successfully on datasource "bookstore"
[propel-migration-down] Reverse migration complete. No more migration available for reverse
```

Comme vous le supposez peut-être, la commande `migration:up` fait exactement l'opposé : elle exécute la migration suivante en _up_ :

```bash
$ propel migration:up

[propel-migration-up] Executing migration PropelMigration_1286483354 up
[propel-migration-up] 1 of 1 SQL statements executed successfully on datasource "bookstore"
[propel-migration-up] Migration complete. 1 migrations left to execute.
```

>**Astuce** Ce qui différencie les tâches `migration:up` et `migrate` est que la première exécute une migration seulement, tandis que `migrate` effectue toutes celles à sa  disposition.

### Les statuts de Migration ###

En suivant l'exemple précédent,  vous avez peut-être observé que le schéma et la base de données sont maintenant désynchronisés. En appelant deux fois la tâche `migration:down` et une fois la tâche `migration:up`, il reste une migration en attente d'exécution. Ce type de situation arrive parfois dans la vie d'un projet, lorsqu'on ne sait plus quelles migrations ont déjà été faites et lesquelles restent à faire.

Dans ce genre de situations, Propel permet d'utiliser la tâche `migration:status` afin de lister les migrations en attente. cela vous aidera à identifier où vous en êtes dans votre processus de migration.


```bash
$ propel migration:status

[propel-migration-status] Checking Database Versions...
[propel-migration-status] Listing Migration files...
[propel-migration-status] 1 migration needs to be executed:
[propel-migration-status]    PropelMigration_1286484196
[propel-migration-status] Call the "migrate" task to execute it
```

>**Astuce** Comme toutes les autres tâches proposées par Propel `migration:status` dispose d'un mode "verbeux" affichant plus de détails dans le CLI output. Ajouter  `--verbose` à la fin de la tâche pour activer ce mode, en prenant soin de rajouter le chemin vers le projet en premier argument.

```bash
$ propel . migration:status --verbose
[propel-migration-status] Checking Database Versions...
[propel-migration-status] Connecting to database "bookstore" using DSN "mysql:dbname=bookstore"
[propel-migration-status] Latest migration was executed on 2010-10-07 22:29:14 (timestamp 1286483354)
[propel-migration-status] Listing Migration files...
[propel-migration-status] 2 valid migration classes found in "/Users/francois/propel/1.6/test/fixtures/migration/build/migrations"
[propel-migration-status] 1 migration needs to be executed:
[propel-migration-status]  > PropelMigration_1286483354 (executed)
[propel-migration-status]    PropelMigration_1286484196
[propel-migration-status] Call the "migrate" task to execute it
```

`migration:up`, `migration:down`, et `migration:status` vous permettront de vous y retrouver dans les fichiers de migration, notamment lorsqu'ils deviendront nombreux ou lorsque vous aurez besoin d'en inverser plus d'un.

>**Astuce** There is no need to keep old migration files if you are sure that you won't ever need to revert to an old state. If a new developer needs to setup the project from scratch, the `sql:build` and `sql:insert` tasks will initialize the database structure to the current XML schema.

## How Do Migrations Work ? ##

The Propel `diff` task creates migration class names (like `PropelMigration_1286483354`) using the timestamp of the date they were created. Not only does it make the classes automatically sorted by date in a standard directory listing, it also avoids collision between two developers working on two structure changes at the same time.

Propel creates a special table in the database, where it keeps the date of the latest executed migration. That way, by comparing the available migrations and the date of the latest ones, Propel can determine the next migration to execute.

```
mysql> select * from propel_migration;
+------------+
| version    |
+------------+
| 1286483354 |
+------------+
1 row in set (0.00 sec)
```

So don't be surprised if your database show a `propel_migration` table that you never added to your schema - this is the Propel migration table. Propel doesn't use this table at runtime, and it never contains more than one line, so it should not bother you.

## Migration Configuration ##

The migration tasks support customization through a few settings from your configuration file:

```yaml
propel:
  migrations:
      # Whether to specify PHP names that are the same as the column names.
      samePhpName: false
      
      # Whether to add the vendor info. It does provide additional information (such as full-text indexes) which can
      # affect the generation of the DDL from the schema.
      addVendorInfo: false
      
      # The name of migrations table
      tableName: propel_migration
      
      # The name of the parser class
      # If you leave this property blank, Propel looks for an appropriate parser class, based on platform: i.e.
      # if the platform is `MysqlPlatform` then parser is `\Propel\Generator\Reverse\MysqlSchemaParser` 
      parserClass:
```

>**Tip**The `diff` task supports an additional parameter, called `editor`, which specifies a text editor to be automatically launched at the end of the task to review the generated migration. Unfortunately, only editors launched in another window are accepted. Mac users will find it useful, though:

```bash
$ propel diff --editor=mate
```

## Migrating Data ##

Propel generates the SQL code to alter the database structure, but your project may require more. For instance, in the newly added `author` table, the developer may want to add a few records.

That's why Propel automatically executes the `preUp()` and `postUp()` migration before and after the structure migration. If you want to add data migration, that's the place to put the related code.

Each of these methods receive a `PropelMigrationManager` instance, which is a good way to get PDO connection instances based on the buildtime configuration.

Here is an example implementation of data migration:

```php
<?php
class PropelMigration_1286483354
{
	// do nothing before structure change
	public function preUp($manager)
	{
	}

	// structure change (generated by Propel)
	public function getUpSQL()
	{
		return array('bookstore' => '
ALTER TABLE `book` ADD
(
	`author_id` INTEGER
);
//...
');
	}

	public function postUp($manager)
	{
		// post-migration code
		$sql = "INSERT INTO author (first_name,last_name) values('Leo','Tolstoi')";
		$pdo = $manager->getAdapterConnection('bookstore');
		$stmt = $pdo->prepare($sql);
		$stmt->execute();
	}
}
```

>**Tip**If you return `false` in the `preUp()` method, the migration is aborted.

You can also use Propel ActiveRecord and Query objects, but you'll then need to bootstrap the `Propel` class and the runtime autoloading in the migration class. This is because the Propel CLI does not know where the runtime classes are.

```php
<?php
// bootstrap the Propel runtime (and other dependencies)
require_once '/path/to/vendor/autoload.php';

set_include_path('/path/to/generated-classes' . PATH_SEPARATOR . get_include_path());
include '/path/to/generated-conf/config.php';

class PropelMigration_1286483354
{

	public function postUp($manager)
	{
		// add the post-migration code here
		$pdo = $manager->getAdapterConnection('bookstore');
		$author = new Author();
		$author->setFirstName('Leo');
		$author->setLastname('Tolstoi');
		$author->save($pdo);
	}

	public function getUpSQL()
	{
		// ...
	}
}
```

Of course, you can add code to the `preDown()` and `postDown()` methods to execute a data migration when reverting migrations.

---
<span class="next">[Next: Configuration &rarr;](10-configuration.html)</span>
