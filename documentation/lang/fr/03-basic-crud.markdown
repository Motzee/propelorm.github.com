---
layout: documentation
title: Opérations basiques C.R.U.D.
---

# Opérations basiques C.R.U.D. #

Dans ce chapitre, vous apprendrez comment effectuer les opérations basique du C.R.U.D (Create, Retrieve, Update, Delete) sur votre base de données via Propel.

## Ajouter des entrées (Create) ##

Pour ajouter des données dans la base de données, instancier un objet grâce à Propel, puis appeler sa méthode `save()`. Propel exécutera alors la consigne SQL INSERT appropriée pour cet objet.

Evidemment, avant de sauvegarder cet objet vous désirerez certainement l'hydrater pour attribuer des valeurs à ses différentes colonnes... Propel génère à cet effet des méthodes `setXXX()` pour chacune des colonnes de la table, méthodes définies dans le modèle de l'objet. Insérer une nouvelle entrée dans une table se fait simplement ainsi :

```php
<?php
/* initialize Propel, etc. */

$author = new Author();
$author->setFirstName('Jane');
$author->setLastName('Austen');
$author->save();
```

Les noms de colonnes utilisés dans les méthodes `setXXX()` correspondent aux attributs `phpName` définis dans les balises `<column>` du schéma de la base de données, ou bien au nom de la colonne en écriture CamelCase si cet attribut `phpName` n'est pas défini.

L'utilisation de la méthode `save()` entrainera l'éxécution de la consigne SQL suivantes sur la base de données :

```sql
INSERT INTO author (first_name, last_name) VALUES ('Jane', 'Austen');
```

## Consulter les propriétés d'un objet ##

Propel convertit les colonnes d'une table en propriétés pour l'objet généré. Pour chacune de ces propriétés, vous pouvez utiliser la méthode `getXXX()` générée pour en récupérer la valeur :

```php
<?php
echo $author->getId();        // 1
echo $author->getFirstName(); // 'Jane'
echo $author->getLastName();  // 'Austen'
```

La colonne `id` est automatiquement complétée par la base de données grâce à la définition de l'`autoIncrement` sur cette colonne dans le `schema.xml`. Une fois l'objet enregistré, il est aisé de récupérer cet identifiant simplement en utilisant son getter.

Ces appels de méthodes ne provoquent pas de nouvelle requête dans la base de données, car l'objet `Author` et ses propriétés est déjà chargé en mémoire.

Vous pouvez aussi exporter ces propriétés d'objet vers d'autres formats de données grâce aux méthodes suivantes :  `toArray()`, `toXML()`, `toYAML()`, `toJSON()`, `toCSV()`, et `__toString()`:

```php
<?php
echo $author->toJSON();
// {"Id":1,"FirstName":"Jane","LastName":"Austen"}
```

>**Astuce** Pour chaque méthode d'export, Propel propose un import équivalent, ce qui vous permettra d'hydrater facilement un objet grâce à un tableau en utilisant la méthode `fromArray()`, ou bien grâce à une chaîne de caractères en utilisant les méthodes `fromXML()`, `fromYAML()`, `fromJSON()`, ou `fromCSV()`.

De nombreuses méthodes tout aussi utiles vous sont offertes pour chaque objet généré. Une liste détaillée de ces méthodes est disponible dans la partie [Active Record reference](/documentation/reference/active-record.html).

## Lire des entrées (Read) ##

Récupérer des objets issus de la base de données consiste essentiellement à faire exécuter une requête SELECT dans la base de données, permettant ainsi de créer pour chaque entrée une nouvelle instance de l'objet équivalant _hydraté_ avec les valeurs de cette ligne.

Sous Propel, il faudra passer par des classes Query générées pour chaque modèle d'objet afin de récupérer dans la base de données ces entrées attendues.

### Récupération via une clef primaire (Primary key) ###

La manière la plus simple de récupérer une entrée de la base de données est d'utiliser la méthode `findPK()` générée. Cette méthode attend simplement en paramètre la valeur de la clef promaire de l'entrée à retrouver.

```php
<?php
$q = new AuthorQuery();
$firstAuthor = $q->findPK(1);
// $firstAuthor est maintenant un objet sur le modèle d'Author, ou bien vaudra NULL si aucune correspondance n'ets trouvée.
```

Ce code exécute simplement une requête SQL de type SELECT, qui serait par exemple traduit pour MySQL en :

```sql
SELECT author.id, author.first_name, author.last_name
FROM `author`
WHERE author.id = 1
LIMIT 1;
```

Dans le cas où la clef primaire consisterait en plus d'une colonne, `findPK()` acceptera alors plusieurs paramètres : un pour chaque colonne portant une clef primaire.

