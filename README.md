## Pulpocon2022 Estrategias de Despliegue
==================================================

> En kubernetes existen varias maneras de desplegar una aplicación, 
se debe elegir cuidadosamente la estrategia más adecuada para hacer que su infraestructura sea resiliente.

- [recreate](recreate/): termina la version anterior y se lanza la nueva
- [rollingUpdate](rolling-update/): la nueva version se despliega lentamente en modo rolling update, una después de otra reemplazando a la anterior
- [blue/green](blue-green/): la nueva version se despliega junto con la versión anterior y luego se cambia el tráfico
- [canary](canary/): la nueva version se despliega a un subconjunto de usuarios, y gradualmente se incrementa hasta el lanzamiento completo
- A/B release: como un despliegue canary, el subconjunto está definido por **condiciones específcas**
- Shadowing: El tráfico se envía a **ambas versiones** y **la nueva versión no afecta a la respuesta**

<!---

![deployment strategy decision diagram](decision-diagram.png)

-->
![grafico de estrategias de despliegue](estrategias-despliegue-graph.png)

## Getting started

Instalación en local para casa:

- [Instalación infraestructura](local-kind/README.md)

## Taller Estrategias de despliegue / Track Devops:

La infraestructura ya está montada para el evento "PulpoCon" en un cluster de kubernetes (servicio EKS) que tenemos desplegado en AWS (Amazon Web Services)
<!--
- [Usuarios PulpoCon](https://docs.google.com/spreadsheets/d/1qm4vZoIYYcHK4AwTuHPsjFciUwVgTlFK7ENTguKx0Tk/edit?usp=sharing)

En este fichero tenemos 3 informaciones:
    - username: pulpocon-user20
    - token: <"token-k8s-pulpocon-user20">
    - kubeconfig: <"file-kube-config-pulpocon-user20">
-->
- En el taller se le asignarán un número de usuario X a cada usuario/pareja

- [Carpeta Kubeconfigs_PulpoCon](https://drive.google.com/drive/folders/1v-eMXMpb5lJ9sqt8fISqKAmH1ylj9x9S?usp=sharing)

En este directorio tenemos los diferentes ficheros <"kubeconfig-pulpocon-userX"> a utilizar cada usuario/pareja con el usuario asignado, descargarlo a vuestro equipo para ser utilizado como credenciales al kuberentes-dashboard o como kubeconfig en el kubectl ( con variable de entorno en ruta absoluta KUBECONFIG o moviendo ese fichero a ~/.kube/config )

- Las urls de trabajo son las siguientes:
    - kubernetes-dashboard: https://kubernetes-dashboard.pulpocon.gradiant.org/#/discovery y seleccionar el namespace de tu usuario en el desplegable.
    - grafana: https://grafana.pulpocon.gradiant.org/ y entrar con user/pass: admin/pulp0c0n  y seleccionar el namespace de tu usuario en el desplegable.
    - pulpocon-app: https://pulpocon-userX.pulpocon.gradiant.org/ donde X es el usuario asignado.

Se pueden seguir trabajando las diferentes estrategias de **un modo de configuración gráfico** por medio del kubernetes-dashboard seleccionando cada usuario su namespace de trabajo o bien de **un modo consola** por linea de comandos en la que se recomienda tener un entorno linux para facilitar la ejecución de los comandos del repo.

- **Modo gráfico** con kubernetes-dashboard:

    - Entrar en la url del kubernetes-dashboard con el token del usuario asignado:
        - kubernetes-dashboard: https://kubernetes-dashboard.pulpocon.gradiant.org/#/overview?namespace=pulpocon-userX donde X es el usuario asignado y meter el fichero kubeconfig <"kubeconfig-pulpocon-userX"> del usuario asignado para poder entrar.
    - Verificar el servicio de la app en el kubernetes-dashboard en la sección correspondiente con el siguiente contenido: [service.yaml](service.yaml)
    - Seguimos las instrucciones de cada estrategia ( [recreate](recreate/), [rollingUpdate](rolling-update/), [blue/green](blue-green/) y [canary](canary/) )
        - :warning: Una vez acabada cada estrategia eliminar el deployment creado al principio.


- **Modo consola** por línea de comandos:

    - [Instalación kubectl](https://kubernetes.io/docs/tasks/tools/)
    - Utilizar el <"kubeconfig-pulpocon-userX"> que es el fichero del excell_compartido 
    ```bash
    export KUBECONFIG=</ruta/absoulta/"kubeconfig-pulpocon-userX">
    ```
    - Una vez configurado el kubeconfig, verificamos el servicio de la app mediante:
    ```
    kubectl get svc pulpocon-app -o yaml
    ```
    - Seguimos las instrucciones de cada estrategia: con el comando kubectl, en la descripción de cada estrategia [recreate](recreate/), [rollingUpdate](rolling-update/), [blue/green](blue-green/) y [canary](canary/)

### Flagger con nginx ingress controller - despliegue progresivo automático

- [flagger](flagger/): es un operador de entrega/despliegue progresiva para Kubernetes. 
Flagger cambia gradualmente el tráfico a la nueva versión mientras monitoriza las métricas configuradas.
Puede realizar análisis y pruebas automatizados en la nueva versión, decidiendo si propagarlo a todo el clúster o detenerlo si se encuentran problemas.
Flagger aumenta lentamente la carga en la nueva versión mientras mantiene disponible la anterior, lo que garantiza un tiempo de inactividad mínimo.
Puede enviar notificaciones a Slack, Microsoft Teams y otras plataformas para notificarte a ti y a tu equipo sobre los eventos ocurridos.

### Diagarama comparativo entre estrategias

![deployment strategy decision diagram](decision-diagram.png)

### Example graphs

Recreate:

![Kubernetes deployment recreate](recreate/grafana-recreate.png)

RollingUpdate:

![Kubernetes deployment ramped](rolling-update/grafana-rolling-update.png)

Blue/Green:

![Kubernetes deployment blue-green](blue-green/grafana-blue-green.png)

Canary:

![Kubernetes deployment canary](canary/grafana-canary.png)

## Bibliography

Checkout the following resources:
- [GitHub CNCF strategies-deployment](https://github.com/ContainerSolutions/k8s-deployment-strategies)
- [CNCF presentation](https://www.youtube.com/watch?v=1oPhfKye5Pg)
- [CNCF presentation slides](https://www.slideshare.net/EtienneTremel/kubernetes-deployment-strategies-cncf-webinar)
- [Kubernetes deployment strategies](https://container-solutions.com/kubernetes-deployment-strategies/)
- [Six Strategies for Application Deployment](https://thenewstack.io/deployment-strategies/).
- [Canary deployment using Istio and Helm](https://github.com/etiennetremel/istio-cross-namespace-canary-release-demo)
- [Automated rollback of Helm releases based on logs or metrics](https://container-solutions.com/automated-rollback-helm-releases-based-logs-metrics/)
- [Canary deployments with Flagger](https://www.weave.works/blog/kubernetes-deployment-strategies)
- [Flagger NGINX Canary Deployments](https://devopstales.github.io/kubernetes/flagger-nginx-canary-deployments/)
- [Progressively Deliver Releases Using Flagger](https://www.digitalocean.com/community/tutorials/how-to-progressively-deliver-releases-using-flagger-on-digitalocean-kubernetes)
