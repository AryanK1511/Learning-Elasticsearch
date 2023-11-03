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
```GET bank/_search```

#### Using the match query
```
GET bank/_search
{
  "query": {
    "match": {
      "state": "CA"
    }
  }
}
```

#### Using boolean with match and must
```
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
```

#### Opposite of must
```
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
}
```

#### Combination of must and must not
```
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
```

# Boost results (Give more priority)
```
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
```

#### Term Query
The term query is slightly different from the must query as this will return an element even if the element contains the field. It doesn't have to match exactly.
```
GET bank/_search
{
  "query": {
    "term": {
      "account_number": 516
    }
  }
}
```
This doesn't work with text fields.

#### Terms can return multiple results
```
GET bank/_search
{
  "query": {
    "terms": {
      "account_number": [516,851]
    }
  }
}
```

#### Range Queries
- gte = Greater-than or equal to
- gt = Greater-than
- lte = Less-than or equal to
- lt = Less-than

# Using the range query
```
GET bank/_search
{
  "query": {
    "range": {
      "account_number": {
        "gte": 516,
        "lte": 851,
        "boost": 2
      }
    }
  }
}
```
```
GET bank/_search
{
  "query": {
    "range": {
      "age": {
        "gt": 35
      }
    }
  }
}
```

#### Analyzing and Tokenization
```
GET bank/_analyze
{
  "tokenizer" : "standard",
  "text" : "The Moon is Made of Cheese Some Say"
}
```

> Mixed String
```
GET bank/_analyze
{
  "tokenizer" : "standard",
  "text" : "The Moon-is-Made of Cheese.Some Say$"
}
```

> Use the letter tokenizer
```
GET bank/_analyze
{
  "tokenizer" : "letter",
  "text" : "The Moon-is-Made of Cheese.Some Say$"
}
```

> URL
```
GET bank/_analyze
{
  "tokenizer": "standard",
  "text": "you@example.com login at https://bensullins.com attempt"
}
```

#### Creating an index with two fields but different analyzers
```
PUT /idx1
{
  "mappings": {
    "properties": {
      "title": {
          "type": "text",
          "analyzer" : "standard"
      },
      "english_title": {
          "type":     "text",
          "analyzer": "english"
      }
    }
  }
}
```

#### Using aggregations
```
GET bank/_search
{
  "size": 0,
  "aggs": {
    "states": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

#### Nesting the metric inside the agg
```
GET bank/_search
{
  "size": 0,
  "aggs": {
    "states": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "avg_bal": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

#### Breakdown further with Nesting
```
GET bank/_search
{
  "size": 0,
  "aggs": {
    "states": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "avg_bal": {
          "avg": {
            "field": "balance"
          }
        },
        "age":{
          "terms": {
            "field": "age"
          }
        }
      }
    }
  }
}
```

#### This is the equivalent of using match_all
```
GET bank/_search
{
  "size": 0,
  "query": {
    "match_all": {}
  },
  "aggs": {
    "states": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

# You can also filter on terms
```
GET bank/_search
{
  "size": 0,
  "query": {
    "bool": {
      "must": [
        {"match": {"state.keyword": "CA"}},
        {"range": {"age": {"gt": 35}}}
      ]
    }
  },
  "aggs": {
    "states": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```

# Percentiles
```
GET bank/_search
{
  "size": 0,
  "aggs": {
    "pct_balances": {
      "percentiles": {
        "field": "balance",
        "percents": [
          1,
          5,
          25,
          50,
          75,
          95,
          99
        ]
      }
    }
  }
}
```


# We can use the percentile ranks agg for checking a individual values
```
GET bank/_search
{
  "size": 0,
  "aggs": {
    "bal_outlier": {
      "percentile_ranks": {
        "field": "balance",
        "values": [35000,50000]        
      }
    }
  }
}
```

# Create a histogram
```
GET bank/_search
{
  "size": 0,
  "aggs": {
    "bals": {
      "histogram": {
        "field": "balance",
        "interval": 500
      }
    }
  }
}
```