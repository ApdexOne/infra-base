## Instalacion del cluster kubernetes

Como plataforma kubernetes se puede utilizar cualquiera de las siguientes:
- kind
- k3s
- minikube
- RancherDesktop

Recomendamos utilizar esta ultima ya que puede ser ejecutada en diferenetes
SO ademas de su intuitiva interfaz para la administracion

Es necesario verificar que luego de instalar la distribucion de k8s se este
ejecutando algun Ingress Controller (Nginx, Ambassador, Traefik, Haproxy)

## Pre-requisitos
Para el manejo del cluster y la instalacion/administracion de los componentes
se precisa instalar las siguientes herramientas:
- kubectl [https://kubernetes.io/docs/tasks/tools/]
- helm [https://helm.sh/]
- kubeseal [https://github.com/bitnami-labs/sealed-secrets]
 - Instalacion: Descargamos el releases desde [https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.17.3/kubeseal-0.17.3-linux-amd64.tar.gz]
    <p>
    Descompactar el archivo , darle permisos de ejecucion y moverlo hasta /usr/local/bin/ <br>
    Descompactar:<br>
    Permisos: chmod +x archivo<br>
    Moverlo: mv kubeseal /usr/local/bin/<br>
    </p>
- ArgoCD cli [https://github.com/argoproj/argo-cd/releases/tag/v2.2.5]
    <p>
    Descargar el archivo , darle  permisos de ejecucion y moverlo hasta /usr/local/bin/<br>
    Permisos: chmod +x argocd-linux-amd64<br>
    Moverlo: mv argocd-linux-amd64 /usr/local/bin/argocd<br>
    Comprobamos que haya quedado instalado: argocd version<br>
    </p>
- kustomize [https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/]
- make (Opcional)

## Instalacion de los componentes del core
Una vez tengamos un cluster k8s funcional iremos instalando las herramientas
necesarias para el funcionamiento de nuestras pruebas

Instalacion del cluster Kubernetes
Desde la raiz del repositorio: <br>
<br>
<code>kind create cluster --config=infra/cluster/kind.yaml</code>

Instalacios del ingress controller
<br>
<code>kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml</code>

1 .  Sealed-secrets <br>
Nos permite guardar secretos de forma segura en nuestros repositorios
Instalacion:<br>
<code>sudo  kubectl apply -f infra/k8s-apps/sealed-secrets/controller.yaml</code>

2 . ArgoCD <br>

<code>
sudo kubectl kustomize infra/k8s-apps/argo-cd/overlays/testing \
| sudo kubectl apply -f -
<br>
</code>

3 . Creamos los secretos para conectarnos al github

4 . Aplicaciones base
 <br>
 <code>
 sudo kubectl apply -f infra/argo-combined-demo/project.yaml
 </code>
 <br>
 <br>
 <code>
 sudo kubectl apply -f infra/argo-combined-demo/apps.yaml
 </code>
