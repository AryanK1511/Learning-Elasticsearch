# Something

#### Get Health of Cluster
```GET _cat/health?v```

#### Get the indices
```GET _cat/indices?v```

#### Get the nodes
```GET _cat/indices?v```

#### Add an index
```PUT /sales```

#### Add a document
```
PUT /sales/_doc/123
{
    "orderID": "123",
    "orderAmount": "500"
}
```

#### Retrieve a document
```
GET /sales/_doc/123
```

#### Delete an index
```
DELETE /sales
```

#### Using the curl command to ingest data into elasticsearch
Make sure that you make this request from the repository your data is in as we have specified a relative path.
```
curl -XPOST -i -H "Content-Type: application/x-ndjson" -H "Authorization: SlhnWWtJc0JLTzU3c0kyYnNBYkI6ODlrSmYtUklRWWlqbC1zckZhamgydw==" "http://127.0.0.1:9200/bank/_bulk?pretty" --data-binary @accounts.json; echo
```

#### Get all search results within a specific index
GET bank/_search

#### Using the match query
GET bank/_search
{
  "query": {
    "match": {
      "state": "CA"
    }
  }
}

#### Using boolean with match and must
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": {"state": "CA"} },
        { "match": {"employer": "Techade"}}
      ]
    }
  }
}

#### Opposite of must
GET bank/_search
{
  "query": {
    "bool": {
      "must_not": [
        { "match": {"state": "CA"} },
        { "match": {"employer": "Techade"}}
      ]
    }
  }


#### Combination of must and must not
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {
          "state": "CA"
        }}
      ], 
      "must_not": [
        { "match": {"employer": "Techade"}}
      ]
    }
  }
}

# Boost results (Give more priority)
GET bank/_search
{
  "query": {
    "bool": {
      "should": [
        { "match": {"state": "CA"} },
        { "match": {
          "lastname": {
            "query": "Smith",
            "boost": 3
            }
          }
        }
      ]
    }
  }
}