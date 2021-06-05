# Elastic Search (7.0)

1. Document in elastic search is single data entity or a row in a SQL DB and Document in MongoDB

2. Index is a collection where Documents stays.
   Table in SQL DB and Collection i MongoDB

> Sample Data and Schema

```bash
$ wget http://media.sundog-soft.com/es7/shakespeare_7.0.json
```

Index -

Document 1

```
Space: The final frontier. these are the voyages.
```

Document 2

```
He's bad, he's number one. He's the space cowboy with the laser gun.
```

Inverted Index -

```
space - 1, 2
the   - 1, 2
final - 1
he    - 2
bad   - 2
...
```

Originally elastic search stores in `Document` it's in and in which position it's in also.

### Relevance

> TF-IDF

- TF - Term Frequency - How Often a term appears in a given document.
- IDF - Inverse Document Frequency - How often a term appears in all documents
- TF-IDF - Measures the relevance of a term in a document.

## New In Elastic Search 7

- The concept of document type are deprecated.
- ES SQL is production ready
- Lots of changes to defaults (ie number of shards, replication)
- Lucene 8 Under the hood
- Several X-Pack plugins now included with ES itself.
- Bundled with java runtime (No Specific Java runtime is needed beforehand)
- Index-Life cycle management. (ILM)
- Cross Cluster replication is production ready

## Architecture

- Sharding
- Multiple Nodes

Primary and Replica Shards. Can't change primary shards later on.

> Worst case - Re-index all data to add primary shards

```http
PUT /indexname
{
    "settings": {
        "number_of_shards" : 3, // primary shards
        "number_of_replicas" : 1 // replica shards
    }
}
```

How many shards at the we have ?
Ans - 6

Description

- 3 Total Primary
- 1 Replica for Each Primary
  so for 1 Primary 1 Replica, in Total - 2
  sor 3 primary (3 \* 1) Replica Total - 6

```http
PUT /indexname
{
    "settings": {
        "number_of_shards" : 3, // primary shards
        "number_of_replicas" : 2 // replica shards
    }
}
```

How many shards at the we have ?
Ans - 9

Description

- 3 Total Primary
- 2 Replica for Each Primary
  so for 1 Primary 2 Replica, in Total - 3
  sor 3 primary (3 \* 2) Replica Total - 9

* The schema for your documents are defined by ?

  Ans - `Index`

* What Purpose the inverted indices serve ?

  Ans - `They Quickly map search terms to documents`

* What Purpose the inverted indices serve ?

  Ans - `They Quickly map search terms to documents`

## Mapping

- What is mapping ?
  Ans - A mapping is a schema definition. Elastic search has a reasonable defaults but sometimes you need to customize them.

```curl
curl -H "Content-Type: application/json" -XPUT 127.0.0.1:9200/movies -d '
{
    "mappings" : {
        "properties" : {
            "year" : { "type" : "date }
        },
    },
}'
```

> Field types

- string
- byte
- short
- integer
- long
- float
- double
- boolean
- date

```
"properties" : {
    "user_id" : {
        "type" : {
            "long"
        }
    }
}
```

> Field Index

Do you want this field indexed for full text search ?

- analyzed
- not_analyzed
- no

```
"properties" : {
    "genre" : {
        "index" : "not_analyzed"
    }
}
```

> Field analyzer

Define your tokenizer and token filter.

- standard
- whitespace
- simple
- english

```
"properties" : {
    "description" : {
        "analyzer" : "english"
    }
}
```

1. Character Filters - Removes HTML encoding, `convert & to and`
2. Tokenizer - Split string on whitespace / punctuation / non-letters
3. Token Filters - Lowercasing, stemming, synonyms, stopwords
   - Standard - Split on word boundaries, removes punctuation, lowercases.
     good choice if language is unknown.
   - Simple - Splits on anything that isn't a letter, and Lowercases.
   - Whitespace - splits on any white space but doesn't lowercase.
   - Language - Accounts for language-specific stopwords and stemming

> Hands On

- Mapping

Create Mapping

```json
curl -XPUT 127.0.0.1:9200/movies -d '
{
        "mappings" : {
                "properties" : { "year" : { "type" :  "date" }}
        }
}
'
```

Output

```json
{ "acknowledged": true, "shards_acknowledged": true, "index": "movies" }
```

Verify mapping was successful or not

```
curl -XGET "127.0.0.1:9200/movies/_mapping"
```

Output

```json
{ "movies": { "mappings": { "properties": { "year": { "type": "date" } } } } }
```

Insert Data after mapping of data

```json
curl -XPOST 127.0.0.1:9200/movies/_doc/109487 -d '
{
    "genre" : ["IMAX", "Sci-Fi"],
    "title" : "Interstellar",
    "year" : 2014
}'
```

Output

```json
{
  "_index": "movies",
  "_type": "_doc",
  "_id": "109487",
  "_version": 1,
  "result": "created",
  "_shards": { "total": 2, "successful": 1, "failed": 0 },
  "_seq_no": 0,
  "_primary_term": 1
}
```

Get Data

```
curl -XGET "127.0.0.1:9200/movies/_search?pretty"
```

Output

```json
{
  "took": 707,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "movies",
        "_type": "_doc",
        "_id": "109487",
        "_score": 1.0,
        "_source": {
          "genre": ["IMAX", "Sci-Fi"],
          "title": "Interstellar",
          "year": 2014
        }
      }
    ]
  }
}
```

Insert Many Documents

```
curl -XPUT 127.0.0.1:9200/_bulk?pretty --data-binary @movies.json
```

Output

```json
{
  "took": 123,
  "errors": true,
  "items": [
    {
      "create": {
        "_index": "movies",
        "_type": "_doc",
        "_id": "135569",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "create": {
        "_index": "movies",
        "_type": "_doc",
        "_id": "122886",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "create": {
        "_index": "movies",
        "_type": "_doc",
        "_id": "109487",
        "status": 409,
        "error": {
          "type": "version_conflict_engine_exception",
          "reason": "[109487]: version conflict, document already exists (current version [1])",
          "index_uuid": "HsWZTa3jSoWQyG064JSZ3w",
          "shard": "0",
          "index": "movies"
        }
      }
    },
    {
      "create": {
        "_index": "movies",
        "_type": "_doc",
        "_id": "58559",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 3,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "create": {
        "_index": "movies",
        "_type": "_doc",
        "_id": "1924",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 4,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}
```
