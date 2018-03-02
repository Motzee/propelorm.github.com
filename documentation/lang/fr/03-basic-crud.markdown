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

>**Astuce** Pour ce cas simple, Propel dispose d'une  méthode magique. Vous pouvez ainsi convertir l'exemple ci-dessus en :

```php
<?php
$author = AuthorQuery::create()->findOneByFirstName('Jane');
```

L'API de requêtage de Propel est particulièrement complète. Dans le chapitre suivant, vous apprendrez comment l'utiliser pour ajouter des conditions on related objects. Pour vous y rendre directement, consultez [Query API reference](/documentation/reference/model-criteria.html).

### Utiliser des requêtes SQL maison ###

La classe `Query` permet de construire une requête de manière simple. Sa neutralité et sa simplicité logique en font un bon choix pour les requêtes les plus courantes. Cependant, pour des requêtes particulièrement complexes, il pourra être plus efficace (et moins difficile) d'utiliser une requête SQL personnalisée pour hydrater vos objets Propel.

Propel utilisant PDO pour interroger la base de données, vous pouvez contniuer d'utiliser des requêtes écrites en syntaxe PDO. Par exemple, si vous devez imbriquer des SELECTs :

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

Avec un peu plus d'efforts seulement, vous pourrez aussi populate `Book` objects from the resulting statement. 

Pour cela, créer un nouvel `ObjectFormatter` pour le modèle de classe `Book`, et appeler la méthode `format()` en passant en paramètre l'instance `DataFetcher` de la connexion à la base de données :

```php
<?php
use Propel\Runtime\Formatter\ObjectFormatter;
$con = Propel::getWriteConnection(\Map\BookTableMap::DATABASE_NAME);
$formatter = new ObjectFormatter();
$formatter->setClass('\Book'); //full qualified class name
$books = $formatter->format($con->getDataFetcher($stmt));
// $books contient une collection d'objets sur le modèle Book
```

Pour utiliser des requêtes SQL maison avec Propel, il faut garder à l'esprit les choses suivantes :
* The resultset columns must be numerically indexed
* The resultset must contain all the columns of the table (except lazy-load columns)
* The resultset must have columns _in the same order_ as they are defined in the `schema.xml` file

## Modifier des objets (Update) ##

Modifier des entrées consiste simplement à en récupérer les objets pour en modifier les propriétés avant de les sauvegarder. Sous propel, cette suite d'opérations est une combinaison de ce qui a déjà été évoqué jusqu'ici :

```php
<?php
$author = AuthorQuery::create()->findOneByFirstName('Jane');
$author->setLastName('Austen');
$author->save();
```

Si votre but est de modifier plusieurs entrée de manière similaire, vous pourrez utiliser comme alternative la méthode `update()` disponible pour les objets Query :

```php
<?php
AuthorQuery::create()
  ->filterByFirstName('Jane')
  ->update(array('LastName' => 'Austen'));
```

Cette méthode est à privilégier pour éditer plusieurs entrées à la fois ou bien si les objets n'avaient pas été récupérés avant.

## Supprimer des objets (Delete) ##

Supprimer des objets se fait de la même méthode que l'édition. Ainsi, pour supprimer un objet existant, faites :

```php
<?php
$author = AuthorQuery::create()->findOneByFirstName('Jane');
$author->delete();
```

Cette méthode `delete()` peut aussi être utilisée directement dans la requête :

```php
<?php
AuthorQuery::create()
  ->filterByFirstName('Jane')
  ->delete();
```

>**Astuce** Un objet supprimé continue d'exister dans le code PHP. Il est identifié comme supprimé et ne peut donc plus être sauvegardé, mais il reste possible d'en consuter les propriétés :

```php
<?php
echo $author->isDeleted();    // true
echo $author->getFirstName(); // 'Jane'
```

## Les méthodes de terminaison de requête ##

Les méthodes des objets Query ne renvoyant pas de requête en cours de construction sont appelées des "méthodes de terminaison" ("Termination Methods" en V.O.). Vous avez déjà croisé certaines d'entre elles : `find()`, `findOne()`, `update()`, `delete()`. Deux méthodes de terminaison supplémentaires sont à connaître :

`count()` renvoit le nombre de résultats d'une requête.

```php
<?php
$nbAuthors = AuthorQuery::create()->count();
```
Il serait possible de compter le nombre de résultats depuis un `find()` obtenu, mais cela serait moins performant puisque cela nécessiterait une hydratation des objets inutile pour ce comptage.

`paginate()` renvoie une liste paginée de résultats :

```php
<?php
$authorPager = AuthorQuery::create()->paginate($page = 1, $maxPerPage = 10);
// Cette méthode prend en paramètres un repère de début et une limite
// qui corespondent respectivement au numéro de page et au nombre maximal de résultats attendus par page.
// Le résultat obtenu sera un objet itérable de modèle PropelModelPager :
foreach ($authorPager as $author) {
  echo $author->getFirstName();
}
```

Un objet modèle PropelModelPager dispose de méthodes spécifiques fournissant plus d'informations :

```php
<?php
echo $pager->getNbResults();   // nombre total de résultats sans tenir compte de la pagination
echo $pager->haveToPaginate(); // renvoie true si le nombre précédent est supérieur au nombre de résultats maximal attendu par page
echo $pager->getFirstIndex();  // index du premier résultat de la page
echo $pager->getLastIndex();   // index du dernier résultat de la page
$links = $pager->getLinks(5);  // tableau des numéros de page autour du numéro de page actuel, bien utile pour l'affichage de la pagination
```

## Les collections et l'hydratation sur demande ##

La méthode `find()` des objets sur le modèle Query renvoie un objet itérable de type `PropelCollection`, utilisable comme un tableau d'objets : vous pouvez utiliser une boucle `foreach` pour accéder aux objets contenus, et exploiter leurs méthodes :

```php
<?php
$authors = AuthorQuery::create()
  ->limit(5)
  ->find();
foreach ($authors as $author) {
  echo $author->getFirstName();
}
```

L'utilisation de collections plutôt que de tableaux classiques permet à Propel d'hydrater les modèles d'objet sur demande. En exploitant cette fonctionnalité, vous ne serez jamais plus à court de mémoire lors d'une récupération massive de résultats. Utilisable via la méthode `setFormatter()` du modèle Query, l'hydratation sur demande se met en place assez simplement :

```php
<?php
$authors = AuthorQuery::create()
  ->limit(50000)
  ->setFormatter(ModelCriteria::FORMAT_ON_DEMAND) // ajouter simplement cette ligne dans la requête
  ->find();
foreach ($authors as $author) {
  echo $author->getFirstName();
}
```

Dans l'exemple ci-dessus, Propel hydratera les objets `Author` entrées par entrée _dans_ la boucle `foreach`, réuitlisant la mémoire à chaque itération. Ainsi, le code ci-dessus n'utilisera pas plus de mémoire pour récupérer 50 000 résultats qu'il n'en utilise pour 5.

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
