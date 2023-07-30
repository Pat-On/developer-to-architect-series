# Summary

- Scalable systems are decentralized, and their components function independently
- To make a system scalable:
  - Cache frequently read and rarely mutating data to reduce load on the backend
  - asynchronous or event driven processing for distributing load over time
  - vertical partitioning of functionality intro independent, stateless, replicated services
  - partitioning and replication of state for extreme scalability
- Scalable systems requires infrastructure:
  - load balancers - HArdware based (L4+ L7) and software based (L7)
  - Discovery services for service discovery and health checks
  - DNS as load balancers at global scale
- Microservices for extreme scalability
  - fully vertical partitioned services and databases leads to eventual consistency