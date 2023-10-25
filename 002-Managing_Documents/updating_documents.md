# Updating Documents
## Updating an existing field
```json
POST /products/_update/100
{
  "doc": {
    "name": "Shampoo",
    "in_stock": true
  }
}
```

## Adding a new field
```json
POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"],
    "qty": 5
  }
}

### How does the update API Work
- It updates a doc even though elastic documents are immutable.
- The current document is retrieved.
- The field values are changed.
- The existing document is replaced with the modified document.
- We could do the exact same thing at the application level.

# Scripted updates
## Reducing the current value of 'qty' by one
```json
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.qty--"
  }
}
```

## Assigning an arbitrary value to `qty`
```json
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.qty = 10"
  }
}
```

## Using parameters within scripts
```json
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.qty -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}
```

## Conditionally setting the operation to `noop`
```json
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.qty == 0) {
        ctx.op = 'noop';
      }
      
      ctx._source.qty--;
    """
  }
}
```

## Conditionally update a field value
```json
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.qty > 0) {
        ctx._source.qty--;
      }
    """
  }
}
```

- There is a minor difference between the two commands above.
- Both of them are used to accomplish the same thing which is decreasing the quantity if it is above 0.
- However, in the first query, we set the operation to 'no operation (no-op)'. This means that the 'result' key in the object that we get after using the update command, will not have the word "updated". However, if we use the second approach, we would still have the word, 'updated' even though we didn't update the field.

## Conditionally delete a document
```json
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.qty < 0) {
        ctx.op = 'delete';
      }
      
      ctx._source.qty--;
    """
  }
}
```
- The command above deletes the document if the condition is satisfied.
- Usually used when  you no longer need the data after reaches a certain point.