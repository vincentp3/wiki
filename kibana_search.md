# Comment chercher dans Kibana ?

Deux langagges sont disponibles : Kibana Query Language et Lucene (cad Apache Lucene utilisé en général dans ELK).
KQL est utilisé pour la rechercher dans les champs `nested` et les champs `scripted`.
Lucene permet de faire des recherches regex et fuzzing (ce que ne fait pas KQL).

Il existe également les Query DSL, qui permettent d'aller tapper directement sur l'API ELK.

## Kibana Query Language

### Recherche de termes exacts

```kql
http.response.status_code:400 401 404 // un de ces mots
http.response.body.content.text:quick //match exact
http.response.body.content.text:quick brown fox //un de ces mots
http.response.body.content.text:"quick brown fox" // exactement cette phrase
"quick brown fox" // recherche de la phrase dans tous les champs
```
### AND/OR/NOT

```kql
response:200 and (extension:php or extension:css) // équivalent à : extension:(php or css)
not response:200 
```

### Range

```kql
account_number >= 100 and items_sold <= 200
@timestamp < "2021-01-02T21:55:59"
@timestamp < "2021"
```

### Regex basiques

```kql
response:* // le champ response n'est pas null
machine.os:win* // l'os commence par 'win'
machine.os*:windows 10 // les champs commençant par 'machine.os' contiennent soit windows soit 10 ! (vérifier mais erreur sur la page elk....)
```

### Ajouts nestedfields

KQL permet de chercher à travers les nested fields (un type de liste d'objets : équivalent en écriture à une liste de dictionnaires avec les même clefs) : exemple : mettons que `items` soit un nested fields :

```kql
items:{ name:banana and stock > 10 } // items ayant pour nom bana et un stock > 10
items:{ name:banana } and items:{ stock:9 } // items au nom banana et items ayant un sock de 9
```

[Source pour KQL](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)

## Lucene

Lucene sert donc principalement pour les regex et pour le fuzzing.

### Recherches basiques

```lucene
banana // recherche de banana dans tous les champs
status:200 // recherche dans un field particulier
```

```lucene
title:brown // title contient brown
title : "brown" 
first\ name:"Alice" // first name strictement égal à brown (escape de l'espace !)
```

```lucene
book.\*:cyber // les champs du type book.* (book.title, book.source...etc) contenant cyber
sample_?:cyber //les champs du type sample_0, sample_a ...etc xontenant cyber
_exists_:title // où title n'a pas d evaleur nulle.
```

### Regex

Les regex sont embarqués entre slashs (`/` regex `/`).
Les caractères réservés sont : `. ? + * | { } [ ] ( ) " \`.

```lucene
name:/s.*/   // si le field name contient un nom commençant par s.
stock:/[1-9][0-9]+/ // stocks >= 10
```
Voir ici pour d'autres exemples : [lien](https://www.elastic.co/guide/en/elasticsearch/reference/7.15/regexp-syntax.html)

### Fuzzing

La méthode de fuzzing ici utilisée est la disatnce de  [Damerau-Levenshtein](https://fr.wikipedia.org/wiki/Distance_de_Damerau-Levenshtein). Le nombre d'opération maximum est de base à 2.
```lucene
type:safari~ // si le type est safari, safar, safara, zzfari zzsafari...
type:safari~1 // le nombre d'opération max est modifié à 1 : safar, safaro, safrai, ...
type:"bad operation"~ // matche operation bad, bad operation, very bad operation... (quand le ~ est sur une phrase entre `"`, alors cela joue sur les mots
```

### Les intervalles

```lucene
status:[400 TO 499] // recherche un range sur un field
status:[400 TO *] // range ouvert
status:[400 TO 500} // 400 à 500 exclu

```

### AND, OR, NOT


```lucene
status:[400 TO 499] AND NOT (extension:php OR extension:html)
```

### Ajouts

```lucene
search:fox^2 brown // le terme cherché fox est deux fois plus important que brown (ordre des résultats)
search: +malware threat open -telegram // je veux tous les champs rehcerche contenant malware obligatoirement avec optionellement threat ou open et sans telegram.
```

## Query DSL

[Source](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

### Script queries

[Source](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-script-query.html)

### Regexp queries

[Source](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-regexp-query.html)

