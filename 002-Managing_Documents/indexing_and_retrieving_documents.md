# Indexing and Retrieving Documents

## Indexing document with auto generated ID
```
POST /products/_doc
{
  "name": "Another Product",
  "price": 30.50,
  "in_stock": false
}
```

## Indexing document with custom ID
```
PUT /products/_doc/100
{
  "name": "Another Product",
  "price": 30.50,
  "in_stock": false
}
```

- In the query results, you can see that the "_id" key contains the specified value.
- Remember that the identifier can be any string value and doesn't have to be an integer.
- It's worth noting that you don't even have to create the index in advance.
    - There's a setting named "action.auto_create_index" that can be configured at the cluster level.
    - By default, it's set to "true," meaning if you try to add a document to a non-existing index, the index will be created automatically.
> Note: While this behavior is convenient in development mode, it's best practice to explicitly create indices.

## Retrieving Documents
```
GET /products/_doc/100
```
> You can use the command above to retrieve using ID.

```
GET products/search 
{
  "query": {
    "match_all": {}
  }
}
```
> You can use the above command to retrieve all data.