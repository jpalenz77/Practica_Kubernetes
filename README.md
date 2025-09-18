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

## 🛠️ Prerrequisitos

Antes de comenzar, asegúrate de tener las siguientes herramientas instaladas:

- **WSL 2**: Windows Subsystem for Linux (se recomienda Ubuntu)
- **Minikube**: Clúster de Kubernetes de un solo nodo para desarrollo local
- **kubectl**: Herramienta de línea de comandos de Kubernetes
- **Helm**: Gestor de paquetes de Kubernetes

### Instalación Rápida (WSL)

```bash
# Iniciar Minikube
minikube start --driver=docker

# Instalar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Instalar Helm
curl -fsSL https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz | tar -xz
sudo mv linux-amd64/helm /usr/local/bin/helm
```

## 🚀 Inicio Rápido

### 1. Habilitar Complementos de Minikube

```bash
minikube addons enable default-storageclass
minikube addons enable ingress
```

### 2. Clonar y Desplegar

```bash
git clone <URL_DE_TU_REPOSITORIO>
cd radarr-chart

# Instalar con contraseña personalizada de base de datos
helm install radarr-release . --set database.password=TuContraseñaSegura
```

### 3. Configurar Acceso Externo

En una nueva terminal, iniciar el túnel:
```bash
minikube tunnel
```

Agregar al archivo hosts (`C:\Windows\System32\drivers\etc\hosts` en Windows):
```
127.0.0.1 radarr.minikube.local
```

### 4. Acceder a la Aplicación

Visita: **http://radarr.minikube.local**

## ⚙️ Configuración

Personaliza tu despliegue editando `values.yaml` o usando banderas `--set`:

| Parámetro | Descripción | Valor por Defecto |
|-----------|-------------|-------------------|
| `replicaCount` | Número de réplicas de Radarr | `2` |
| `database.storage` | Tamaño del volumen de PostgreSQL | `5Gi` |
| `persistence.config.storage` | Tamaño del volumen de configuración de Radarr | `10Gi` |
| `persistence.downloads.storage` | Tamaño del volumen de descargas de Radarr | `50Gi` |
| `autoscaling.enabled` | Habilitar auto-escalado | `true` |
| `autoscaling.targetCPUUtilizationPercentage` | Umbral de CPU para escalar | `70` |
| `ingress.enabled` | Habilitar Ingress | `true` |
| `ingress.host` | Dominio de la aplicación | `radarr.minikube.local` |

### Ejemplo de Instalación Personalizada

```bash
helm install radarr-release . \
  --set replicaCount=3 \
  --set database.storage=10Gi \
  --set persistence.downloads.storage=100Gi \
  --set database.password=MiContraseñaSegura123
```

## 📊 Comandos de Gestión

### Verificar Despliegue

```bash
# Verificar todos los recursos
kubectl get all -l app.kubernetes.io/instance=radarr-release

# Ver logs
kubectl logs <nombre-pod-radarr>

# Verificar estado del auto-escalado
kubectl get hpa
```

### Escalado Manual

```bash
kubectl scale deployment radarr-release-radarr-chart --replicas=4
```

### Actualizar Configuración

```bash
helm upgrade radarr-release . --set database.password=NuevaContraseña
```

### Desinstalar

```bash
helm uninstall radarr-release
```

## 📁 Estructura del Proyecto

```
radarr-chart/
├── Chart.yaml                    # Metadatos del chart
├── values.yaml                   # Configuración por defecto
├── templates/
│   ├── _helpers.tpl             # Funciones auxiliares de plantillas
│   ├── hpa.yaml                 # HorizontalPodAutoscaler
│   ├── postgres-statefulset.yaml # StatefulSet de PostgreSQL
│   ├── postgres-service.yaml    # Service de PostgreSQL
│   ├── radarr-deployment.yaml   # Deployment de Radarr
│   ├── radarr-service.yaml      # Service de Radarr
│   ├── radarr-pvc.yaml          # PersistentVolumeClaims de Radarr
│   ├── radarr-ingress.yaml      # Configuración de Ingress
│   └── secrets.yaml             # Secrets de Kubernetes
└── README.md                     # Esta documentación
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
kubectl describe ingress radarr-release-radarr-chart
```

**Problemas de conexión a base de datos:**
```bash
# Verificar logs de PostgreSQL
kubectl logs <nombre-pod-postgres>

# Verificar secrets
kubectl get secrets radarr-release-radarr-chart -o yaml
```

### Monitoreo

```bash
# Observar estado de pods
kubectl get pods -w

# Monitorear uso de recursos
kubectl top pods

# Verificar eventos
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 🤝 Contribuir

1. Haz fork del repositorio
2. Crea una rama de características (`git checkout -b feature/caracteristica-increible`)
3. Confirma tus cambios (`git commit -m 'Agregar característica increíble'`)
4. Empuja a la rama (`git push origin feature/caracteristica-increible`)
5. Abre un Pull Request

## 📝 Licencia

Este proyecto está licenciado bajo la Licencia MIT - consulta el archivo [LICENSE](LICENSE) para más detalles.

## 🙏 Agradecimientos

- [Radarr](https://radarr.video/) - El gestor de colección de películas
- [PostgreSQL](https://www.postgresql.org/) - La base de datos de código abierto más avanzada del mundo
- [Kubernetes](https://kubernetes.io/) - Plataforma de orquestación de contenedores
- [Helm](https://helm.sh/) - El gestor de paquetes para Kubernetes

---

⭐ ¡Si este proyecto te ayudó, por favor dale una estrella!
