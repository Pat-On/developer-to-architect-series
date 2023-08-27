# Static Routing Challenge and Discovery Services

![Alt text](image-4.png)

- simplification of the communication - client -> gateway


## What about order service -> inventory service
- how to omit static communication?
- it is called static communication configuration

## issue
![Alt text](image-5.png)


## solution - Discovery Service
![Alt text](image-6.png)
- provide dynamic configuration
- discovery service is updated about heart bit that server is alive

## Different options

![Alt text](image-7.png)

- Consul
- Netflix Eureka (availability over consistency)
- and more....