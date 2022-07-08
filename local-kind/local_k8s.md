# Local k8s

En caso de no tener acceso al cluster EKS del taller, aquí se muestran unas instrucciones para recrear el entorno en un PC a través de kind (kubernetes in docker).

## Instalación de Kind

https://kind.sigs.k8s.io/docs/user/quick-start#installation


## Creación de cluster local

```
kind create cluster --config=kind-cluster.yaml
```

## Instalación de Ingress Controller

```
kubectl apply -f resources/ingress-nginx.yaml
```

## Despliegue de la Aplicación "podinfo"

```
kubectl apply -f resources/podinfo_base.yaml
```

Acceso a través de: [http://podinfo.fbi.com](http://podinfo.fbi.com)

## Instalación de Kubernetes-dashboard

Se han modificado la instalación de kubernetes-dashboard para permitir saltar la autorización y trabajar como administrador. Por favor, no utilizar esta instalación en clusters de kubernetes que no sean efímeros.

```
kubectl apply -f resources/kubernetes-dashboard.yaml
```

Acceso a través de: [http://kubernetes-dashboard.fbi.com](http://kubernetes-dashboard.fbi.com)

