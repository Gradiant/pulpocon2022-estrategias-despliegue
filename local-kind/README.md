# Local k8s

En caso de no tener acceso al cluster EKS del taller, aquí se muestran unas instrucciones para recrear el entorno en un PC a través de kind (kubernetes in docker).

## Instalación de Kind y Helm

https://kind.sigs.k8s.io/docs/user/quick-start#installation

https://helm.sh/docs/intro/install/


## Creación de cluster local

```
kind create cluster --name pulpocon --config=kind-cluster.yaml
```

## Instalación de Ingress Controller

```
kubectl apply -f resources/ingress-nginx.yaml
```

Espera a que el ingress controller esté listo:
```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

## Instalación de Kubernetes-dashboard

 :warning: Se han modificado la instalación de kubernetes-dashboard para permitir **"saltar"** la autorización y trabajar como **administrador**. Por favor, cuidado de no utilizar esta instalación en clusters de kubernetes que no sean efímeros.

```
kubectl apply -f resources/kubernetes-dashboard.yaml
```

Acceso a través de: [http://kubernetes-dashboard.fbi.com](http://kubernetes-dashboard.fbi.com)

## Instalación de Prometheus y Grafana

Añadimos el repo de prometheus-stack a helm:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Instalamos el helm chart:

```
helm install -n monitoring --create-namespace \
    kube-prometheus-stack prometheus-community/kube-prometheus-stack \
    --set kube-state-metrics.prometheus.monitor.interval=1s \
    --set grafana.ingress.enabled=true \
    --set grafana.ingress.hosts={grafana.fbi.com} \
    --set grafana.defaultDashboardsEnabled=false \
    -f resources/prom-custom-values.yaml \
    --set grafana.ini.min_refresh_interval=1s
```

Espera a que el prometheus-stack esté listo:

```
kubectl wait --namespace monitoring \
  --for=condition=available \
  --all deployments \
  --timeout=120s
```

Instalamos el dashboard, para ello creamos un configmap a partir de la definición del fichero donde está definido el dashboard:

```
kubectl create configmap -n monitoring pulpoconf-dashboard  --from-file=resources/pulpocon2022.json
kubectl label configmap -n monitoring pulpoconf-dashboard grafana_dashboard="1"
```

Accedemos a grafana con user "admin" password "prom-operator" en http://grafana.fbi.com

## Configuración del ingress de acceso de nuestra aplicación pulpocon-app

Instalamos el ingress para que nuestra aplicación "pulpocon-app" ( con las etiquetas "app: pulpocon-app" en el namespace default ) sea accesible mediante la url [http://pulpocon-app.fbi.com](http://pulpocon-app.fbi.com)

```
kubectl apply -f resources/ingress-app.yaml
```

 :warning: **Una vez desplegados los deployments de las sucesivas estrategias tendremos el acceso a través de: [http://pulpocon-app.fbi.com](http://pulpocon-app.fbi.com)**


## Cleanup del cluster local

```
kind delete cluster --name pulpocon
```