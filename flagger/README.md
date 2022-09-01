Flagger despliegue progresivo automático canary
===================
<!--
> Version A termina entonces la version B es deplegada.

![kubernetes recreate deployment](grafana-recreate.png)

La estrategia recreate es un dummy deployment que consiste en parar la versión A e implementar la versión B después de que se para y elimina la versión A. Esta técnica implica downtime del servicio que depende de las duraciones del shutdown (apagado) y del boot (arranque) de la aplicación. 

-->
## Pasos a seguir

<!-- 1. re-instalamos el ingress controller para que le envíe métricas al prometheus -->
1. instalamos flagger
1. desplegar aplicacion podinfo 6.0.1, flagger loadtester y el recurso canary 
1. cambiamos la versión de la aplicacion podinfo 6.0.3, mandamos tráfico para verificar ok y versión cambia ok 
1. cambiamos la version de la aplicación a la version 3.1.1, manadamos tráfico 1/2 ok y la otra 1/2 ko y version no cambia


## En la práctica

<!--
#Install ingress Controller again with helm and metrics for prometheus to check the deployments
kubectl delete -f ../local-kind/resources/ingress-nginx.yaml
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install nginx-ingress ingress-nginx/ingress-nginx \
 --namespace ingress-nginx \
 --create-namespace \
 --set controller.metrics.enabled=true \
 --set controller.podAnnotations."prometheus\.io/scrape"=true \
 --set controller.podAnnotations."prometheus\.io/port"=10254 \
 --version 4.0.6 \
 -f nginx_ingress_values.yaml

-->

```bash
# Install Flagger
helm repo add flagger https://flagger.app
helm repo update
helm install flagger flagger/flagger \
--set prometheus.install=true \
--set meshProvider=nginx

# Deploy Applicacion podinfo
kubectl create ns test
# kubectl apply -k https://github.com/fluxcd/flagger//kustomize/podinfo?ref=main
kubectl apply -f deployment.yaml
kubectl apply -f hpa.yaml

kubectl get pods,rs,deploy,horizontalpodautoscaler -n test
# Ingress with url "podinfo.fbi.com" for podinfo
kubectl apply -f podinfo-ingress.yaml
# Deploy Flagger’s load tester, which allows canary resources to test releases by sending HTTP requests
helm install flagger-loadtester flagger/loadtester -n test
# Define Canary Resource
kubectl apply -f podinfo-canary.yaml

#Get resources deployed
kubectl get pods,canaries,svc,ingress -n test
kubectl describe canary podinfo -n test

# Test if the deployment was successful
curl podinfo.fbi.com
curl -k https://podinfo-user20.pulpocon.gradiant.org

# Check by listing the events associated with the podinfo canary
# left in one window to check the evolution
while true; do  \
  kubectl describe canary/podinfo -n test; \
  echo "";  \
  sleep 0.5; \
done

# Then deploy A Canary Release, version 2 of the application
kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:6.0.3 -n test

# Test the second deployment progress 
# left in other window to test app 
# and check the progress deploying until Succeeded or Failed 
while true; do  \
  kubectl get canaries -n test; \
  curl http://podinfo.fbi.com/;  \
  echo "";  \
  sleep 0.5; \
done

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo -n test

kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:3.1.1 -n test
while true; do   \   
  kubectl get canaries -n test; \    
  curl http://podinfo.fbi.com/; \
  curl http://podinfo.fbi.com/status/500; \  
  echo "";  \
  sleep 0.5; \
done

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo -n test

# Dejamos la versión anterior para que quede como está actualmente
kubectl set image deployment/podinfo podinfod=stefanprodan/podinfo:6.0.3 -n test
```

<!---
#kubectl apply -k https://github.com/fluxcd/flagger//kustomize/podinfo?ref=main

watch curl http://podinfo.fbi.com//status/500

# Check by listing the events associated with the podinfo canary
kubectl describe canary/podinfo

    flagger-loadtester
    https://github.com/fluxcd/flagger/pkgs/container/flagger-loadtester
    
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
kubectl delete all -l app=podinfo -n test
kubectl delete -f hpa.yaml
helm uninstall flagger-loadtester -n test
helm uninstall flagger
kubectl delete ns test
```