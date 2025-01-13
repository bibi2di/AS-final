## Integración continua y despliegue continuo

Partiendo de un repositorio de código: verificar código -> construir artefactos -> desplegar en un entorno de ejecución.
Con las técnicas de integración continua y despliegue continuo, se pueden automatizar flujos de trabajo.
Github actions -> workflows
```
name: primer workflow
on: 
  push:
    branches:
      - main
      - feature-a

jobs:
  test:
    runs_on: ubuntu-latest
    steps: ...

  build: 
    runs_on: ubuntu_latest
    needs: test
    steps: ...

  hola_mundo:
    runs_on: ubuntu-latest
    permissions:
      security-events: write

    steps:
      - name: Mostrar mensaje
        run: echo "Hola mundo"
      - name: Bajar codigo
        uses: actions/checkout@v4
      - name: Ejecutar codigo
        run: python3 main.py

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: bidane/holamundo
```

#### Técnicas de cloud computing

Tres tipos de servicios en la nube:
* Infrastructure as a Service
* Platform as a Service
* Software as a Service

##### Autoescalado:
Permite crear y eliminar recursos de forma automática en base a umbrales. Existen dos tipos: horizontal (más recursos del mismo tipo) y vertical (aumentar la capacidad del recurso).
En kubernetes:
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: mi-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```
```
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: mi-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  resourcePolicy:
    containerPolicies:
      - containerName: "*"
        minAllowed:
          cpu: 100m
          memory: 50Mi
        maxAllowed:
          cpu: 1
          memory: 500Mi
        controlledResources: ["cpu", "memory"]
```

###### Serverless:
Ejecución de código sin definición infraestructura. P. ej: cloud run. 
