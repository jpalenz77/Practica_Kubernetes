# Despliegue de Radarr en Kubernetes con Helm

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)](https://helm.sh/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)

Un **chart de Helm** listo para producción para desplegar **Radarr** con **PostgreSQL** en Kubernetes. Esta solución proporciona un despliegue robusto, escalable y persistente adecuado para entornos en la nube.

## 🎯 Características

- **📦 Chart de Helm Completo**: Encapsula todos los recursos necesarios de Kubernetes
- **💾 Persistencia de Datos**: PostgreSQL y la configuración de Radarr utilizan PersistentVolumeClaims (PVCs)
- **🔒 Configuración Segura**: Credenciales sensibles gestionadas con Secrets de Kubernetes
- **🚀 Alta Disponibilidad**: Despliegue multi-réplica para garantizar disponibilidad
- **📈 Auto-escalado**: HorizontalPodAutoscaler ajusta las réplicas según el uso de CPU
- **🌐 Acceso Externo**: Controlador Ingress con soporte para dominio personalizado
- **🏥 Verificaciones de Salud**: Sondas de liveness y readiness aseguran que el tráfico solo vaya a pods saludables
- **📚 Documentación Completa**: Guía de instalación, configuración y gestión

## 🏗️ Arquitectura del Sistema

![Diagrama de Arquitectura](https://github.com/KeepCodingCloudDevops12/Jose_M_Palenzuela_Kubernetes/blob/main/Diagrama.png)

## 🛠️ Requisitos Previos

Antes de comenzar, asegúrate de tener las siguientes herramientas instaladas:

- **Minikube**: Clúster de Kubernetes de dos nodos para desarrollo local
- **kubectl**: Herramienta de línea de comandos de Kubernetes
- **Helm**: Gestor de paquetes de Kubernetes
- **Docker**
- **Git**

### Instalación de Herramientas

```bash
# Iniciar Minikube
minikube start --nodes 2

# Instalar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Instalar Helm
curl -fsSL https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz | tar -xz
sudo mv linux-amd64/helm /usr/local/bin/helm

# Habilitar addons necesarios
minikube addons enable metrics-server
minikube addons enable ingress
```

## 📦 Instalación

### Clonar el Repositorio

```bash
git clone https://github.com/jpalenz77/Practica_Kubernetes
cd Practica_Kubernetes
```

### Despliegue Básico

```bash
# Instalar con contraseña personalizada de base de datos
helm install k8s-practica . --set database.password=TuContraseñaSegura
```

### Despliegue con Configuración Personalizada

```bash
helm install k8s-practica . \
  --set replicaCount=3 \
  --set database.storage=10Gi \
  --set persistence.downloads.storage=100Gi \
  --set database.password=MiContraseñaSegura123
```

## 🌐 Acceso a la Aplicación

### Opción 1: Port-Forward (Recomendado)

```bash
kubectl port-forward svc/k8s-practica-radarr-chart 7878:7878
```

Luego abre tu navegador en: `http://localhost:7878`

**Nota**: Este es el método más confiable para acceder a la aplicación en entornos de desarrollo local.

### Opción 2: Ingress (Problemático en Minikube)

**⚠️ Advertencia**: El acceso vía Ingress presenta problemas en Minikube debido a la compleja estructura de red virtualizada y posibles conflictos con cookies/sesiones. Se recomienda usar port-forward para desarrollo local.

Si aún deseas intentar con Ingress, actualiza `values.yaml`:

```yaml
ingress:
  enabled: true
  host: "radarr.practica.local"
```

Luego:

```bash
# Actualizar el chart
helm upgrade k8s-practica . --set database.password=TuContraseñaSegura

# Iniciar túnel de Minikube (en una terminal separada)
minikube tunnel

# Añadir al archivo hosts
echo "127.0.0.1 radarr.practica.local" | sudo tee -a /etc/hosts

# Intentar acceder en el navegador (puede fallar)
# http://radarr.practica.local
```

**Problemas conocidos con Ingress en Minikube**:
- La red virtualizada de Minikube puede causar problemas de enrutamiento
- Conflictos con la gestión de cookies y sesiones
- Latencia adicional que puede afectar la funcionalidad de la aplicación
- En entornos de producción (clusters reales), Ingress funciona correctamente

## ⚙️ Configuración

### Parámetros Principales

| Parámetro | Descripción | Valor por Defecto |
|---|---|---|
| `replicaCount` | Número de réplicas de Radarr | `2` |
| `database.storage` | Tamaño del volumen de PostgreSQL | `5Gi` |
| `persistence.config.storage` | Tamaño del volumen de configuración de Radarr | `10Gi` |
| `persistence.downloads.storage` | Tamaño del volumen de descargas de Radarr | `50Gi` |
| `autoscaling.enabled` | Habilitar auto-escalado | `true` |
| `autoscaling.targetCPUUtilizationPercentage` | Umbral de CPU para escalar | `70` |
| `ingress.enabled` | Habilitar Ingress | `false` |
| `ingress.host` | Dominio de la aplicación | `radarr.practica.local` |

### Personalización

Personaliza tu despliegue editando `values.yaml` o usando banderas `--set`:

```bash
helm install k8s-practica . \
  --set replicaCount=3 \
  --set database.storage=10Gi \
  --set persistence.downloads.storage=100Gi \
  --set database.password=MiContraseñaSegura123
```

## 📁 Estructura del Proyecto

```
radarr-chart/
├── Chart.yaml                           # Metadatos del chart
├── values.yaml                          # Configuración por defecto
├── templates/
│   ├── _helpers.tpl                     # Funciones auxiliares de plantillas
│   ├── hpa.yaml                         # HorizontalPodAutoscaler
│   ├── postgres-statefulset.yaml        # StatefulSet de PostgreSQL
│   ├── postgres-service.yaml            # Service de PostgreSQL
│   ├── postgres-initdb-configmap.yaml   # ConfigMap para inicialización de bases de datos
│   ├── radarr-deployment.yaml           # Deployment de Radarr
│   ├── radarr-service.yaml              # Service de Radarr
│   ├── radarr-pvc.yaml                  # PersistentVolumeClaims de Radarr
│   ├── radarr-ingress.yaml              # Configuración de Ingress
│   ├── radarr-config-configmap.yaml     # ConfigMap con config.xml de Radarr
│   └── secrets.yaml                     # Secrets de Kubernetes
└── README.md                            # Esta documentación
```

## 🗂️ Arquitectura de Base de Datos

Radarr requiere **dos bases de datos separadas** en PostgreSQL:

- **`radarr-main`**: Almacena toda la configuración e historial
- **`radarr-log`**: Almacena eventos que producen entradas de log

### Configuración PostgreSQL

La configuración de PostgreSQL se gestiona automáticamente mediante:

1. **ConfigMap de inicialización** (`postgres-initdb-configmap.yaml`): Crea las bases de datos automáticamente
2. **ConfigMap de configuración Radarr** (`radarr-config-configmap.yaml`): Configura Radarr para usar PostgreSQL
3. **InitContainers**: Aseguran el orden correcto de inicialización

## 🔄 Gestión del Ciclo de Vida

### Verificar Estado

```bash
# Verificar todos los recursos
kubectl get all -l app.kubernetes.io/instance=k8s-practica

# Ver logs de componentes específicos
kubectl logs deployment/k8s-practica-radarr-chart -c radarr
kubectl logs postgres-statefulset-0

# Verificar estado del auto-escalado
kubectl get hpa
```

### Actualizar Despliegue

```bash
helm upgrade k8s-practica . --set database.password=NuevaContraseña
```

### Escalar Manualmente

```bash
kubectl scale deployment k8s-practica-radarr-chart --replicas=4
```

### Desinstalar

```bash
helm uninstall k8s-practica
```

## 🔍 Solución de Problemas

### Problemas Comunes

**Pod atascado en estado Pending:**
```bash
kubectl describe pod <nombre-pod>
# Verificar problemas de clase de almacenamiento o restricciones de recursos
```

**No se puede acceder vía Ingress:**
```bash
# Verificar que el controlador ingress esté funcionando
kubectl get pods -n ingress-nginx
# Verificar configuración de ingress
kubectl describe ingress k8s-practica-radarr-chart
```

**Problemas de conexión a base de datos:**
```bash
# Verificar logs de PostgreSQL
kubectl logs postgres-statefulset-0

# Verificar que las bases de datos se crearon correctamente
kubectl exec -it postgres-statefulset-0 -- psql -U radarr -d postgres -c "\l"

# Verificar configuración de Radarr
kubectl exec -it <radarr-pod-name> -- cat /config/config.xml
```

**Error "failed to load tags from api":**
- Este error indica problemas de conectividad con la base de datos
- Verificar que PostgreSQL esté funcionando y que las bases de datos existan
- Comprobar que el archivo `config.xml` contiene la configuración PostgreSQL correcta

### Problemas Específicos de PostgreSQL

**Incompatibilidad de versiones:**
```bash
# Si aparece error de versiones incompatibles, limpiar datos:
helm uninstall k8s-practica
kubectl delete pvc postgres-data-postgres-statefulset-0 --force
# Reinstalar desde cero
```

**Verificar configuración de bases de datos:**
```bash
# Conectar a PostgreSQL y verificar bases de datos
kubectl exec -it postgres-statefulset-0 -- psql -U radarr -d radarr-main -c "SELECT 1;"
kubectl exec -it postgres-statefulset-0 -- psql -U radarr -d radarr-log -c "SELECT 1;"
```

### Monitoreo

```bash
# Observar estado de pods
kubectl get pods -w

# Monitorear uso de recursos
kubectl top pods

# Verificar eventos del cluster
kubectl get events --sort-by=.metadata.creationTimestamp | tail -20

# Verificar configuración específica de Radarr
kubectl logs deployment/k8s-practica-radarr-chart -c setup-config
```

### Logs Útiles para Debugging

```bash
# Logs del InitContainer de configuración
kubectl logs <radarr-pod-name> -c setup-config

# Logs del InitContainer de espera de PostgreSQL
kubectl logs <radarr-pod-name> -c wait-for-postgres

# Logs principales de Radarr
kubectl logs <radarr-pod-name> -c radarr --tail=50

# Logs de PostgreSQL
kubectl logs postgres-statefulset-0 -f
```

## 🔧 Notas Técnicas

### Configuración de Radarr

Radarr se configura mediante un archivo `config.xml` que se genera automáticamente con la configuración PostgreSQL correcta. La configuración incluye:

```xml
<PostgresUser>radarr</PostgresUser>
<PostgresPassword>TuContraseñaSegura</PostgresPassword>
<PostgresPort>5432</PostgresPort>
<PostgresHost>postgres-service</PostgresHost>
<PostgresMainDb>radarr-main</PostgresMainDb>
<PostgresLogDb>radarr-log</PostgresLogDb>
```

### InitContainers

El deployment utiliza dos InitContainers:

1. **setup-config**: Configura el archivo `config.xml` con la configuración PostgreSQL
2. **wait-for-postgres**: Espera a que PostgreSQL esté disponible antes de iniciar Radarr

### Persistencia

- **PostgreSQL**: Usa StatefulSet con volumeClaimTemplates para persistencia automática
- **Radarr**: Usa PersistentVolumeClaims para configuración y descargas

---

⭐ ¡Si este proyecto te gusta, por favor dale una estrella!