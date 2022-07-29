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

```
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
# Ver screen -ls para ver las que tengas abiertas 
$ screen -ls
# Kill detached session screen
$ screen -S frontend-podinfo_ports -X quit
```

### Cleanup

```bash
$ kubectl delete -f deployment612.yaml
$ kubectl delete -f service.yaml
```