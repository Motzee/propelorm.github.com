---
layout: documentation
title: Documentation
---


# Documentation #

 * [Les nouveautés de Propel 2.0](whats-new.html) Utilisateurs des versions précédentes, découvrez ici les changements apportés par la version 2.
 * [Changelog](https://github.com/propelorm/Propel2/blob/master/UPDATE.md) Updates of the version 2.0.
 * [API Documentation](http://api.propelorm.org/) The generated API documentation.

## Project Setup ##

 * [InstallerPropel](01-installation.html) Installer Propel en utilisant Git, ou un zipball/tarball.
 * [Construire un projet](02-buildtime.html) Générer un modèle PHP basé sur un schéma XML.

## Les bases de Propel ##

* [Le CRUD de base](03-basic-crud.html) Les bases de Propel pour faire les opérations du C.R.U.D.(Create, Retrieve, Update, Delete)
* [Les relations](04-relationships.html) Rechercher et manipuler des données issues de tables relationnelles.
* [Les transactions](05-transactions.html) Quand et où utiliser les trnasactions
* [Behaviors](06-behaviors.html) Le système de comportement permet d'empaqueter et réutiliser des caractéristiques communes d'un modèle.
* [Enregistrement de logs et débuguage](07-logging.html) Propel peut garder des traces de ce qu'il a fait (logs), notamment les commandes SQL qu'il a éxécutées.
* [Inheritance](08-inheritance.html) Single Table Inheritance, Class Table Inheritance, and Concrete Table Inheritance come free with Propel.
* [Les migrations](09-migrations.html) Changer la structure de la base de données sans altérer les données.
* [Configuration](10-configuration.html) Configurer Propel

## Reference ##

* [XML Schema Format](/documentation/reference/schema.html) All the database, table, column and foreign key options explained
* [Active Record Classes](/documentation/reference/active-record.html) Complete list of the methods of Active Record classes.
* [Active Query Classes](/documentation/reference/model-criteria.html) Complete list of the methods of Propel Query classes.
* [Compatibility index](/documentation/reference/compatibility-index.html) A list of primary limitations regarding different databases.
* [Configuration file](/documentation/reference/configuration-file.html) Complete list of Propel configuration properties.


## Behaviors Reference ##

* [Writing A Behavior](/documentation/cookbook/writing-behavior.html) How to write a custom behavior to reuse model code horizontally.
* [aggregate_column](/documentation/behaviors/aggregate-column.html)
* [archivable](/documentation/behaviors/archivable.html)
* [auto_add_pk](/documentation/behaviors/auto-add-pk.html)
* [delegate](/documentation/behaviors/delegate.html)
* [i18n](/documentation/behaviors/i18n.html)
* [nested_set](/documentation/behaviors/nested-set.html)
* [query_cache](/documentation/behaviors/query-cache.html)
* [sluggable](/documentation/behaviors/sluggable.html)
* [timestampable](/documentation/behaviors/timestampable.html)
* [sortable](/documentation/behaviors/sortable.html)
* [validate](/documentation/behaviors/validate.html)
* [versionable](/documentation/behaviors/versionable.html)
* And [concrete_inheritance](08-inheritance.html), documented in the Inheritance Chapter even if it's a behavior

You can also look at [user contributed behaviors](../documentation/cookbook/user-contributed-behaviors.html).

## Cookbook ##

### Common Tasks ###

* [Additional SQL Files](/documentation/cookbook/adding-additional-sql-files.html) How to execute custom SQL statements at buildtime
* [Advanced Column Types](/documentation/cookbook/working-with-advanced-column-types.html) How to work with BLOBs, serialized PHP objects, ENUM, and ARRAY column types.
* [How to Use Namespaces](/documentation/cookbook/namespaces.html) How to generate model classes with namespaces, and how to use them.
* [Model Introspection At Runtime](/documentation/cookbook/runtime-introspection.html) How to use the Map classes to discover table properties at runtime.
* [Multi-Component Data Model](/documentation/cookbook/multi-component-data-model.html) How to generate model classes in subdirectories, and organize your model into independent packages / modules.
* [Object Copy](/documentation/cookbook/copying-persisted-objects.html) How to clone and copy persisted objects.
* [Replication](/documentation/cookbook/replication.html) How to use Propel in a Master-Slave Replication Environment.
* [Using Propel With MSSQL Server](/documentation/cookbook/using-mssql-server.html) How to choose and configure Propel to persist data to a Microsoft SQL Server database.
* [Using SQL Schemas](/documentation/cookbook/using-sql-schemas.html) How to organize tables into SQL schemas (only for MySQL, PostgreSQL, and MSSQL).
* [Working With Existing Databases](/documentation/cookbook/working-with-existing-databases.html) How to build an XML schema from an existing db structure, how to dump data to XML, how to import it into a new database, etc.

### Contribuer à Propel ###

* [Writing A Behavior](/documentation/cookbook/writing-behavior.html) How to write a custom behavior to reuse model code horizontally.
* [Working with Propel's Test Suite](/documentation/cookbook/working-with-test-suite.html) How to work with Propel's test suite if you want to add a missing case or attach a regression test with your patch.

### Travailler avec Silex ###

* [Travailler avec Silex](/documentation/cookbook/silex/working-with-silex.html)

### Travailler avec Symfony2 ###

* [Travailler avec Symfony2 (Introduction)](/documentation/cookbook/symfony2/working-with-symfony2.html)
* [Maîtriser les formulaires sous Symfony2 grâce à Propel](/documentation/cookbook/symfony2/mastering-symfony2-forms-with-propel.html)
* [Le composant de sécurité Symfony2 et Propel](/documentation/cookbook/symfony2/the-symfony2-security-component-and-propel.html)

>**Astuce** Ceci est la documentation à jour pour la dernière version de Propel.
> Pour accéder à l'ancienne documentation, vous pouvez consulter :
[trac.propelorm.org](http://trac.propelorm.org) ou
[propelorm.org/Propel](http://propelorm.org/Propel/).
