## Instalaci&oacute;n del cluster kubernetes

Para la instalaci&oacute;n del cluster Kubernetes podemos utilizar cualquiera
de las siguientes herramientas:
<br>
- kind
- k3s
- minikube
- RancherDesktop
<br>
Recomendamos utilizar esta &uacute;ltima ya que puede ser ejecutada en diferenetes
SO ademas de su intuitiva interfaz para la administraci&oacute;n

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

```
kind create cluster --config=infra/cluster/kind.yaml
```

Instalacion del ingress controller
```
sudo kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Luego de la instalacion del cluster instalaremos un software (Gitea),lo
utilizaremos como un repo paralelo al github real para poder realizar pruebas
de integracion entre los componentes de la CI/CD.

Ejecutaremos:

```
helm install --dependency-update gitea -f infra/argo-combined-demo/gitea/values.yaml infra/argo-combined-demo/gitea
```

Luego de instalado podemos acceder desde [http://git.127.0.0.1.nip.io/]


1 .  Sealed-secrets <br>
Nos permite guardar secretos de forma segura en nuestros repositorios
Instalacion:<br>

```
sudo  kubectl apply -f infra/k8s-apps/sealed-secrets/controller.yaml
```

2 . ArgoCD

```
sudo kubectl kustomize infra/k8s-apps/argo-cd/overlays/testing | sudo kubectl apply -f -
```

3 . Creamos los secretos para conectarnos al github

4 . Aplicaciones base

```
sudo kubectl apply -f infra/argo-combined-demo/project.yaml
```

```
sudo kubectl apply -f infra/argo-combined-demo/apps.yaml
```
