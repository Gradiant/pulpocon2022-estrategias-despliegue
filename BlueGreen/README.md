Blue/green deployment para un único servicio
=====================

> Version B se despliega con la version A, entonces el tráfico de la version A se cambia a la versión B. Se conoce también como red/black deployment.

![kubernetes blue-green deployment](grafana-blue-green.png)

La estartegia blue/gree se dierencia de la de rollingUpdate, ya que la versión B (verde) se implementa junato con la versión A (azul) con la misma cantidad de instancias. 
Despues de probar que la nueva versión es ok, el tráfico se cambia de la A a la B a nivel de balanceador de carga.

> En este ejemplo, desplegamos una nueva version de un único servicio usando la estrategia de blue/green.

## Pasos a seguir

1. version 1 está sirviendo tráfico
1. desplegamos version 2
1. esperamos hasta que version 2 esté disponible
1. cambiamos el tráfico entrante de la versión 1 a la versión 2
1. paramos version 1

## In practice

```bash
# Deploy the first application
$ kubectl apply -f deploy-app-v1-blue.yaml
$ kubectl apply -f service-blue-green.yaml
$ kubectl apply -f ingress-app.yaml

# Test if the deployment was successful
$ curl myapp.fbi.com
2018-01-28T00:22:04+01:00 - Host: host-1, Version: v1.0.0

# To see the deployment in action, open a new terminal and run the following command.
$ watch kubectl get pods

# Then deploy version 2 of the application
$ kubectl apply -f deploy-app-v2-green.yaml

# Wait for all the version 2 pods to be running
$ kubectl rollout status deploy my-app-v2 -w
deployment "my-app-v2" successfully rolled out

# Side by side, 3 pods are running with version 2 but the service still send
# traffic to the first deployment.

# If necessary, you can manually test one of the pod by port-forwarding it to
# your local environment.

# Once your are ready, you can switch the traffic to the new version by patching
# the service to send traffic to all pods with label version=v2.0.0
$ kubectl patch service my-app -p '{"spec":{"selector":{"version":"v2.0.0"}}}'

# Test if the second deployment was successful
$ while sleep 0.1; do curl "myapp.fbi.com"; done

# In case you need to rollback to the previous version
$ kubectl patch service my-app -p '{"spec":{"selector":{"version":"v1.0.0"}}}'

# If everything is working as expected, you can then delete the v1.0.0
# deployment
$ kubectl delete deploy my-app-v1
```

### Cleanup

```bash
$ kubectl delete all -l app=my-app
```

**Se puede aplicacr el despliegue blue/gree para un único servicio o para varios servicios usando un Ingress controller:**

**You can apply the blue/green deployment technique for a single service or
multiple services using an Ingress controller:**

- [multiple services using Ingress](https://github.com/ContainerSolutions/k8s-deployment-strategies/tree/master/blue-green/multiple-services)
