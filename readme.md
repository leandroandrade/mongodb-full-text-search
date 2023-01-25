# mongodb-full-text-search

## TL;DR
> Ideal para propósitos gerais, cenários em que features de pesquisa mais robustas encontradas em search engine específicas, como **ElasticSearch**, não são necessárias.


Uma collection pode ter somente **um** índice `text search`, mas o índice pode comtemplar mais de um campo da collection.

## Features

```js
db.getCollection('movies').insertMany([
    { title: 'Trek Nation' },
    { title: 'Star Trek' },
    { title: 'Star Trek Into Darkness' },
    { title: 'Star Trek: Nemesis' },
    { title: 'Star Trek: Insurrection' },
    { title: 'Star Trek: Generations' },
    { title: 'Star Trek: First Contact' },
    { title: 'Star Trek: The Motion Picture' },
    { title: 'Star Trek VI: The Undiscovered Country' },
    { title: 'Star Trek V: The Final Frontier' },
    { title: 'Star Trek IV: The Voyage Home' },
    { title: 'Star Trek III: The Search for Spock' },
    { title: 'Star Trek II: The Wrath of Khan' }
])

db.movies.createIndex({ title: "text" });
```

Pequisar por uma palavra específica:
```js
db.movies.find({ $text: { $search: "trek" } })
```

Pesquisar por uma frase:
```js
db.movies.find({ $text: { $search: "\"star trek\"" } })
```

Pequisar por uma frase omitindo um trecho (negotiation term: use `-`):
```js
db.movies.find({ $text: { $search: "\"star trek\" -\"into darkness\"" } })
```
Ordenar resultados por relevância com `textScore` e `$meta`. Score alto, mais exatidão. Diminui score a medida que a string aumenta:
```js
db.movies
    .find({ $text: { $search: "\"star trek\" -\"into darkness\"" } })
    .sort({ score: { $meta: "textScore" } })
    .projection({ _id: 0, title: 1, score: { $meta: "textScore" } })
// OBS.: To use a $text query in an $or expression, all clauses in the $or array must be indexed.
```

## Use Cases

A pesquisa funciona muito bem com palavras separadas. Exemplo:
```js
db.heroes.insertMany([
    { name: "Spider Man" },
    { name: "Super Man" },
    { name: "Superman" },
]);

db.heroes.createIndex({ name: "text" });
```

```js
db.heroes.find({
    $text: { $search: "man" }
})
```

Tanto ***Spider Man*** quando ***Super Man*** são localizados, mas ***Superman*** não é retornado.

Outro exemplo:
```js
db.coffees.insertMany([
    { description: "Great Coffee Drinks (with Recipes!)" },
    { description: "Coffee Shop Drinks and Treats You Can Make at Home" },
    { description: "Sample description with only coffee word" },
    { description: "Sample description with only recipe word" }
]);

db.coffees.createIndex({ description: "text" });

/**
 *  Com duas ou mais palavras, o MongoDB executa o operator lógico `or`
 */
db.coffees.find({
    $text: { $search: "coffee recipe" }
})
```

Todos os registros são retornados já que as palavras pesquisadas estão separadas.

## Portuguese

```js
db.movies.insertMany([
    { title: 'Atração Fatal' },
    { title: 'Cão de Briga' },
    { title: 'Os Caçadores' },
])

db.movies.createIndex(
    { title: 'text' },
    { default_language: "portuguese" }
)

db.movies.find({
    $text: { $search: "cacador" }
})
```

## Referências:
* https://www.mongodb.com/docs/drivers/node/current/fundamentals/crud/read-operations/text/
* https://www.mongodb.com/docs/v4.2/reference/operator/query/text/#text-query-operator-behavior
* https://www.digitalocean.com/community/tutorials/how-to-perform-full-text-search-in-mongodb
* https://www.mongodb.com/docs/manual/reference/text-search-languages/
* https://www.mongodb.com/docs/manual/reference/text-search-languages/
* https://www.mongodb.com/docs/manual/tutorial/specify-language-for-text-index/
* https://www.mongodb.com/docs/manual/core/index-text/#std-label-index-feature-text