>**Astuce** Chaque objet Query généré dispose d'une factory méthode `create()` créant une nouvelle instance de la requête, sur laquelle on pourra enchaîner sur une même ligne les méthodes permettant de compléter cette requête :

>```php
><?php
>$firstAuthor = AuthorQuery::create()->findPK(1);
>```

Il est possible de sélectionner plusieurs objets à la fois grâce à leur clef primaire en utilisant plutôt la méthode générée `findPKs()`, qui prend un tableau de clefs primires en paramètre :

```php
<?php
$selectedAuthors = AuthorQuery::create()->findPKs(array(1,2,3,4,5,6,7));
// $selectedAuthors est une collection d'objets sur le modèle Author
```

### Interroger la base de données ###

En dehors de l'usage des clefs primaires, il est possible de récupérer des entrées dans la base de données en utilisant la méthode `find()` d'un objet Query.

Sans paramètre indiqué, cette méthode renverra toutes les entrées de la table.

```php
<?php
$authors = AuthorQuery::create()->find();
// $authors contient une collection d'objets instances d'Author
// chaque objet représente une entrée dans la table author
foreach($authors as $author) {
  echo $author->getFirstName();
}
```

Pour ajouter un critère de recherche concernant une colonne donnée, utiliser les méthodes `filterByXXX()` générées pour l'objet Query, où `XXX` est le phpName d'une colonne. `filterByXXX()` retournant la requête d'objet écrite, il est possible d'ajouter des critères de recherche si besoin, avant de terminer et éxécuter cette requête avec `find()`. On pourra par exemple filtrer les auteurs portant le prénom Jane ainsi :

```php
<?php
$authors = AuthorQuery::create()
  ->filterByFirstName('Jane')
  ->find();
```

Les valeurs passées aux méthodes `filterByXXX()` sont systématiquement transformées en requêtes préparées via PDO, votre table est donc protégée des risques d'injection SQL.

