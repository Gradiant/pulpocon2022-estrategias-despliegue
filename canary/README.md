Canary deployment con funcionalidades de k8s nativas
=================

> Version B es deplegada a un subconjunto de usuarios, entonces procedemos al despliegue completo.

![kubernetes canary deployment](grafana-canary.png)

Un despliegue canary consiste en cambiar gradualmente el tráfico 
de la version A a la versión B. 
Normalmente el tráfico se divide en función del peso, ej. 90% de las peticiones van a la versión A y 10% a la B.

Esta técnica se usa principalmente cuando faltan pruebas o no son fiables o hay poca confianza en la estabilidad de la nueva versión en la plataforma.

Para nuestro caso usamos configuración y reparo de réplicas. 

## Pasos a seguir

1. 10 replicas de la version 1 sirven tráfico
1. Desplegamos 1 replica de version 2 ( 10% del tráfico)
1. Esperamos hasta que la version 2 es estable 
y no errores inesperados
1. Escalamos la version 2 a 10 replicas
1. Esperamos que las instancias estén listas
1. Paramos la version 1

## En práctica

```bash

# Deploy the service one time for almost all deployment strategies
kubectl apply -f ../service.yaml

# Deploy the first application
kubectl apply -f app-v1.yaml

# Test if the deployment was successful
curl pulpocon-app.fbi.com

# To see the deployment in action, open a new terminal and run a watch command.
# It will show you a better view on the progress
watch kubectl get pods

# Then deploy version 2 of the application and scale down version 1 to 9 replicas at same time
kubectl apply -f app-v2.yaml
kubectl scale --replicas=9 deploy pulpocon-app-v1

# Only one pod with the new version should be running.
# You can test if the second deployment was successful
while sleep 0.1; do curl "pulpocon-app.fbi.com"; done

# If you are happy with it, scale up the version 2 to 10 replicas
kubectl scale --replicas=10 deploy pulpocon-app-v2

# Then, when all pods are running, you can safely delete the old deployment
kubectl delete deploy pulpocon-app-v1
```

### Cleanup

```bash
kubectl delete deploy -l app=pulpocon-app
```

**Se puede implementar de forma nativa ajustando el número de réplicas o podemos usar un Nginx como Controlador de Ingress de entrada y se puede configurar la división del trafico más fino a través de anotaciones de Ingress (nginx.ingress.kubernetes.io/canary: "true" and nginx.ingress.kubernetes.io/canary-weight: "10"**

#### Bibliography

- [nginx-ingress] (https://github.com/ContainerSolutions/k8s-deployment-strategies/tree/master/canary/nginx-ingress)

- [Istio] (https://github.com/ContainerSolutions/k8s-deployment-strategies/tree/master/canary/istio)

- [Istio-and-Helm] (https://github.com/etiennetremel/istio-cross-namespace-canary-release-demo)

*If you use Helm to deploy applications, the following repository demonstrate how to make a [canary deployment using Istio and
Helm](https://github.com/etiennetremel/istio-cross-namespace-canary-release-demo).*
