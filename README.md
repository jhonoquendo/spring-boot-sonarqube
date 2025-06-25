# Spring Boot SonarQube


## Comandos usados en este laboratorio


### Comando para crear cluster en EKS - AWS
```
eksctl create cluster \
  --name jhon-devops-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3 \
  --managed
```

### Verificar creacion de cluster luego de 10 min
```
aws eks --region us-east-1 update-kubeconfig --name jhon-devops-cluster
kubectl get nodes
```

### Se crea namespace
```
kubectl create namespace actions-runner-system
```

## Se instala los CRDs de cert-manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.crds.yaml
```

### Se Agrega el repo de Helm y actualizar
```
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

### Instalar cert-manager
```
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4
```


### Espera 1–2 minutos, y verifica:

```
kubectl get pods -n cert-manager
```

### Se genera nuevo token
```
Selecciona estos scopes (permisos mínimos):

repo (para acceder a los repos de la org)

admin:org (solo si registrarás runners a nivel de organización)

read:org

workflow

admin:public_key

admin:repo_hook
```

### Se Instala el CRD y el controller
```
helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller

helm repo update

helm install runner-controller actions-runner-controller/actions-runner-controller \
  --namespace actions-runner-system \
  --set githubWebhookServer.enabled=false \
  --set authSecret.create=true \
  --set authSecret.github_token=<YOUR_GITHUB_TOKEN> \
  --set githubOrganization=jhonoquendo
```

### Creacion de runner

```
kubectl apply -f runner-deployment.yaml
```

### Verifica que se esté creando el runner como pod
```
kubectl get pods -n actions-runner-system
```

### Prueba un workflow desde GitHub

- Crea un archivo .github/workflows/test-runner.yml con este contenido:

```
name: Test Runner on Kubernetes

on: [push]

jobs:
  test:
    runs-on: [self-hosted]
    steps:
      - uses: actions/checkout@v3
      - name: Echo desde el pod
        run: echo "¡Hola desde Kubernetes, Jhon!"

```

### Crear el repositorio con GitHub CLI
```
gh repo create spring-boot-sonarqube \
  --public \
  --confirm \
  --description "Repo de prueba para GitHub Actions en EKS con self-hosted runner"
```

### Eliminar cluster para evitar costos:

```
eksctl delete cluster --name jhon-devops-cluster
```

🔍 ¿Qué puedes revisar en la consola de AWS por si acaso?
Aunque eksctl elimina la mayoría de recursos, es buena práctica revisar que no quedaron huérfanos:

Servicio	Qué revisar	Acción
EC2	Volúmenes EBS y snapshots	Eliminar si ya no se usan
VPC	VPC personalizadas del clúster	Verifica y elimina si no hay más recursos
CloudWatch Logs	Registros generados por los pods	Opcional: limpiar
IAM	Roles creados para el EKS	Verifica si no están en uso