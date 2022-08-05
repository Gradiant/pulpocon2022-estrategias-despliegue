## Pulpocon2022 Estrategias de Despliegue
==========================================

> En kuberentes hay pocas formas diferentes de lanzar una aplicación, 
se debe elegir cuidadosamente la estrategia correcta para hacer que su infraestructura sea resilente.

- [recreate](recreate/): termina la version anterior y se lanza la nueva
- [rollingUpdate](rolling-update/): release una nueva version en modo rolling update, una después de otra
- [blue/green](blue-green/): release una nueva version junto con la versión anterior 
  y luego cambiar el tráfico
- [canary](canary/): release una nueva version a un subconjunto de usuarios, entonces procedemos
  al lanzamiento completo

![deployment strategy decision diagram](decision-diagram.png)

## Getting started

Instalación en local para casa:

- [Instalación infraestructura](local-kind/README.md)


Taller Estrategias de despliegue / Track Devops:

- [Usuarios PulpoCon](https://docs.google.com/spreadsheets/d/1qm4vZoIYYcHK4AwTuHPsjFciUwVgTlFK7ENTguKx0Tk/edit?usp=sharing)
En este fichero tenemos 3 informaciones:
- username: pulpocon-user20
- token: <token-k8s-pulpocon-user20>
- kubeconfig: <file-kube-config-pulpocon-user20>

Las urls de trabajo son las siguientes:
- kubernetes-dashboard: https://kubernetes-dashboard.pulpocon.gradiant.org/#/overview?namespace=pulpocon-user20
- grafana: https://grafana.pulpocon.gradiant.org/d/6CSAiFg4k/pulpocon2022?orgId=1&var-cluster=&var-namespace=pulpocon-user20 entrar con user/pass: admin/pulp0c0n
- pulpocon-app: https://pulpocon-user20.pulpocon.gradiant.org/

Se pueden seguir trabajando las diferentes estrategias por medio del kubernetes-dashboard seleccionando cada usuario su namespace de trabajo ó bien por linea de comandos para ello se necesita la instalación del kubectl, para esta segunda opción recomendamos tener un entorno linux para facilitar la ejecución de los comandos del repo.

Modo gráfico con kubernetes-dashboard:

- Entrar en url del kubernetes-dashboard con el usuario asignado:
  - kubernetes-dashboard: https://kubernetes-dashboard.pulpocon.gradiant.org/#/overview?namespace=pulpocon-user20
  y meter el token <token-k8s-pulpocon-user20> de ese usuario descrito en fichero doc compartido de Usuarios PulpoCon

  y crear el servicio y deployments de las diferentes estrategias mediante el "botón +" del kubernetes-dashboard
  se necesita hacer un diff entre los diferentes deployments versiones y editar el deployment en la sección deployments en los tres puntos del deployment y editar y poner los cambios observados y guardar.
  Una vez acabada cada estrategia eliminar el deployment creado al principio.


Línea de comandos:

- [Instalación kubectl](https://kubernetes.io/docs/tasks/tools/)

<file-kube-config-pulpocon-user20>=kubeconfig-pulpocon-user20 es el fichero del excell_compartidos

export KUBECONFIG=kubeconfig-pulpocon-user20
Una vez configurado el kubeconfig, seguir las instrucciones de cada estrategia con el comando kubectl ...


#### Example graph

Recreate:

![Kubernetes deployment recreate](recreate/grafana-recreate.png)

RollingUpdate:

![Kubernetes deployment ramped](rolling-update/grafana-rolling-update.png)

Blue/Green:

![Kubernetes deployment blue-green](blue-green/grafana-blue-green.png)

Canary:

![Kubernetes deployment canary](canary/grafana-canary.png)

#### Bibliography

Checkout the following resources:
- [GitHub CNCF strategies-deployment](https://github.com/ContainerSolutions/k8s-deployment-strategies)
- [CNCF presentation](https://www.youtube.com/watch?v=1oPhfKye5Pg)
- [CNCF presentation slides](https://www.slideshare.net/EtienneTremel/kubernetes-deployment-strategies-cncf-webinar)
- [Kubernetes deployment strategies](https://container-solutions.com/kubernetes-deployment-strategies/)
- [Six Strategies for Application Deployment](https://thenewstack.io/deployment-strategies/).
- [Canary deployment using Istio and Helm](https://github.com/etiennetremel/istio-cross-namespace-canary-release-demo)
- [Automated rollback of Helm releases based on logs or metrics](https://container-solutions.com/automated-rollback-helm-releases-based-logs-metrics/)