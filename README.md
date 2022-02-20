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
    Descompactar el archivo , darle permisos de ejecucion y moverlo hasta /usr/local/bin/
    Descompactar:
    Permisos: chmod +x archivo
    Moverlo: mv kubeseal /usr/local/bin/
    </p>
- ArgoCD cli [https://github.com/argoproj/argo-cd/releases/tag/v2.2.5]
    <p>
    Descargar el archive , darle  permisos de ejecucion y moverlo hasta /usr/local/bin/
    Permisos: chmod +x archivo
    Moverlo: mv  argocd-linux /usr/local/bin/
    Comprobamos que haya quedado instalado: argocd version
    </p>
- kustomize [https://kustomize.io/]
- make (Opcional)

## Instalacion de los componentes del core
Una vez tengamos un cluster k8s funcional iremos instalando las herramientas
necesarias para el funcionamiento de nuestras pruebas

Instalacion del cluster Kubernetes
- kind create cluster --config=kind.yaml

Instalacios del ingress controller
- kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

1 .  Sealed-secrets
Nos permite guardar secretos de forma segura en nuestros repositorios
Instalacion:
- sudo  kubectl apply -f k8s-apps/sealed-secrets/controller.yaml

2 . ArgoCD
sudo kubectl kustomize k8s-apps/argo-cd/overlays/testing \
| sudo kubectl apply -f -

3 . Creamos los secretos para conectarnos al github

4 . Aplicaciones base
 - sudo kubectl
