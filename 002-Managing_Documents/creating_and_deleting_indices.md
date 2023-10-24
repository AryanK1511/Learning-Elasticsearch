# Creating and Deleting Indices

### Creating an Index with Custom Settings

```json
PUT /products
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 2
  }
}
```

### Deleting an Index
```json
DELETE /pages
```