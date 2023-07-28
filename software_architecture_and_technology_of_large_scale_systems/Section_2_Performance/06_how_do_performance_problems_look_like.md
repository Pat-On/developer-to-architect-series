# Performance Problems
- How to spot a performance problem?
- How does it looks like?


every performance problem is the result of some queue building somewhere.
- network socket queue, 
- db io queue
- os run queue


- Reasons for queue build-up
  - inefficient slow processing
    - slow running process - inefficient code
  - serial resource access
    - for example withdrawing and deposit money
  - limited resource capacity
    - limited number CPU or Ram (just examples)

