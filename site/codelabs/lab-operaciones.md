author: Marcos Manuel Ortega
summary: Administración y operaciones en K8s
id: lab-operaciones
categories: introduction-to-k8s
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Operaciones

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- xxx
- kubectl explain pods
- kubectl run pod
- pod manifest
- kubectl get pods
- kubectl describe pods
- curl
- kubectl get, describe, exec, logs, port-forward

## Entorno
1. Accede a [console.cloud.google.com](https://console.cloud.google.com)
2. Loguéate con tu cuenta de usuario elegida para el curso.
3. Una vez ya en la consola, en la esquina superior izquierda busca un desplegable con 3 hexágonos pequeños para seleccionar tu proyecto.
4. Ábrelo y selecciona tu proyecto asignado. Puedes tener que hacer click en el desplegable de organizaciones y cambiar a **Sin organización**.
5. Te recomendamos dividir el escritorio horizontalmente en dos ventanas, una con la consola y/o Cloud Shell y otra con las instrucciones.
6. Activa Cloud Shell en el icono de la parte superior derecha `>_`.
7. Puedes configurar la completación de comandos para kubectl con la tecla "TAB":
```
source <(kubectl completion bash)
```

    Comprueba que funciona escribiendo `kubectl` y presionando la tecla "TAB" 2 veces.

## Desplegar un clúster de K8s
1. Configurar la completación de comandos para kubectl con la tecla "TAB":
```
source <(kubectl completion bash)
```

1. Despliega un clúster de K8s en Google Kubernetes Engine:
```
gcloud container clusters create lab-cluster --num-nodes 3 --zone europe-west2-c
```

1. Configura el acceso a dicho clúster para kubectl:
```
gcloud container clusters get-credentials lab-cluster --zone europe-west2-c
```

1. Comprueba que estás conectado a dicho clúster:
```
kubectl cluster-info
```

## Para terminar

En esta práctica hemos visto cómo:

### What we've covered
- Desplegar un clúster de K8s
- Conectarte a dicho clúster
- Revisar la información del clúster

### Eliminar los recursos usados
- En este caso, no tienes que eliminar ningún recurso. Continuaremos usando el clúster en siguientes ejercicios