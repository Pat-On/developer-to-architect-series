# Internal Cluster Monitoring

- Periodic exchange of heartbeats between redundancy cluster nodes
  - Requires protocols for communication and recovery
- Useful for stateful cluster components
  - Examples are NoSQL DB and Load Balancers

![Alt text](image-16.png)


So it is like "aware monitoring"

![Alt text](image-17.png)

So Service instance know if something happened in other instance

![Alt text](image-18.png)

! it is complicated to achieve
So it is mostly done only for:
- databases 
- load balancers