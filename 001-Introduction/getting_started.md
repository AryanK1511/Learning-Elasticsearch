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

## Making requests to elasticsearch from terminal using curl
- Start elasticsearch from your terminal
- curl --cacert config/certs/http_ca.crt -u elastic:<your_password> -X GET https://localhost:9200/