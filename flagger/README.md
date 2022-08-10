Flagger despliegue progresivo automático canary
===================
<!--
> Version A termina entonces la version B es deplegada.

![kubernetes recreate deployment](grafana-recreate.png)

La estrategia recreate es un dummy deployment que consiste en parar la versión A e implementar la versión B después de que se para y elimina la versión A. Esta técnica implica downtime del servicio que depende de las duraciones del shutdown (apagado) y del boot (arranque) de la aplicación. 

-->
## Pasos a seguir

1. re-instalamos el ingress controller para que le envíe métricas al prometheus
1. instalamos flagger
1. desplegar aplicacion podinfo 6.0.1, flagger loadtester y el recurso canary 
1. cambiamos la versión de la aplicacion podinfo 6.0.3, mandamos tráfico para verificar ok y versión cambia ok 
1. cambiamos la version de la aplicación a la version 3.1.1, manadamos tráfico 1/2 ok y la otra 1/2 ko y version no cambia


## En la práctica

```bash

#Install ingress Controller again with helm and metrics for prometheus to check the deployments
kubectl delete -f ../local-kind/resources/ingress-nginx.yaml

kubectl create ns ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx \
 --namespace ingress-nginx \
 --set controller.metrics.enabled=true \
 --set controller.podAnnotations."prometheus\.io/scrape"=true \
 --set controller.podAnnotations."prometheus\.io/port"=10254 \
 --version 4.0.6 \
 -f nginx_ingress_values.yaml

# Install Flagger
helm repo add flagger https://flagger.app
helm repo update
helm install flagger flagger/flagger \
--set prometheus.install=true \
--set meshProvider=nginx

# Deploy Applicacion podinfo
kubectl apply -k https://github.com/fluxcd/flagger//kustomize/podinfo?ref=main
kubectl get pods
# Ingress with url "podinfo.fbi.com" for podinfo
kubectl apply -f podinfo-ingress.yaml
# Deploy Flagger’s load tester, which allows canary resources to test releases by sending HTTP requests
helm install flagger-loadtester flagger/loadtester
# Define Canary Resource
kubectl apply -f podinfo-canary.yaml

#Get resources deployed
kubectl get pods
kubectl get canaries
kubectl get services
kubectl describe canary podinfo

# Test if the deployment was successful
curl podinfo.fbi.com

# Then deploy A Canary Release, version 2 of the application
kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:6.0.3
# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo

# Test the second deployment progress
while true; do  \
  kubectl get canaries; \
  curl http://podinfo.fbi.com/;  \
  echo "";  \
  sleep 0.5; \
done

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo

kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:3.1.1
while true; do   \   
  kubectl get canaries; \    
  curl http://podinfo.fbi.com/; \
  curl http://podinfo.fbi.com/status/500; \  
  echo "";  \
  sleep 0.5; \
done

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo

# Dejamos la versión anterior para que quede como está actualmente
kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:6.0.3
```
<!---
watch curl http://podinfo.fbi.com//status/500

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo

    flagger-loadtester
    
    # alerting (optional)
    alerts:
      - name: "qa team Discord"
        severity: warn
        providerRef:
          name: qa-discord

-->


### Cleanup

```bash
kubectl delete -f podinfo-ingress.yaml
kubectl delete -f podinfo-canary.yaml
kubectl delete deploy -l app=podinfo
kubectl delete https://github.com/fluxcd/flagger//kustomize/podinfo?ref=main
helm uninstall flagger-loadtester
helm uninstall flagger
```