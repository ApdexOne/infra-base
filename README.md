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
    Descompactar el archivo , darle permisos de ejecucion y moverlo hasta
    /usr/local/bin/ <br>
    Descompactar:<br>
    Permisos: chmod +x archivo<br>
    Moverlo: mv kubeseal /usr/local/bin/<br>
    </p>
- ArgoCD cli [https://github.com/argoproj/argo-cd/releases/tag/v2.2.5]
    <p>
    Descargar el archivo , darle  permisos de ejecuci&oacute;n y moverlo hasta
    /usr/local/bin/<br>
    Permisos: chmod +x argocd-linux-amd64<br>
    Moverlo: mv argocd-linux-amd64 /usr/local/bin/argocd<br>
    Comprobamos que haya quedado instalado: argocd version<br>
    </p>
- kustomize [https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/]
- make (Opcional)

## Instalaci&oacute;n de los componentes del core
Una vez tengamos un cluster k8s funcional iremos instalando las herramientas
necesarias para el funcionamiento de nuestras pruebas

Instalaci&oacute;n del cluster Kubernetes
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
sudo kubectl create ns services
helm install --dependency-update gitea -f infra/argo-combined-demo/gitea/values.yaml infra/argo-combined-demo/gitea
```

Luego de instalado podemos acceder desde [http://git.127.0.0.1.nip.io/].
Creamos una organizacion nombrada ApdexOne y dentro de esta un proyecto
llamada platform.

Adicionamos a Gitea como remoto de nuestro repositorio y subimos el codigo
de la plataforma

```
git remote add gitea http://git.127.0.0.1.nip.io/ApdexOne/platform.git
git add .
git commit -am "Initial commit"
git push -u gitea master
```

Una vez que tengamos en una replica del repo de Github en nuestro repo local
continuamos con los demas pasos de la instalacion.


1 .  Sealed-secrets <br>
Nos permite guardar secrets(accesos a git, keys, passwords) de forma segura en
nuestros repositorios

Instalacion:<br>

```
sudo  kubectl apply -f infra/k8s-apps/sealed-secrets/controller.yaml
```

Luego de la instalacion crearemos secrets para los accesos al repositorio y al
docker registry.

```
echo "apiVersion: v1
kind: Secret
metadata:
  name: git-access
  namespace: workflows
type: Opaque
data:
  token: $(echo -n token_gitea | base64)
  user: $(echo -n ApdexOne | base64)
  email: $(echo -n user_email | base64)" \
    | kubeseal --format yaml \
    | tee argo-workflows/overlays/workflows/githubcred.yaml
```

```
echo "apiVersion: v1
kind: Secret
metadata:
  name: github-access
  namespace: argo-events
type: Opaque
data:
  token: $(echo -n token_gitea | base64)" \
    | kubeseal --format yaml \
    | tee argo-events/overlays/production/githubcred.yaml
```

```
kubectl --namespace workflows \
    create secret \
    docker-registry regcred \
    --docker-server=$REGISTRY_SERVER \
    --docker-username=$REGISTRY_USER \
    --docker-password=$REGISTRY_PASS \
    --docker-email=$REGISTRY_EMAIL \
    --output json \
    --dry-run=client \
    | kubeseal --format yaml \
    | tee argo-workflows/overlays/production/regcred.yaml
```

2 . ArgoCD

```
sudo kubectl kustomize infra/argo-cd/overlays/testing | sudo kubectl apply -f -
```

3 . Creamos los secretos para conectarnos al github

4 . Aplicaciones base

```
sudo kubectl apply -f infra/argo-combined-demo/project.yaml
```

```
sudo kubectl apply -f infra/argo-combined-demo/apps.yaml
```
