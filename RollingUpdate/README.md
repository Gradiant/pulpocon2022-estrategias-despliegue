RollingUpdate deployment
========================

> Version B se implementa lentamente y va reemplazando a la version A. También conocido como ramped o incremental.

![kubernetes rollingUpdate deployment](grafana-rollingUpdate.png)

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
# Deploy the first application
$ kubectl apply -f deployment611.yaml
$ kubectl apply -f service.yaml
# port-forward from 9898 to localhost to port 9000
$ screen -d -m -S "frontend-podinfo_ports" bash -c "kubectl port-forward deployment/frontend-podinfo 9000:9898"
# Test if the deployment was successful
$ curl localhost:9000
{
  "hostname": "frontend-podinfo-bd7b98cf4-5wnqf",
  "version": "6.1.1",
  "revision": "",
  "color": "#34577c",
  "logo": "https://raw.githubusercontent.com/stefanprodan/podinfo/gh-pages/cuddle_clap.gif",
  "message": "greetings from podinfo v6.1.1",
  "goos": "linux",
  "goarch": "amd64",
  "runtime": "go1.17.8",
  "num_goroutine": "9",
  "num_cpu": "8"
}
#Ver screen -ls para ver las que tengas abiertas 
$ screen -ls
# Kill detached session screen
$ screen -S frontend-podinfo_ports -X quit

# To see the deployment in action, open a new terminal and run the following
# command
$ watch kubectl get pods

# Then deploy version 2 of the application
$ kubectl apply -f deployment612.yaml
# port-forward from 9898 to localhost to port 9000
$ screen -d -m -S "frontend-podinfo_ports" bash -c "kubectl port-forward deployment/frontend-podinfo 9000:9898"
# Test the second deployment progress
$ while sleep 0.1; do curl "localhost:9000"; done
#Ver screen -ls para ver las que tengas abiertas 
$ screen -ls
# Kill detached session screen
$ screen -S frontend-podinfo_ports -X quit

# In case you discover some issue with the new version, you can undo the
# rollout
$ kubectl rollout undo deploy frontend-podinfo

# If you can also pause the rollout if you want to run the application for a
# subset of users
$ kubectl rollout pause deploy frontend-podinfo

# Then if you are satisfy with the result, rollout
$ kubectl rollout resume deploy frontend-podinfo
```

### Cleanup

```bash
$ kubectl delete -f deployment612.yaml
$ kubectl delete -f service.yaml
```