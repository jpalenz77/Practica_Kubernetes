# Práctica: Despliegue de Radarr en Kubernetes con Helm

Este repositorio contiene un **chart de Helm** para desplegar la aplicación **Radarr** junto con una base de datos **PostgreSQL** en un clúster de Kubernetes.  
La solución está diseñada para ser **robusta, escalable y persistente**, cumpliendo con los objetivos de la práctica de despliegue de aplicaciones en la nube.

## 🎯 Objetivos de la Práctica
- **Chart de Helm**: encapsula todos los recursos de Kubernetes necesarios para el despliegue.  
- **Persistencia de Datos**: PostgreSQL y la configuración de Radarr utilizan *PersistentVolumeClaim (PVC)* para asegurar que los datos no se pierdan al reiniciar los pods.  
- **Configuración Segura**: las credenciales sensibles se gestionan con *Secrets* de Kubernetes.  
- **Alta Disponibilidad (HA)**: despliegue con un mínimo de réplicas para garantizar disponibilidad.  
- **Auto-escalado**: *HorizontalPodAutoscaler (HPA)* ajusta el número de réplicas según el uso de CPU.  
- **Exposición Externa**: acceso desde fuera del clúster mediante *Ingress* con nombre de dominio.  
- **Resiliencia**: *liveness* y *readiness probes* aseguran que solo los pods sanos reciban tráfico.  
- **Documentación**: este README actúa como guía completa de instalación, configuración y gestión.

## 🛠️ Prerrequisitos
Antes de comenzar, asegúrate de tener las siguientes herramientas instaladas:
- **WSL 2**: Windows Subsystem for Linux (Ubuntu recomendado)  
- **Minikube**: clúster de Kubernetes de un solo nodo para desarrollo local  
- **kubectl**: CLI de Kubernetes  
- **Helm**: gestor de paquetes de Kubernetes

### Instalación rápida (desde WSL)
```bash
# Iniciar minikube con el driver de docker
minikube start --driver=docker

# Instalar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Instalar Helm
curl -fsSL https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz | tar -xz
sudo mv linux-amd64/helm /usr/local/bin/helm
🚀 Instalación y Despliegue
1️⃣ Habilitar Addons de Minikube
bash
Copy code
minikube addons enable default-storageclass   # Aprovisionador de volúmenes
minikube addons enable ingress                # Controlador Ingress (Nginx)
2️⃣ Clonar el Repositorio e Instalar el Chart
bash
Copy code
git clone <URL_DEL_REPOSITORIO>
cd radarr-chart

# Reemplaza "TuContrasenaSegura" con una contraseña robusta
helm install radarr-release . --set database.password=TuContrasenaSegura
3️⃣ Configurar el Acceso con minikube tunnel
En una nueva terminal:

bash
Copy code
minikube tunnel
Edita el archivo C:\Windows\System32\drivers\etc\hosts (como Administrador) y añade:

lua
Copy code
127.0.0.1    radarr.minikube.local
⚙️ Configuración del Chart
Personaliza el despliegue editando values.yaml o pasando valores con --set.

Parámetro	Descripción	Valor por defecto
replicaCount	Réplicas del despliegue de Radarr	2
database.storage	Tamaño del volumen de PostgreSQL	5Gi
persistence.config.storage	Tamaño del volumen para configuración de Radarr	10Gi
persistence.downloads.storage	Tamaño del volumen para descargas de Radarr	50Gi
autoscaling.enabled	Habilita el auto-escalado	true
autoscaling.targetCPUUtilizationPercentage	Porcentaje de CPU para escalar	70
ingress.enabled	Habilita Ingress	true
ingress.host	Dominio de acceso a la app	radarr.minikube.local

🌐 Acceso y Gestión
Acceder a la aplicación
Una vez activo el túnel y configurado el hosts, visita:
URL: http://radarr.minikube.local

Verificación del despliegue
bash
Copy code
kubectl get all -l app.kubernetes.io/instance=radarr-release   # Estado de recursos
kubectl logs <radarr-pod-name>                                 # Logs de un pod
kubectl get hpa                                                # Estado del HPA
Escalado manual
bash
Copy code
kubectl scale deployment radarr-release-radarr-chart --replicas=4
📂 Estructura del Chart
bash
Copy code
radarr-chart/
├── Chart.yaml                   # Metadatos del chart
├── values.yaml                  # Configuraciones por defecto
├── templates/
│   ├── _helpers.tpl             # Funciones de plantilla
│   ├── hpa.yaml                 # HorizontalPodAutoscaler
│   ├── postgres-statefulset.yaml# StatefulSet de la base de datos
│   ├── postgres-service.yaml    # Service de PostgreSQL
│   ├── radarr-deployment.yaml   # Deployment de Radarr
│   ├── radarr-service.yaml      # Service de Radarr
│   ├── radarr-pvc.yaml          # PVCs para Radarr
│   ├── radarr-ingress.yaml      # Ingress
│   └── secrets.yaml             # Secrets para credenciales
└── README.md                    # Documentación del chart
Copy code







Ask ChatGPT





ChatGPT can make mistakes. Check important info. See Cookie Preferences.
