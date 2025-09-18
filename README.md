Práctica: Despliegue de Radarr en Kubernetes con Helm
Este repositorio contiene un chart de Helm para desplegar la aplicación Radarr junto con una base de datos PostgreSQL en un clúster de Kubernetes. La solución está diseñada para ser robusta, escalable y persistente, cumpliendo con los objetivos de la práctica de despliegue de aplicaciones en la nube.

Objetivos de la Práctica
El chart de Helm ha sido diseñado para cumplir los siguientes objetivos:

Chart de Helm: Se ha creado un chart que encapsula todos los recursos de Kubernetes necesarios para el despliegue.

Persistencia de Datos: La base de datos de PostgreSQL y la configuración de Radarr utilizan PersistentVolumeClaim (PVC) para asegurar que los datos no se pierdan al reiniciar los pods.

Configuración Segura: Las credenciales sensibles, como la contraseña de la base de datos, se gestionan de forma segura utilizando Secrets de Kubernetes, evitando que aparezcan en el repositorio.

Alta Disponibilidad (HA): El despliegue de Radarr se configura con un número mínimo de réplicas para garantizar la alta disponibilidad de la aplicación.

Auto-escalado: Se ha implementado un HorizontalPodAutoscaler (HPA) que ajusta automáticamente el número de réplicas de Radarr en función del uso de CPU.

Exposición Externa: La aplicación es accesible desde fuera del clúster utilizando un recurso de Ingress, lo que permite un acceso más flexible y basado en nombres de dominio.

Resiliencia: Se han configurado liveness y readiness probes para asegurar que solo los pods sanos reciban tráfico y que los pods que no responden sean reiniciados.

Documentación: Este README proporciona una guía completa para la instalación, configuración y gestión del despliegue.

Prerrequisitos
Antes de comenzar, asegúrate de tener las siguientes herramientas instaladas y configuradas:

WSL 2: Windows Subsystem for Linux con una distribución como Ubuntu.

Minikube: Un clúster de Kubernetes de un solo nodo para desarrollo local.

kubectl: La línea de comandos de Kubernetes.

Helm: El gestor de paquetes de Kubernetes.

Puedes instalar Minikube y Helm desde WSL con los siguientes comandos:

Bash

# Iniciar minikube con el driver de docker en WSL
minikube start --driver=docker

# Instalar kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Instalar Helm
curl -fsSL https://get.helm.sh/helm-v3.12.3-linux-amd64.tar.gz | tar -xz && sudo mv linux-amd64/helm /usr/local/bin/helm
Instalación y Despliegue
Sigue estos pasos para desplegar la aplicación en tu clúster de Kubernetes local.

1. Habilitar Addons de Minikube
Para asegurar el correcto funcionamiento del aprovisionamiento de volúmenes persistentes y el enrutamiento de Ingress, habilita los addons necesarios en Minikube.

Bash

# Habilitar el aprovisionador de volúmenes por defecto
minikube addons enable default-storageclass

# Habilitar el controlador de Ingress (Nginx)
minikube addons enable ingress
2. Clonar el Repositorio e Instalar el Chart de Helm
Clona el repositorio que contiene el chart de Helm y navega hasta la carpeta raíz del chart.

Bash

git clone <URL_del_repositorio>
cd radarr-chart
Instala el chart de Helm. Es fundamental que la contraseña de PostgreSQL se pase como un valor secreto y no se suba al repositorio.

Bash

# Reemplaza "TuContrasenaSegura" con una contraseña robusta
helm install radarr-release . --set database.password=TuContrasenaSegura
3. Configurar el Acceso con minikube tunnel
Para que el Ingress funcione, necesitas crear un túnel de red que redirija el tráfico de tu host a Minikube.

Abre una nueva terminal en WSL y ejecuta el siguiente comando:

Bash

minikube tunnel
Esta terminal se quedará abierta y no mostrará un prompt. Es la encargada de mantener el túnel activo.

Añade una entrada al archivo hosts de tu sistema operativo para resolver el nombre de dominio del Ingress. Abre el Bloc de Notas (Notepad) como Administrador en Windows y añade la siguiente línea al final del archivo C:\Windows\System32\drivers\etc\hosts:

127.0.0.1       radarr.minikube.local
Configuración del Chart
Puedes personalizar el despliegue editando el archivo values.yaml o pasando los valores directamente en la línea de comandos (--set). A continuación, se detallan los parámetros más importantes.

Parámetro	Descripción	Valor por defecto
replicaCount	Número de réplicas del despliegue de Radarr.	2
database.storage	Tamaño del volumen persistente para la base de datos de PostgreSQL.	5Gi
persistence.config.storage	Tamaño del volumen persistente para la configuración de Radarr.	10Gi
persistence.downloads.storage	Tamaño del volumen persistente para la carpeta de descargas de Radarr.	50Gi
autoscaling.enabled	Habilita el auto-escalado horizontal de pods.	true
autoscaling.targetCPUUtilizationPercentage	Porcentaje de uso de CPU para activar el escalado.	70
ingress.enabled	Habilita el recurso de Ingress para exponer la aplicación.	true
ingress.host	Nombre de dominio para acceder a la aplicación a través del Ingress.	radarr.minikube.local

Exportar a Hojas de cálculo
Acceso y Gestión
Acceder a la Aplicación
Una vez que el túnel de Minikube esté activo y hayas configurado el archivo hosts, accede a Radarr desde tu navegador:

URL: http://radarr.minikube.local

Verificación del Despliegue
Puedes verificar el estado de los recursos de Kubernetes con los siguientes comandos:

Bash

# Ver el estado de todos los recursos del chart
kubectl get all -l app.kubernetes.io/instance=radarr-release

# Ver los logs de un pod de Radarr (reemplaza <pod-name>)
kubectl logs <radarr-pod-name>

# Ver el estado del HorizontalPodAutoscaler
kubectl get hpa
Escalado Manual
Si necesitas ajustar el número de réplicas manualmente, puedes hacerlo con un comando kubectl scale o modificando el values.yaml y usando helm upgrade.

Bash

# Escalar el despliegue a 4 réplicas
kubectl scale deployment radarr-release-radarr-chart --replicas=4
Estructura del Chart
radarr-chart/
├── Chart.yaml                  # Metadatos del chart
├── values.yaml                 # Configuraciones por defecto
├── templates/
│   ├── _helpers.tpl            # Funciones de plantilla reutilizables
│   ├── hpa.yaml                # HorizontalPodAutoscaler para el auto-escalado
│   ├── postgres-statefulset.yaml # StatefulSet para la base de datos persistente
│   ├── postgres-service.yaml   # Service para la base de datos
│   ├── radarr-deployment.yaml  # Deployment de la aplicación Radarr
│   ├── radarr-service.yaml     # Service de la aplicación Radarr
│   ├── radarr-pvc.yaml         # PVCs para la persistencia de Radarr
│   ├── radarr-ingress.yaml     # Ingress para la exposición externa
│   └── secrets.yaml            # Secret para las credenciales sensibles
└── README.md                   # Documentación del chart
