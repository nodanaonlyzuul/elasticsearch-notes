# Elastic Search Notes

## Running & Installing

```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:5.4.1

docker run -p 9200:9200 -e "http.host=0.0.0.0" -e "transport.host=127.0.0.1" docker.elastic.co/elasticsearch/elasticsearch:5.4.1
```

## The API

`<REST Verb> /<Index>/<Type>/<ID>`

This REST access pattern is so pervasive throughout all the API commands that if you can simply remember it, you will have a good head start at mastering Elasticsearch.

### Creating an index:

 curl -u elastic:changeme -XPUT 'localhost:9200/customer?pretty'

### Creating a Document in an Index:


```bash
curl -u elastic:changeme -XPUT 'localhost:9200/customer/external/1?pretty&pretty' -H 'Content-Type: application/json' -d '
{
  "name": "John Doe"
}
'
```

Adds document into the customer index, "external" type, with an ID of 1 as follows:

* `customer` is the Index
* `external` is the [Type](https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html#_type)

Or you can have Elasticsearch create an ID for you.

```bash
curl -u elastic:changeme -XPOST 'localhost:9200/customer/external?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "name": "Jane Doe"
}
'
```

Note this is a **POST**

### Bulk Loading

```bash
curl -u elastic:changeme -H "Content-Type: application/json" -XPOST 'localhost:9200/bank/account/_bulk?pretty&refresh' --data-binary "@./resources/accounts.json"

curl -u elastic:changeme 'localhost:9200/_cat/indices?v'
```

### Fetching a Document

```bash
curl -u elastic:changeme -XGET 'localhost:9200/customer/external/1'
```

returns
```javascript
{
   "_version" : 1,
   "_id" : "1",
   "_index" : "customer",
   "_source" : {
      "name" : "John Doe"
   },
   "_type" : "external",
   "found" : true
}
```

### Deleting an Index

`curl -u elastic:changeme -XDELETE 'localhost:9200/customer?pretty&pretty'`

### Updating a Document

Updates can only be performed on a single document at a time.
In the future, Elasticsearch might provide the ability to update multiple
documents given a query condition (like an SQL UPDATE-WHERE statement).

```bash
curl -u elastic:changeme -XPOST 'localhost:9200/customer/external/1/_update?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "doc": { "name": "Jane Doe", "age": 20 }
}
'
```

Or using some programatic....

```bash
curl -u elastic:changeme -XPOST 'localhost:9200/customer/external/1/_update?pretty&pretty' -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.age += 5"
}
'
```

### Deleting a Document

```bash
curl -u elastic:changeme -XDELETE 'localhost:9200/customer/external/2?pretty&pretty'
```

There's also a way to [delete by query](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-delete-by-query.html).

### Batch Processing

See the [\__bulk_ API documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) for more.

This indexes two documents (ID 1 - John Doe and ID 2 - Jane Doe) in one bulk operation:

```bash
curl -u elastic:changeme -XPOST 'localhost:9200/customer/external/_bulk?pretty&pretty' -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
'
```

This updates the first document (ID of 1) and then deletes the second document (ID of 2) in one bulk operation:

```bash
curl -u elastic:changeme -XPOST 'localhost:9200/customer/external/_bulk?pretty&pretty' -H 'Content-Type: application/json' -d'
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
'
```

The Bulk API does not fail due to failures in one of the actions.
If a single action fails for whatever reason, it will continue to process the
remainder of the actions after it. When the bulk API returns, it will provide a
status for each action (in the same order it was sent in) so that you can check
if a specific action failed or not.

### Searching

You can search by GET using params or use an expressive syntax in the POST.

The GET:

```
curl -u elastic:changeme -XGET 'localhost:9200/customers/_search?q=*&sort=account_number:asc&pretty&pretty'
```

Look into the [JSON Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).

The Equivelant POST:

```bash
curl -XGET 'localhost:9200/bank/_search?pretty' -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
'
```
