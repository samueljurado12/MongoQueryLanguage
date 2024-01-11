# MongoQueryLanguage

 Repositorio para el segundo laboratorio del Bootcamp Backend de Lemoncode

## General

En este base de datos puedes encontrar un montón de apartamentos y sus reviews, esto está sacado de hacer
webscrapping.

**Pregunta:** Si montaras un sitio real, ¿qué posibles problemas pontenciales le ves a cómo está almacenada la
información?

> El principal problema de la base de datos es que todos los datos están en una única colección. Esto no sería un problema si no fuese porque, además de los apartamentes, todas las reviews se almacenan en el mismo documento.
Esto haría que a la larga, conforme se recibieran más reviews, el tamaño de los documentos pueda exceder el límite o ser demasiado grandes como para que sean manejables en el Working Set

## Consultas

### Básico

- Saca en una consulta cuántos apartamentos hay en España.

```js
db.listingsAndReviews.countDocuments(
    {'address.country': 'Spain'},
)
```

- Lista los 10 primeros:
  - Solo muestra: nombre, camas, precio, government_area
  - Ordenados por precio

```js
db.listingsAndReviews.find(
    {},
    {
          _id:0,
          name: 1,
          beds: 1,
          government_area: '$address.government_area'
    })
    .limit(10)
    .sort({price:1})

```

### Filtrando

- Queremos viajar cómodos, somos 4 personas y queremos:
  - 4 camas.
  - Dos cuartos de baño.
  
```js
db.listingsAndReviews.find(
    {
        beds: 4,
        bathrooms: 2
    }
)
```

- Al requisito anterior, hay que añadir que nos gusta la tecnología queremos que el apartamento tenga wifi.

```js
db.listingsAndReviews.find(
    {
        beds: 4,
        bathrooms: 2,
        amenities: {
            $all: ['Wifi']
        }
    }
)
```

- Y bueno, un amigo se ha unido que trae un perro, así que a la query anterior tenemos que buscar que permitan mascota Pets Allowed

```js
db.listingsAndReviews.find(
    {
        beds: 4,
        bathrooms: 2,
        amenities: {
            $all: ['Wifi', 'Pets allowed']
        }
    }
)
```

### Operadores Lógicos

- Estamos entre ir a Barcelona o a Portugal, los dos destinos nos valen, peeero... queremos que el precio nos salga baratito (50 $), y que tenga buen rating de reviews

```js
db.listingsAndReviews.find(
    {
        price: 50,
        $or: [
            {'address.market': 'Barcelona'},
            {'address.country': 'Portugal'}
        ],
        'review_scores.review_scores_rating': {$gte: 80}
    }
)
```

## Agregaciones

- Queremos mostrar los pisos que hay en España, y los siguientes campos:
  - Nombre,
  - De que ciudad (no queremos mostrar un objeto, solo el string con la ciudad)
  - El precio

```js
db.listingsAndReviews.aggregate([
    {
      $match:
        {
          "address.country": "Spain",
        },
    },
    {
      $project:
        {
          _id: 0,
          name: 1,
          city: "$address.market",
          price: 1,
        },
    },
])
```

- Queremos saber cuántos alojamientos hay disponibles por país.

```js
db.listingsAndReviews.aggregate([
    {
        $group: {
          _id: "$address.country",
          total: {
            $sum: 1
          }
        }
    }
])
```

## Opcional

- Queremos saber el precio medio de alquiler de airbnb en España

```js
db.listingsAndReviews.aggregate([
    {
        $match: {
            "address.country": "Spain"
        }
    },
    {
        $group: {
            _id: "$address.country",
            precio_medio: {$avg: "$price"}
        }
    }
])
```

- ¿Y si quisiéramos hacer como el anterior, pero sacarlo por paises?

```js
db.listingsAndReviews.aggregate([
    {
        $group: {
            _id: "$address.country",
            precio_medio: {$avg: "$price"}
        }
    }
])
```

- Repite los mismos pasos pero agrupando también por número de habitaciones.

```js
db.listingsAndReviews.aggregate([
    {
        $match: {
            "address.country": "Spain"
        }
    },
    {
        $group: {
            _id: {
                country:"$address.country",
                bedrooms:"$bedrooms"
            },
            precio_medio: {$avg: "$price"}
        }
    }
])
```

```js
db.listingsAndReviews.aggregate([
    {
        $group: {
            _id: {
                country:"$address.country",
                bedrooms:"$bedrooms"
            },
            precio_medio: {$avg: "$price"}
        }
    }
])
```

## Desafio

Queremos mostrar el top 5 de apartamentos más caros en España, y sacar los siguentes campos:

- Nombre.
- Ciudad.
- Amenities, pero en vez de un array, un string con todos los amenities.

```js
db.listingsAndReviews.aggregate([
    {
      $match: {
        "address.country": "Spain",
      },
    },
    {
      $sort: {
        price: -1,
      },
    },
    {
      $limit: 5,
    },
    {
      $project:
        {
          _id: 0,
          name: 1,
          city: "$address.market",
          amenities: {
            $reduce: {
              input: "$amenities",
              initialValue: "",
              in: {
                $cond: [
                    {$eq: ["$$value", ""]},
                    {
                        $concat: [
                              "$$value",
                              "$$this",
                            ],
                    },
                    {
                        $concat: [
                              "$$value",
                              ", ",
                              "$$this",
                            ],
                    }
                ]
              },
            },
          },
        },
    },
  ])
```
