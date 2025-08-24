# helm_delivery-app

Repositorio Helm para *delivery-app*, una aplicación de gestión de pedidos que incluye la aplicación principal y una base de datos como dependencia.

## NOTA: Los archivos .tgz se encuentran en la rama gh-pages.

## Contenido

En la carpeta _dist se encuentran los paquetes (.tgz) de las distintas versiones del chart y un index.yaml generado automáticamente para que Helm pueda reconocer el repositorio.  
El index.yaml en el directorio raíz es el archivo que GitHub Pages sirve públicamente en:

https://andresosa21.github.io/helm_delivery-app/index.yaml

## Instalación manual con Helm

1. *Agregar el repositorio Helm:*

   ```bash
   helm repo add delivery-app https://andresosa21.github.io/helm_delivery-app
   helm repo update

2. *Verificar versiones disponibles*

helm search repo delivery-app

3. *Instalar la aplicación*

helm install my-delivery delivery-app/delivery-app --version 0.1.4

4. *Actualizar la release a una nueva versión*

helm upgrade my-delivery delivery-app/delivery-app --version 0.1.5

5. *Desinstalar*

helm uninstall my-delivery

## Sincronización con ArgoCD

1. *Registrar el repo en ArgoCD*

argocd repo add https://andresosa21.github.io/helm_delivery-app --type helm

2. *Definir una aplicación en ArgoCD (ejemplo Application manifest)*

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: delivery-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://andresosa21.github.io/helm_delivery-app
    chart: delivery-app
    targetRevision: 0.1.4   # versión del chart a sincronizar
  destination:
    server: https://kubernetes.default.svc
    namespace: delivery
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

en donde:

targetRevision: define la versión del chart a usar.

Con automated: true, ArgoCD sincroniza automáticamente la versión especificada del chart con el cluster.

Si se sube un nuevo .tgz al repo y se actualiza el index.yaml, basta con cambiar targetRevision en el Application para que ArgoCD lo despliegue.
