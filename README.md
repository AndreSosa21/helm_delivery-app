# helm_delivery-app

## hecho por: Andrea Sosa, David Medina, Samuel Rodriguez

Repositorio Helm para *delivery-app*, una aplicación de gestión de pedidos que incluye la aplicación principal y una base de datos como dependencia.

## NOTA: Los archivos .tgz se encuentran en la rama gh-pages.

## Antes de instalar el chart asegúrate de tener:

- Un cluster **Kubernetes** en funcionamiento (v1.22+ recomendado).
- **Helm** instalado en tu máquina local (v3.8+).
- (Opcional) **ArgoCD** instalado en el cluster para despliegues GitOps.
- Un **Ingress Controller** instalado (por ejemplo, NGINX Ingress Controller) si deseas exponer la aplicación vía HTTP/HTTPS.

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

   ```bash
   helm install my-delivery delivery-app/delivery-app --version 0.1.4

4. *Actualizar la release a una nueva versión*

   ```bash
   helm upgrade my-delivery delivery-app/delivery-app --version 0.1.5

5. *Desinstalar*

   ```bash
   helm uninstall my-delivery

## Configuración de Ingress

El chart incluye soporte para Ingress, el cual se puede habilitar y configurar mediante los valores en values.yaml:

      ingress:
        enabled: true
        className: nginx
        annotations:
          nginx.ingress.kubernetes.io/rewrite-target: /
        hosts:
            paths:
              - path: /
                pathType: Prefix
        tls: []

URL ingress frontend: 192.212.145.158/delivery/users
URL ingress backend: 192.212.145.158/delivery/api

### NOTA: La API solo se conecta con /users.

- `ingress.enabled`: habilita o deshabilita el Ingress.
- `ingress.className`: define el controlador de ingress (por ejemplo, nginx).
- `ingress.hosts`: lista de hosts (dominios o subdominios) que apuntarán al servicio de la app.
- `ingress.tls`: configuración TLS para HTTPS (requiere un certificado válido, ej. con cert-manager).

Para instalar con un dominio personalizado, puedes usar:

      ```bash
      helm install my-delivery delivery-app/delivery-app \
        --set ingress.enabled=true \
        --set ingress.hosts[0].host=delivery.example.com \
        --set ingress.hosts[0].paths[0].path=/

Una vez aplicado, se accederá a la aplicación en el dominio definido.

## Sincronización con ArgoCD

1. *Registrar el repo en ArgoCD*

   ```bash
   argocd repo add https://andresosa21.github.io/helm_delivery-app --type helm

2. *Definir una aplicación en ArgoCD (ejemplo Application manifest)*

   ```bash
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

`targetRevision`: define la versión del chart a usar.

Con automated: true, ArgoCD sincroniza automáticamente la versión especificada del chart con el cluster.

Si se sube un nuevo .tgz al repo y se actualiza el index.yaml, basta con cambiar targetRevision en el Application para que ArgoCD lo despliegue.
