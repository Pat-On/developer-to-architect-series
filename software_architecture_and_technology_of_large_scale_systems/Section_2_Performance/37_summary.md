# Summary

- Performance Problems are a result of requests/job queue building up in a system
- Performance Measurement - Latency, Throughput, and Resource Saturation
  - Watch out tail latency for hidden problems or future problems
- Improving Latency
  - Reduce requests response time of serial requests by improving resource utilization
    - CPU, Network, Memory, Disk
  - Caching - Minimze fetching frequent read rarely mutated data from disk or network
- Improving Throughput 
  - Improve concurrency of concurrent requests/jobs
  - Minimize request/job serialization
    - reduce lock contention by reducing lock granularity, lock striping, lock splitting and CAS
    - Prefer Optimistic locking over Pessimistic locking when lock contention is low
    - Eliminate deadlocks when using pessimistic locking

