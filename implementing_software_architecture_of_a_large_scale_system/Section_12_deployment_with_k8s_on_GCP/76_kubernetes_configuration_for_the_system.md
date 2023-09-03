# Kubernetes Configuration

check the `kubernetes/config` - wow! nice! 

## ENV variables, namespaces etc

- `namespaces` - way of encapsulation of the services for example
  - in case of this application we divided it by: 
    - data
    - ui
    - test
    - service
    - monitor
- `ConfigMap` - we are providing by this all required .env variables that we need to setup in the namespace
- `SecretConfig` - you can chose this to provide the passwords to the K8s
  - securely stored in the k8s
- `resource-limits` by this we are going to force limits on our services based on cpu and hdd


----

## SERVICES

Workloads

Component = 
  - `LB` (`Service` in case of the kubernetes, perceive it as a reverse proxy) 
  - 
  - + 
  - actual component `instances` - `workloads`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: product-svc
  namespace: service
  labels:
    app: product-svc
  annotations:
    service.kubernetes.io/topology-aware-hints: auto
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  selector:
    app: product-svc
  type: ClusterIP


# Check topologyKeys <- interesting 
```

Deployment - simplify now - pod (pod it is set of containers)
this pod contain only one type of the container

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-svc-pods
  namespace: service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product-svc
  strategy:
    type: RollingUpdate
    rollingUpdate:
       maxUnavailable: 25%
       maxSurge: 1
  template:
    metadata:
      labels:
        app: product-svc
    spec:
      initContainers:
      - name: wait-for-postgres
        image: postgres
        command: [ '/bin/bash','-c', 'until pg_isready -h postgres.data; do echo waiting for postgres; sleep 2; done;' ]
      - name: wait-for-cassandra
        image: cassandra
        command: [ '/bin/bash','-c', 'until echo exit | cqlsh cassandra.data; do echo waiting for cassandra; sleep 5; done;' ]
      containers:
      - name: product-svc-cont
        image: IMAGE_REGISTRY_PATH/services:REVISION_ID
        args: ["product"]
        imagePullPolicy: Always
        ports:
          - containerPort: 8080
        volumeMounts:
          - name: log-data
            mountPath: /var/log/oms
        envFrom:
          - configMapRef:
              name: service-config
        env:
          - name: database.type
            value: CQL
        livenessProbe:
          httpGet:
            path: /status
            port: 8080
          initialDelaySeconds: 180
        readinessProbe:
          httpGet:
            path: /status
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 2
      volumes:
        - name: log-data
          emptyDir: {}
      topologySpreadConstraints:                                                                                                                     
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: product-svc

```

We have pod with two containers in the case of the UI
because of the dynamic and static part of the application
```yaml
--
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app-pods
  namespace: ui
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
       maxUnavailable: 25%
       maxSurge: 1
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app-static-cont
        image: IMAGE_REGISTRY_PATH/lb-web:REVISION_ID
        imagePullPolicy: Always
        ports:
          - containerPort: 80      
        env:
          - name: WEB_HOSTS
            value: localhost
          - name: WEB_PORT
            value: "8000"
      - name: web-app-dynamic-cont
        image: IMAGE_REGISTRY_PATH/web:REVISION_ID
        imagePullPolicy: Always
        ports:
          - containerPort: 8000
        envFrom:
          - configMapRef:
              name: web-config
        livenessProbe:
          httpGet:
            path: /status
            port: 8000
          initialDelaySeconds: 60
        readinessProbe:
          httpGet:
            path: /status
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 2
        volumeMounts:
          - name: log-data
            mountPath: /var/log/oms
      volumes:
        - name: log-data
          emptyDir: {}

```

---

## DATA - stateful services


### StatefulSet
- it will create separated disk for all pods
- statefulset - you can reach specific instance
- the rest is the same! 


```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra-pods
  namespace: data
  labels:
    app: cassandra
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  serviceName: "cassandra"
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
        - name: cassandra-cont
          image: IMAGE_REGISTRY_PATH/cassandra:REVISION_ID
          imagePullPolicy: Always
          ports:
            - protocol: TCP
              containerPort: 9042
          livenessProbe:
            exec:
               command:
               - /bin/bash
               - -c
               - /check-readiness.sh
            initialDelaySeconds: 180
          readinessProbe:
            exec:
               command:
               - /bin/bash
               - -c
               - /check-readiness.sh
            initialDelaySeconds: 60
            periodSeconds: 2
          envFrom:
            - configMapRef:
                name: cassandra-config
          env:
            - name: MAX_HEAP_SIZE
              value: 1024M
            - name: HEAP_NEWSIZE
              value: 512M
          volumeMounts:
            - mountPath: "/var/lib/cassandra"
              name: cassandra-storage-claim
          resources:
            requests:
              cpu: 0.5
              memory: "512Mi"
            limits:
              cpu: 1
              memory: "2048Mi"              
      topologySpreadConstraints:                                                                                                                     
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app: cassandra
  volumeClaimTemplates:
  - metadata:
      name: cassandra-storage-claim
      namespace: data
    spec:
      accessModes: 
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

```

### Getting the .env from config deployed to k87s
```yaml
          envFrom:
            - configMapRef:
                name: cassandra-config
```

### Deamon Set
- this type of deployment is making sure that for example fluently and jeager are going to spread on all our node levels

## Scaler - horizontal autocale

```yaml
---
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: gateway-svc-scaler
  namespace: service
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: gateway-svc-pods
  minReplicas: 1
  maxReplicas: 3
  targetCPUUtilizationPercentage: 50

```