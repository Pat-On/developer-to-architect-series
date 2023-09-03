# High Availability with K8s

## creating more instances manually

```sh
kubectl get deployments -n service
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
admin-svc-pods       1/1     1            1           15m
auth-svc-pods        1/1     1            1           15m
gateway-svc-pods     1/1     1            1           15m
inventory-svc-pods   1/1     1            1           15m
order-svc-pods       1/1     1            1           15m
product-svc-pods     1/1     1            1           15m
```

```sh
kubectl scale deployment gateway-svc-pods --replicas 3 --namespace service
deployment.apps/gateway-svc-pods scaled
```