## Pulpocon2022 Estrategias de Despliegue
==========================================

> En kuberentes hay pocas formas diferentes de lanzar una aplicación, 
se debe elegir cuidadosamente la estrategia correcta para hacer que su infraestructura sea resilente.

- [recreate](Recreate/): termina la version anterior y se lanza la nueva
- [rollingUpdate](RollingUpdate/): release una nueva version en modo rolling update modo, una después de otra
- [blue/green](BlueGreen/): release una nueva version junto la versión anterior 
  y luego cambiar el tráfico
- [canary](Canary/): release una nueva version a un subconjunto de usuarios, entonces procedemos
  al lanzamiento completo

![deployment strategy decision diagram](decision-diagram.png)

## Getting started

Instalación en local:

![Kubernetes deployment recreate](local-kind/README.md)


#### Example graph

Recreate:

![Kubernetes deployment recreate](Recreate/grafana-recreate.png)

RollingUpdate:

![Kubernetes deployment ramped](RollingUpdate/grafana-rollingUpdate.png)

Blue/Green:

![Kubernetes deployment blue-green](BlueGreen/grafana-blue-green.png)

Canary:

![Kubernetes deployment canary](Canary/grafana-canary.png)

#### Bibliography

Checkout the following resources:
- [GitHub CNCF strategies-deployment](https://github.com/ContainerSolutions/k8s-deployment-strategies)
- [CNCF presentation](https://www.youtube.com/watch?v=1oPhfKye5Pg)
- [CNCF presentation slides](https://www.slideshare.net/EtienneTremel/kubernetes-deployment-strategies-cncf-webinar)
- [Kubernetes deployment strategies](https://container-solutions.com/kubernetes-deployment-strategies/)
- [Six Strategies for Application Deployment](https://thenewstack.io/deployment-strategies/).
- [Canary deployment using Istio and Helm](https://github.com/etiennetremel/istio-cross-namespace-canary-release-demo)
- [Automated rollback of Helm releases based on logs or metrics](https://container-solutions.com/automated-rollback-helm-releases-based-logs-metrics/)