Recreate despliegue
===================

> Version A termina entonces la version B es deplegada.

![kubernetes recreate deployment](grafana-recreate.png)

La estrategia recreate es un dummy deployment que consiste en parar la versión A e implementar la versión B después de que se para y elimina la versión A. Esta técnica implica downtime del servicio que depende de las duraciones del shutdown (apagado) y del boot (arranque) de la aplicación. 

## Pasos a seguir

1. version 1 está disponible
1. eliminar version 1
1. desplegar version 2
1. esperar hasta que todas las réplicas estén listas

## En la práctica

```bash
cd recreate
# Deploy the first application
$ kubectl apply -f app-v1.yaml
$ kubectl apply -f ingress-app.yaml

# Test if the deployment was successful
$ curl myapp.fbi.com

# To see the deployment in action, open a new terminal and run the following command.
$ watch kubectl get pods

# Then deploy version 2 of the application
$ kubectl apply -f app-v2.yaml

# Test the second deployment progress
$ while sleep 0.1; do curl "myapp.fbi.com"; done
```

### Cleanup

```bash
$ kubectl delete all -l app=my-app
$ kubectl delete -f ingress-app.yaml
```