# Scalability Principles

- `Decentralization` - monolith is an anti-pattern for scalability
  - more workers - instances, threads
  - specialized workers - services
- `Independence` 
  - multiple workers are as good as a single worker if they can not work independently 
    - They must work concurrently to maximum extent
  - independence is impeded by
    - shared resources
    - shared mutable data

