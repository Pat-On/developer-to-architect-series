# Elasticsearch Architecture

- Document Oriented data-model
- horizontally scalable
  - petabytes of data
  - data is sharded with key as document id
  - put/get request goes to specific shard
  - search queries go to all shards
- highly available
  - shards are replicated
- index structure is based on merge sort
- index not updated with every update or insert
- index maintained in memory
  - occasionally flushed to disk

![Alt text](image-45.png)