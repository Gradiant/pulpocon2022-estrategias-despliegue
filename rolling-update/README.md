RollingUpdate deployment
========================

> Version B se implementa lentamente y va reemplazando a la version A. También conocido como ramped o incremental.

![kubernetes rollingUpdate deployment](grafana-rolling-update.png)

La estrategia rollingUpdate consiste en desplagar lentamente una version de una aplicación reemplazando
las instancias una tras otra hasta qye todas las instancias son desplegadas.
Normalmete sigue el siguiente proceso: con un pool de version A detras de un loadBalancer, se despliega una instancia de versión B. Cuando el servio está listo para aceptar tráfico, la instancia es añadida al pool. 
Entonces una instancia de la versión A es eliminada del pool y se cierra.

Despendiendo del sistema que se encarge de este rollingUpdate, se pueden configurar los siguientes parámetros 
para aumentar el tiempo de despliegue:

- Paralelismo, max batch size: Number of instancias concurrentes para desplegar.
- Max surge: Cuantas instancias agregar además de la cantidad actual.
- Max unavailable: Número de instancias no disponibles en el proceso de rollingUpdate.

## Pasos a seguir

1. version 1 está disponible
1. deploy version 2
1. esperar hasta que todas las réplicas sean reemplazadas con version 2

## En la práctica

```bash
cd rolling-update
# Deploy the first application
$ kubectl apply -f app-v1.yaml
$ kubectl apply -f ingress-app.yaml

# Test if the deployment was successful
$ curl myapp.fbi.com

# To see the deployment in action, open a new terminal and run the following
# command
$ watch kubectl get pods

# Then deploy version 2 of the application
$ kubectl apply -f app-v2.yaml

$ while sleep 0.1; do curl "myapp.fbi.com"; done

# In case you discover some issue with the new version, you can undo the
# rollout
$ kubectl rollout undo deploy my-app

# If you can also pause the rollout if you want to run the application for a
# subset of users
$ kubectl rollout pause deploy my-app

# Then if you are satisfy with the result, rollout
$ kubectl rollout resume deploy my-app
```

### Cleanup

```bash
$ kubectl delete all -l app=my-app
$ kubectl delete -f ingress-app.yaml
```