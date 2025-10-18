# Tarea 7. Namespaces, Configuración y Persistencia
# Task Manager Kubernetes - Aplicación de 2 Capas

Aplicación completa de gestión de tareas con backend Node.js + PostgreSQL, desplegada en Kubernetes con arquitectura de 2 capas.
Curso: Docker & Kubernetes - Clase 3
Estudiante: Carlos Roberto Martinez Rivadeneira

## 🏗️ Arquitectura del Sistema

```ascii
┌─────────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────┐        ┌─────────────────────────────────┐ │
│  │   API BACKEND   │        │        POSTGRESQL               │ │
│  │   (Deployment)  │        │       (StatefulSet)             │ │
│  │                 │        │                                 │ │
│  │  ┌─────────────┐│        │  ┌───────────────────────────┐  │ │
│  │  │   Pod-1     ││        │  │         Pod-0             │  │ │
│  │  │  Node.js    │┼─────┼─►│  │       PostgreSQL          │  │ │
│  │  │  Express    ││        │  │                           │  │ │
│  │  └─────────────┘│        │  └───────────────────────────┘  │ │
│  │                 │        │                 ▲               │ │
│  │  ┌─────────────┐│        │                 │               │ │
│  │  │   Pod-2     ││        │          ┌─────────────┐        │ │
│  │  │  Node.js    │┼─────┼─►│          │    PVC      │        │ │
│  │  │  Express    ││        │          │  (1Gi)      │        │ │
│  │  └─────────────┘│        │          └─────────────┘        │ │
│  │                 │        │                                 │ │
│  │         ▲       │        └─────────────────────────────────┘ │
│  │         │       │                                            │
│  │  ┌─────────────┐│                                            │
│  │  │  Service    ││                                            │
│  │  │ (NodePort)  ││                                            │
│  │  └─────────────┘│                                            │
│  │         ▲       │                                            │
│  └─────────┼───────┘                                            │
│            │                                                    │
│    ┌───────────────┐                                            │
│    │   ConfigMap   │   ┌─────────────┐   ┌──────────────────┐   │
│    │  api-config   │   │   ConfigMap │   │     Secret       │   │
│    │               │   │ postgres-   │   │  postgres-secret │   │
│    │ NODE_ENV      │   │   config    │   │                  │   │
│    │ API_PORT      │   │ database    │   │ username         │   │
│    │ CORS_ORIGIN   │   │ host        │   │ password         │   │
│    └───────────────┘   │ port        │   └──────────────────┘   │
│                        └─────────────┘                          │
└─────────────────────────────────────────────────────────────────┘


## 📋 Stack Tecnológico
- Backend: Node.js + Express
- Base de Datos: PostgreSQL 15
- Orquestación: Kubernetes (Minikube)
- Almacenamiento: Persistent Volume Claim (1Gi)
- Configuración: ConfigMaps + Secrets

## 🗂️ Estructura del Proyecto

task-manager-kubernetes/
├── k8s/
│   ├── configs.yaml              # ConfigMaps y Secrets
│   └── postgres-statefulset-fixed.yaml  # PostgreSQL StatefulSet
├── manifests/
│   ├── api-deployment-fixed.yaml # Backend API Deployment
│   └── services.yaml             # Kubernetes Services
├── docs/                         # Documentación
├── screenshots/                  # Capturas de pantalla
└── README.md                     # Este archivo
```

## 🚀 Cómo Ejecutar
### 1. Clonar y Entrar al Directorio
```bash
git clone https://github.com/ccrrmmrr/task-manager-kubernetes.git
cd task-manager-kubernetes
```

### 2. Desplegar la Aplicación
```bash
# Aplicar ConfigMaps y Secrets
kubectl apply -f k8s/configs.yaml

# Desplegar PostgreSQL con StatefulSet
kubectl apply -f k8s/postgres-statefulset-fixed.yaml

# Desplegar Backend API
kubectl apply -f manifests/api-deployment-fixed.yaml
kubectl apply -f manifests/services.yaml
```

### 3. Acceder a la Aplicación
```bash
# Opción 1: Port-forward
kubectl port-forward service/api-service 8080:80

# Opción 2: Minikube service
minikube service api-service --url

# URL: http://localhost:8080
```

## ✅ Cómo Probar
### Verificación de Recursos
```bash
# Ver todos los recursos desplegados
kubectl get all

# Ver pods específicos
```bash
kubectl get pods -l app=api-backend
kubectl get pods -l app=postgres
```
### Ver servicios
```bash
kubectl get services
Probar Endpoints de la API
```
### Health check
```bash
curl http://localhost:8080/api/health
```
### Listar tareas
```bash
curl http://localhost:8080/api/tareas
```
### Crear nueva tarea
```bash
curl -X POST http://localhost:8080/api/tareas \
  -H "Content-Type: application/json" \
  -d '{"titulo": "Mi nueva tarea desde Kubernetes"}'
```
## Escalar la Aplicación

### Escalar backend a 3 réplicas
```bash
kubectl scale deployment api-backend --replicas=3
```
### Verificar el escalado
```bash
kubectl get pods -l app=api-backend
Verificar Base de Datos
```
## 🔧 Componentes Kubernetes
### ConfigMaps
```bash
postgres-config: Configuración de la base de datos

api-config: Variables de entorno del backend
```
### Secrets
```bash
postgres-secret: Credenciales de PostgreSQL
```
### StatefulSet
```bash
postgres: Base de datos con almacenamiento persistente
```
### Deployment
```bash
api-backend: Backend API con 2 réplicas
```
### Services
```bash
api-service: NodePort para acceso externo

postgres-headless: Servicio interno para PostgreSQL
```

## Capturas de Pantalla

### Recursos desplegados
- [kubectl get all](https://github.com/ccrrmmrr/curso-docker-kubernetes-tareas/blob/main/clase7/screenshots/resources.PNG)

### Aplicación funcionando
- [App](https://github.com/ccrrmmrr/curso-docker-kubernetes-tareas/blob/main/clase7/screenshots/aplicacion.PNG)

### Health
- [Health_check](https://github.com/ccrrmmrr/curso-docker-kubernetes-tareas/blob/main/clase7/screenshots/health_check.PNG)