>**Astuce** `filterByXXX()` est la méthode à préférer pour créer des requêtes. Elle est flexible, et accepte en valeur autant des caractères génériques que des tableaux à des usages plus complexes. Consulter les [Méthodes de filtre de colonne](/documentation/reference/model-criteria.html#column_filter_methods) pour de plus amples détails.

Vous pouvez facilement limiter ou ordonner les résultats d'une requête. Les méthodes de l'objet Query retournant la requête en cours de rédaction, il est facile de les enchainer ainsi :

```php
<?php
$authors = AuthorQuery::create()
  ->orderByLastName()
  ->limit(10)
  ->find();
```

`find()` retourne systématiquement une collection d'objets, même s'il n'y a qu'un seul résultat obtenu. Ainsi, si vous savez que vous n'obtiendrez qu'un seul résultat, utilisez plutôt la méthode `findOne()`. Cela limitera la requête à un résultat et renverra un seul objet au lieu d'un tableau :

```php
<?php
$author = AuthorQuery::create()
  ->filterByFirstName('Jane')
  ->findOne();
```

>**Tip**Propel provides magic methods for this simple use case. So you can write the above query as:

```php
<?php
$author = AuthorQuery::create()->findOneByFirstName('Jane');
```

The Propel Query API is very powerful. The next chapter will teach you to use it to add conditions on related objects. If you can't wait, jump to the [Query API reference](/documentation/reference/model-criteria.html).

### Using Custom SQL ###

The `Query` class provides a relatively simple approach to constructing a query. Its database neutrality and logical simplicity make it a good choice for expressing many common queries. However, for a very complex query, it may prove more effective (and less painful) to simply use a custom SQL query to hydrate your Propel objects.

As Propel uses PDO to query the underlying database, you can always write custom queries using the PDO syntax. For instance, if you have to use a sub-select:

```php
<?php
use Propel\Runtime\Propel;
$con = Propel::getWriteConnection(\Map\BookTableMap::DATABASE_NAME);
$sql = "SELECT * FROM book WHERE id NOT IN "
        ."(SELECT book_review.book_id FROM book_review"
        ." INNER JOIN author ON (book_review.author_id=author.ID)"
        ." WHERE author.last_name = :name)";
$stmt = $con->prepare($sql);
$stmt->execute(array(':name' => 'Austen'));
```

With only a little bit more work, you can also populate `Book` objects from the resulting statement. Create a new `ObjectFormatter` for the `Book` model, and call the `format()` method using the `DataFetcher` instance of the current connection with the pdo statement:

```php
<?php
use Propel\Runtime\Formatter\ObjectFormatter;
$con = Propel::getWriteConnection(\Map\BookTableMap::DATABASE_NAME);
$formatter = new ObjectFormatter();
$formatter->setClass('\Book'); //full qualified class name
$books = $formatter->format($con->getDataFetcher($stmt));
// $books contains a collection of Book objects
```

There are a few important things to remember when using custom SQL to populate Propel:

* The resultset columns must be numerically indexed
* The resultset must contain all the columns of the table (except lazy-load columns)
* The resultset must have columns _in the same order_ as they are defined in the `schema.xml` file

## Updating Objects ##

Updating database rows basically involves retrieving objects, modifying the contents, and then saving them. In practice, for Propel, this is a combination of what you've already seen in the previous sections:

```php
<?php
$author = AuthorQuery::create()->findOneByFirstName('Jane');
$author->setLastName('Austen');
$author->save();
```

Alternatively, you can update several rows based on a Query using the query object's `update()` method:

```php
<?php
AuthorQuery::create()
  ->filterByFirstName('Jane')
  ->update(array('LastName' => 'Austen'));
```

This last method is better for updating several rows at once, or if you didn't retrieve the objects before.

## Deleting Objects ##

Deleting objects works the same as updating them. You can either delete an existing object:

```php
<?php
$author = AuthorQuery::create()->findOneByFirstName('Jane');
$author->delete();
```

Or use the `delete()` method in the query:

```php
<?php
AuthorQuery::create()
  ->filterByFirstName('Jane')
  ->delete();
```

>**Tip**A deleted object still lives in the PHP code. It is marked as deleted and cannot be saved anymore, but you can still read its properties:

```php
<?php
echo $author->isDeleted();    // true
echo $author->getFirstName(); // 'Jane'
```

## Query Termination Methods ##

The Query methods that don't return the current query object are called "Termination Methods". You've already seen some of them: `find()`, `findOne()`, `update()`, `delete()`. There are two more termination methods that you should know about:

`count()` returns the number of results of the query.

```php
<?php
$nbAuthors = AuthorQuery::create()->count();
```
You could also count the number of results from a `find()`, but that would be less effective, since it implies hydrating objects just to count them.

`paginate()` returns a paginated list of results:

```php
<?php
$authorPager = AuthorQuery::create()->paginate($page = 1, $maxPerPage = 10);
// This method will compute an offset and a limit
// based on the number of the page and the max number of results per page.
// The result is a PropelModelPager object, over which you can iterate:
foreach ($authorPager as $author) {
  echo $author->getFirstName();
}
```

A pager object gives more information:

```php
<?php
echo $pager->getNbResults();   // total number of results if not paginated
echo $pager->haveToPaginate(); // return true if the total number of results exceeds the maximum per page
echo $pager->getFirstIndex();  // index of the first result in the page
echo $pager->getLastIndex();   // index of the last result in the page
$links = $pager->getLinks(5);  // array of page numbers around the current page; useful to display pagination controls
```

## Collections And On-Demand Hydration ##

The `find()` method of generated Model Query objects returns a `PropelCollection` object. You can use this object just like an array of model objects, iterate over it using `foreach`, access the objects by key, etc.

```php
<?php
$authors = AuthorQuery::create()
  ->limit(5)
  ->find();
foreach ($authors as $author) {
  echo $author->getFirstName();
}
```

The advantage of using a collection instead of an array is that Propel can hydrate model objects on demand. Using this feature, you'll never fall short of memory when retrieving a large number of results. Available through the `setFormatter()` method of Model Queries, on-demand hydration is very easy to trigger:

```php
<?php
$authors = AuthorQuery::create()
  ->limit(50000)
  ->setFormatter(ModelCriteria::FORMAT_ON_DEMAND) // just add this line
  ->find();
foreach ($authors as $author) {
  echo $author->getFirstName();
}
```

In this example, Propel will hydrate the `Author` objects row by row, after the `foreach` call, and reuse the memory between each iteration. The consequence is that the above code won't use more memory when the query returns 50,000 results than when it returns 5.

`ModelCriteria::FORMAT_ON_DEMAND` is one of the many formatters provided by the Query objects. You can also get a collection of associative arrays instead of objects, if you don't need any of the logic stored in your model object, by using `ModelCriteria::FORMAT_ARRAY`.

The [ModelCriteria Query API reference](/documentation/reference/model-criteria.html) describes each formatter, and how to use it.

## Propel Instance Pool ##

Propel keeps a list of the objects that you already retrieved in memory to avoid calling the same request twice in a PHP script. This list is called the instance pool, and is automatically populated from your past requests:

```php
<?php
// first call
$author1 = AuthorQuery::create()->findPk(1);
// Issues a SELECT query
...
// second call
$author2 = AuthorQuery::create()->findPk(1);
// Skips the SQL query and returns the existing $author1 object
```

---
<span class="next">[Next: Les relations &rarr;](04-relationships.html)</span>
