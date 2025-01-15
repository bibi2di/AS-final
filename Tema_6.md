## Elasticsearch

Pila ELK: Elasticsearch, Logstash, Kibana.

Uso de la api rest: curl o Postman
Verificación:
```
curl -XGET <ip>:9200
```
### Conceptos
* Documentos: elementos que se buscan en la BBDD. Cada uno tiene un ID único y un tipo.
* Índices: permiten buscar documentos de un tipo determinado. Contienen el esquema de los datos. Se usan índices invertidos.
* Mapping: esquema para un índice. 
Ejemplo:
```
{
"mappings" : {
  "properties" : {
    "titulo" : {"type": "text" },
    "autor" : {"type": "text" },
    "isbn" : {"type": "integer" }
    }
  }
}
```
* Shard: unidad en la que se distribuyen datos en el cluster

### Operaciones CRUD:
Crear un índice con un mapping:
```
curl -H "Content-Type: application/json" -XPUT
<IP-servidor>:<puerto>/<nombre-indice> <definición-mapping>
```
El mapping se puede indicar con un fichero adjunto --data-binary@<fichero> o escribirlo como parte del comando: -d '<mapping>'

Carga de un documento:
```
curl -H "Content-Type: application/json" -XPOST
<IP>:<puerto>/<índice>/_doc/<ID-documento> --data-binary @<fichero>
```

Recuperar índices:
```
curl 127.0.0.1:9200/_cat/indices?v
```
Mapping de índice:
```
curl -XGET 127.0.0.1:9200/libros/_mapping
```
Número de documentos de índice:
```
curl -XGET 127.0.0.1:9200/libros/_count?pretty
```
Documentos de un índice:
```
curl -XGET 127.0.0.1:9200/libros/_search?pretty
```
Carga de múltiples elementos:
```
curl -H "Content-Type: application/json" -XPUT '127.0.0.1:9200/libros/_bulk'
--data-binary @muchos-libros.json
```
Modificar documentos (cambiar versión y borrar anterior):
```
curl -H "Content-Type: application/json" -XPOST
127.0.0.1:9200/libros/_update/11 --data-binary @modificar-libro.json
```
Borrar documento:
```
curl -XDELETE <IP>:<puerto>/<índice>/_doc/<ID>
```
Comandos con ejemplos: https://documenter.getpostman.com/view/5283544/S1LyUnSc

### Ejercicio 2:
* Crear un índice "películas" con mapping: "titulo" -> text, "director" -> text, "año" -> integer:
```
curl -X PUT "localhost:9200/peliculas" -H 'Content-Type: application/json' -d'
{
  "mappings": {
    "properties": {
      "titulo": {
        "type": "text"
      },
      "director": {
        "type": "text"
      },
      "año": {
        "type": "integer"
      }
    }
  }
}'
```
* Cargar los datos usando carga en bruto:
```
{"create" : {"_index": "peliculas", "_id" : "1" } }
{"id" : "1", "titulo" : "El padrino", "director" : "Francis Ford Coppola", "año": 1972}
{"create" : {"_index": "peliculas", "_id" : "2" } }
{"id" : "2", "titulo" : "Gladiator", "director" : "Ridley Scott", "año": 2030}
{"create" : {"_index": "peliculas", "_id" : "3" } }
{"id" : "3", "titulo" : "Inception", "director" : "Christopher Nolan", "año": 2010}
```
Resolución:
```
curl -X POST "http://localhost:9200/peliculas/_doc/1" -H 'Content-Type: application/json' -d'
{
  "id": "1",
  "titulo": "El padrino",
  "director": "Francis Ford Coppola",
  "año": 1972
}
'

curl -X POST "http://localhost:9200/peliculas/_doc/2" -H 'Content-Type: application/json' -d'
{
  "id": "2",
  "titulo": "Gladiator",
  "director": "Ridley Scott",
  "año": 2030
}
'

curl -X POST "http://localhost:9200/peliculas/_doc/3" -H 'Content-Type: application/json' -d'
{
  "id": "3",
  "titulo": "Inception",
  "director": "Christopher Nolan",
  "año": 2010
}
'
```

* Modificar el campo "año" de "Gladiator" 2030 -> 2000':
```
curl -X POST "http://localhost:9200/peliculas/_update/2" -H 'Content-Type: application/json' -d'
{
  "doc": {
    "año": 2000
  }
}
'
```
* Borrar "Inception" jeje:
```
curl -X DELETE "http://localhost:9200/peliculas/_doc/3"
```
* Verificar:
```
curl -XGET localhost:9200/peliculas/_search?pretty
```

### Búsquedas:
Se pueden limitar los resultados de búsqueda mediante consultas o filtros. 
Se utiliza el endpoint: _search. 
Ejemplo: buscar en índice "libros" palabra "roja" en campo "titulo":
```
curl -H "Content-Type: application/json" -XGET
'127.0.0.1:9200/libros/_search?pretty' -d '
{ "query":
{ "match":
{ "titulo": "roja" }
}
}'
```
```
Todos los documentos:
{"match_all": {}}
Coincidencia:
{"match": {"titulo": "roja"}}
Coincidencia múltiple:
{"multi_match": {"query": "roja", "fields": ["titulo", "descripcion"]}
Coincidencia de frase:
{"match_phrase": {"descripcion": "la casa era verde", "slop": x}}
Combinaciones booleanas: must, must_not, should:
"query": {
  "bool": {
    "must": [ { "match": { "title": "Verde"}}, { "match": { "anyo": "1928"}} ],
    "must_not": { "match": { "autor": "Jose"}}
  }
}
Valor exacto:
{ "term": { "anyo": 2014 }}
Uno de los valores del listado:
{ "term": { "genero": ["novela","crimen"] }}
Rango de números: gt, gte, lt, lte
{ "range": { "anyo": {"gte": 2010 }} }
Existencia del campo:
{ "exists": {"field": "editorial"} }
Falta del campo:
{ "missing": {"field": "editorial"} }
```
### Ejercicio 3:
Crear índice con mapping:
```
curl -XPUT localhost:9200/shakespeare -H 'Content-Type: application/json' -d'
{
    "mappings": {
        "properties": {
            "speaker": {"type": "keyword"},
            "play_name": {"type": "keyword"},
            "line_id": {"type": "integer"},
            "speech_number": {"type": "integer"}
        }
    }
}'
```
Cargar dataset:
```
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/_bulk?pretty' --data-binary @shakes_cut.json
```
Búsquedas:
```
curl -XGET 'localhost:9200/shakespeare/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "match_phrase" : {
            "text_entry" : "To be or not to be"
        }
    }
}'

curl -XGET 'localhost:9200/shakespeare/_search?pretty' -H 'Content-Type: application/json' -d'
{
    "query" : {
        "bool" : {
            "must" : [
                { "match" : { "play_name" : "Antony and Cleopatra" } },
                { "match" : { "speaker" : "OCTAVIUS CAESAR" } }
            ]
        }
    }
}'
```
