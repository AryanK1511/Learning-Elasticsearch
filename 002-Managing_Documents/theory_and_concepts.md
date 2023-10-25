# Elasticsearch Routing

- Elasticsearch uses routing to determine which shard a document should be stored in and where to find existing documents.
- When indexing a new document, Elasticsearch uses a formula to calculate which shard it should be stored on. By default, this formula is based on the document's ID.
- The same formula is used for updating or deleting documents.
- When retrieving a document by its ID, Elasticsearch uses the same routing formula based on the specified ID to locate the shard.
- Elasticsearch provides a default routing strategy, making it transparent to users. The default strategy ensures even distribution of documents across shards in an index.

## Metadata
- Elasticsearch stores metadata along with the documents.
  - `_source`: Contains the JSON provided when adding a document.
  - `_id`: Contains the document's identifier.
  - `_routing`: Used in the routing formula (visible only when custom routing is specified during indexing).

## Changing Shard Count
- The number of shards cannot be changed after an index has been created because it is used in the routing formula.
- Changing the shard count would yield different results in the routing formula, potentially causing issues with existing documents.

## Adding Shards
- If more shards are needed, a new index must be created, and documents need to be reindexed. This can be facilitated using the Shrink and Split APIs.

### Bonus Information
- Increasing the number of shards in an existing index can lead to issues with document retrieval, as the routing formula may yield different results.
- Uneven distribution of documents could also occur if additional shards were added to an index.
