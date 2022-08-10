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

#Install ingress Controller again with helm and metrics for prometheus to check the deployments
kubectl delete -f ../local-kind/resources/ingress-nginx.yaml

helm install nginx-ingress ingress-nginx/ingress-nginx \
 --namespace ingress-nginx
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
kubectl create ns test
kubectl apply -k https://github.com/fluxcd/flagger//kustomize/podinfo?ref=main
# Verify podinfo pods
kubectl get pods -n test
# Ingress with url "podinfo.fbi.com" for podinfo
kubectl apply -f podinfo-ingress.yaml
# Deploy Flagger’s load tester, which allows canary resources to test releases by sending HTTP requests
helm install flagger-loadtester flagger/loadtester -n test
# Define Canary Resource
kubectl apply -f podinfo-canary.yaml

#Get resources deployed
kubectl get pods -n test
kubectl get canaries -n test
kubectl get services -n test
kubectl describe canary podinfo -n test

# Test if the deployment was successful
curl podinfo.fbi.com

# Then deploy A Canary Release, version 2 of the application
kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:6.0.3 -n test
# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo -n test

# Test the second deployment progress
while true; do  \
  kubectl get canaries; \
  curl http://podinfo.fbi.com/;  \
  echo "";  \
  sleep 0.5; \
done

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo -n test

kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:3.1.1 -n test
while true; do   \   
  kubectl get canaries; \    
  curl http://podinfo.fbi.com/; \
  curl http://podinfo.fbi.com/status/500; \  
  echo "";  \
  sleep 0.5; \
done

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo -n test

kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:6.0.3 -n test
watch curl http://podinfo.fbi.com//status/500

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo -n test
```

### Cleanup

```bash
kubectl delete -f podinfo-ingress.yaml
kubectl delete -f podinfo-canary.yaml
kubectl delete deploy -l app=podinfo
```