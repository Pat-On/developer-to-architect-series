# RDBMS Limitations


- RDBMS limitations:
    - single instance
    - < 1~5 TB of data


![Alt text](image.png)


## Partitioning Data Load
- Replication
  - high read load
    - eventually consistent read queries
  - high availability
  - write load is still a problem

![Alt text](image-1.png)

## vertical partitioning - application level - micro services

![Alt text](image-2.png)

- Vertical partitioning
  - microservices architecture
    - independent development and deployment
  - within a database
    - acis, joins, constraints
  - across databases
    - no integrity constraints
    - no query joins
    - no acid transactions
  - complexity of maintaining multiple databases

## Partitioning - DB Record Level

![Alt text](image-3.png)

- Data Partitioning 
  - record level
- data replication
- routing request to serving instances
- schema on demand
- limited indexing
  - puts limit on query patterns

## Hybrid Approach

![Alt text](image-4.png)

- RDBMS 
  - for services that need strong consistency
- Distributed database
  - for services that can be eventually consistent

