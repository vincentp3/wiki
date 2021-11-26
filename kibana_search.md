# Comment chercher dans Kibana ?

Deux langagges sont disponibles : Kibana Query Language et Lucene.
KQL est utilisé pour la rechercher dans les champs `nested` et les champs `scripted`.
Lucene permet de faire des recherches regex et fuzzing (ce que ne fait pas KQL).

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

[Source pour KQL](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)


























