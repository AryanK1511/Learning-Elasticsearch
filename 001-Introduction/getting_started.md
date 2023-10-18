# Setting up an environment on your computer

### Elasticsearch
1. Download Elasticsearch from [here](https://www.elastic.co/downloads/elasticsearch).
2. Download Kibana from [here](https://www.elastic.co/downloads/kibana)
3. Navigate to the elasticsearch folder using the terminal.
4. Run the command `bin/elasticsearch` to start elasticsearch.
5. Navigate to the kibana folder using the terminal.
6. Run the command `bin/kibana` to start kibana.
7. Link for elastic in my case: `http://localhost:5601/app/home#/`
8. Additional Commands:
    - `bin/elasticsearch-reset-password -u elastic`: To reset elastic password
    - `bin/elasticsearch-create-enrollment-token -s node`: To create a new enrollment token
    -  `xattr -d -r com.apple.quarantine /path/to/kibana`: To disable apple security stuff

## Architecture of Elasticsearch

### Nodes
- A node is an instance of Elasticsearch that stores data.
- Multiple nodes can be run on the same machine.
- Allows for distributed storage of data across machines.

### Clusters
- A cluster is a collection of related nodes that collectively store all data.
- Multiple clusters can be created, but one is usually sufficient.
- Clusters are independent by default.

### Cluster Formation
- When a node starts, it either joins an existing cluster or forms its own cluster.
- A node will always be part of a cluster, even if it's the only node.
- Single-node clusters are suitable for development but have limitations.

### Documents
- A document is a JSON object containing desired data.
- When indexed, the original JSON object is stored along with internal metadata.
- Documents represent units of information (e.g., person, product, review).

### Indices
- Documents are organized within indices.
- An index groups similar documents and provides configuration options for scalability and availability.
- Indices are logical collections of documents.

### Example
- Document (e.g., person) contains fields (e.g., name, country).
- Document is stored within an index (e.g., "people").
- Indices allow for logical grouping of related documents.

### Key Takeaways
- Elasticsearch cluster consists of nodes responsible for data storage.
- Nodes can run on physical, virtual machines, or containers.
- Data is stored as documents, representing units of information.
- Documents are grouped within indices for logical organization.
- Clusters are formed automatically when nodes start.

# Using Elasticsearch

## Accessing Elasticsearch through REST API
- Elasticsearch cluster exposes a REST API.
- HTTP verbs (GET, POST, PUT, DELETE) are used for different actions.
- Kibana's Console tool provides a convenient interface to interact with the API.

## Some basic commands to try out
- Use `GET _cluster/health` to retrieve cluster health information.
- __Tip:__ Leading forward slash is optional in Console tool.
- Use `GET _cat/nodes?v` for a human-readable list of nodes.
- Cloud deployments may have multiple nodes for high availability.
- Use GET _cat/indices?v to see existing indices.
- Initially, no user-created indices (system indices are present).
- System indices are used by Elasticsearch and Kibana for internal storage.
- Adding expand_wildcards=all parameter displays hidden system indices.
- Use `PUT /<index_name>` to create a new index using the default settings with one primary and one replica shard. The health would be yellow as there are no backup nodes.

## Making requests to elasticsearch from terminal using curl
- Start elasticsearch from your terminal
- curl --cacert config/certs/http_ca.crt -u elastic:<your_password> -X GET https://localhost:9200/

# Sharding and Scalability
Elasticsearch uses sharding to scale both data storage and disk space. Sharding is done at the index level, allowing large indices to be divided into smaller, manageable pieces called shards.

## Purpose of Sharding
1. **Horizontal Scaling**: Dividing an index into multiple shards allows for distributing data across nodes, increasing storage capacity.
2. **Parallelization of Queries**: Queries can be executed on multiple shards simultaneously, improving search performance and throughput.

## Shards and Nodes
- A shard represents a portion of an index and is stored on a single node.
- Multiple shards can exist on the same node, but they can also be distributed across different nodes.
- Each shard is an Apache Lucene index.

## Sharding Example
- If we have two nodes, each with 500GB of storage, and a 600GB index, we can divide it into two shards, each requiring 300GB of disk space.
- This allows us to store one shard on each node without running out of disk space.

## Shard Limitations
- While a shard does not have a predefined size in terms of allocated disk space, there is a limit to the number of documents a shard can store (just over two billion documents).

## Sharding Configuration
- Indices can be configured to have a specific number of shards at creation. The default is one shard per index.
- For larger indices, consider increasing the number of shards at index creation (e.g., five shards for millions of documents).

## Considerations for Shard Count
- The optimal number of shards depends on various factors, including the number of nodes, node capacity, index sizes, query frequency, and more.
- The default is one shard at the start which used to be 5 shards before version 7. This led to oversharding.
- A common starting point is five shards for indices with millions of anticipated documents.
- Increase the number of shards using the _split API_.
- Reduce the number of shards using the _shrink API_.

## Conclusion
- Sharding enables Elasticsearch to efficiently handle large volumes of data, improving storage capacity and query performance.
- Understanding shard configuration and making informed decisions about shard count can significantly impact the efficiency and scalability of an Elasticsearch cluster.

# Elasticsearch Replication
Replication in Elasticsearch provides fault tolerance and failover mechanisms to prevent data loss in case of node or disk failures. It involves creating copies (replicas) of each shard in an index. This is done by default in elasticsearch.

## Purpose of Replication
1. **Data Redundancy**: Replicas ensure that data is not lost if a node or disk fails.
2. **Failover Mechanism**: Replicas allow for continued operation even if a primary shard becomes unavailable.

## Replica Shards
- Replica shards are complete copies of primary shards and can serve search requests just like them.
- A primary shard and its replica shards form a replication group.

## Replica Configuration
- Replication is configured at the index level.
- By default, Elasticsearch enables replication with one replica per shard.

## Fault Tolerance
- Replica shards are never stored on the same node as their corresponding primary shard, ensuring redundancy.
- In case of node failure, there will always be at least one copy of shard data available on a different node.

## Adding Replicas
- Replicas can be added when creating an index.
- Additional nodes are required to fully utilize the benefits of replication.

## Replication in Action
- Example scenario: Two indices, each with one shard and one replica, initially on a single node.
- Replication becomes effective when an additional node is added, ensuring data availability. You need to assign the replica shard to a node.
- Kibana defaults don't have replica shards. We need to add more nodes and the 'auto_expand_replicas' will dynamically change the number of replicas.

## Snapshot vs. Replication
- **Snapshot**: Backs up the current state of the cluster or specific indices, enabling restoration to a previous state.
- **Replication**: Prevents data loss and increases index throughput, serving live data.

## Replication for Throughput
- Replica shards can be queried simultaneously, increasing the throughput of an index.
- Utilizes multiple cores of a node's CPU for parallel processing.

## Considerations for Replica Configuration
- Optimal replica count depends on the criticality of the system and the risk of simultaneous node failures.
- For critical systems, consider replicating shards twice or more.

## Summary
- Replication in Elasticsearch provides data redundancy and failover mechanisms to prevent data loss.
- It also increases the throughput of an index by allowing simultaneous queries on replica shards.
- Balancing nodes, shards, and replicas requires careful consideration of system requirements and potential risks.

# Adding Nodes to an Elasticsearch Cluster
Sharding and replication are essential for scalability and fault tolerance in Elasticsearch. Adding more nodes is crucial to support increased document storage and maintain data redundancy.

## Adding Nodes (Local Setup)
- Cloud deployments handle node addition via the cloud interface.
- For local setups, follow these steps:

1. **Check Cluster Status**
   - Use Cluster Health API to assess the current state of the cluster.
   - Note: Cluster contains a single node.
   - `GET _cluster/health`

2. **Prepare Elasticsearch Archive**
   - Ensure you have the Elasticsearch archive used during the initial setup.

3. **Extract Archive for Each Additional Node**
   - Do not copy existing node's directory; extract fresh archive for each new node.

4. **Configure Node Names**
   - Edit `elasticsearch.yml` in the `config` directory to assign meaningful names to new nodes.

5. **Generate Enrollment Token**
   - Use `bin/elasticsearch-create-enrollment-token` script to generate a token (e.g., scope: "node").
   - Ensure existing node is running.
   - This has to be done in the original node folder.

6. **Start Second Node**
   - Run the script with the enrollment token to start the new node.
   - `bin/elasticsearch --enrollment-token <your_enrollment_token>`.
   - The working directory should be the second node's root directory.

7. **Verify Second Node Addition**
   - Check the terminal output of the first node for confirmation of the new node's addition.

8. **Check Cluster Health**
   - Verify that the cluster now has two nodes and the health status has transitioned to "green."
   - Replicas have been assigned.

9. **Distribute Shards**
   - Confirm primary and replica shards are stored on different nodes, as expected.

10. **Adding a Third Node**
    - Repeat steps 5-9 for a third node if desired.

## Considerations
- For three-node clusters, ensure at least two nodes are running for proper master node election.

## Simulating Node Failure
- Simulate a node failure by abruptly terminating the node process.

## Handling Node Departure
- Observe master node's output to see notification of node leaving due to lost connection.
- Delayed shard reallocation occurs after about a minute.
- This is because we do not want the shards to be reassigned in case of a small network failure or something. This could be an expensive process.

## Restoring Node
- Restart the failed node and observe shard distribution to all available nodes.

## Conclusion
- Adding nodes to a development cluster enhances scalability and fault tolerance in Elasticsearch.
- Elasticsearch automates many aspects of node management, simplifying cluster operations.

# Roles in an Elasticsearch Cluster
An Elasticsearch cluster comprises one or more nodes, where data is stored on shards. Each node can have specific roles that define its purpose within the cluster.

## Node Roles
1. **Master Role**
   - Makes a node eligible for being the cluster's master node.
   - Master node handles cluster-wide actions (e.g., creating/deleting indices, shard allocation).
   - Master node is elected based on a voting process.
   - Large clusters often have dedicated master nodes for stability.
   - Configuration: `node.master: true | false`

2. **Data Role**
   - Enables a node to store a portion of the cluster's data.
   - Performs queries (search queries and modifications) related to stored data.
   - Typically assigned for most nodes, especially in small to medium-sized clusters.
   - Large clusters may have dedicated data nodes to separate master and data nodes.
   - Configuration: `node.data: true | false`

3. **Ingest Role**
   - Allows a node to run ingest pipelines.
   - Ingest pipelines define steps (processors) for manipulating documents before indexing.
   - Useful for relatively simple data transformations.
   - Configuration: `node.ingest: true | false`

4. **Machine Learning Node**
   - Enables a node to run machine learning jobs.
   - Setting `xpack.ml.enabled` controls the node's capability to respond to machine learning API requests.
   - Useful for dedicated machine learning nodes.
   - Configuration: `node.ml: true | false`

5. **Coordination Node**
   - Responsible for processing requests and delegating work internally.
   - Does not search data on its own; work is delegated to data nodes.
   - Helpful for load balancing in large clusters.
   - You have to set the configurations of all the other nodes to false.

6. **Voting-Only (Optional)**
   - Participates in master node election process but cannot be elected as the master node.
   - Rarely used and specific to large clusters.

## Default Roles in the Cluster
- In Kibana, view nodes with their respective roles:
  - Columns of interest: "node.role" and "master".
  - Example: "dim" stands for "data," "ingest," and "master."

## When to Change Node Roles
- Typically done for large clusters, not small ones.
- Consider changing roles when:
  - Scaling throughput is required (high hardware usage).
  - Optimizing cluster based on hardware resources.
- **Caution**: Changing roles should be done with understanding and a clear reason.

## Conclusion
- Understanding node roles helps optimize cluster performance and stability.
- Changing roles is a strategic decision, usually for larger clusters or specific performance needs.
- Default roles are generally suitable unless specific requirements arise.