author: Marcos Manuel Ortega
summary: Autoescalado de Pods
id: lab-hpa
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Autoescalado de Pods

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo configurar el autoescalado de Pods

Basado en la [documentación oficial de K8s](https://kubernetes.io/docs/tasks) ([licencia CC-BY-4.0](https://github.com/kubernetes/website/blob/master/LICENSE)).

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

### Crea un namespace
Crea un namespace para aislar los recursos de este ejercicio del resto de recursos en tu clúster:
```
kubectl create namespace lab-hpa
```

## Desplegar una aplicación de ejemplo
1. Crea el archivo `deployments/demo-app.yaml` con este manifiesto:
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      run: app
  template:
    metadata:
      labels:
        run: app
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: app
        ports:
        - containerPort: 8080
          protocol: TCP
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-app.yaml -n lab-hpa
```

1. Verifica que los Pods están ejecutándose:
```
kubectl get pods -n lab-hpa
```

1. Verifica los detalles del Despliegue:
```
kubectl describe deployments demo-app -n lab-hpa
```

1. Expón el Despliegue con un Servicio NodePort:
```
kubectl expose deployment demo-app --target-port 8080 --type NodePort -n lab-hpa
```

1. Verifica el Servicio creado:
```
kubectl get service demo-app -n lab-hpa
```

## Configurar autoescalado horizontal para los Pods
1. Verifica el número de réplicas del Despliegue:
```
kubectl describe deployment demo-app -n lab-hpa
```

1. Configura el autoescalado para el Despliegue:
```
kubectl autoscale deployment demo-app --max 4 --min 1 --cpu-percent 10 -n lab-hpa
```

1. Verifica que el HorizontalPodAutoscaler se ha creado correctamente:
```
kubectl get horizontalpodautoscaler -n lab-hpa
```

1. Verifica sus detalles:
```
kubectl describe horizontalpodautoscaler demo-app -n lab-hpa
```

## Comprueba el autoescalado de Pods
Para comprobarlo, necesitamos generar carga sobre la aplicación para que escale, lo que haremos con una aplicación de generación de carga.

1. Crea el archivo `deployments/load-generator.yaml` con este manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: load-generator
spec:
  replicas: 4
  selector:
    matchLabels:
      app: load_generator
  template:
    metadata:
      labels:
        app: load_generator
    spec:
      containers:
      - name: load_generator
        image: k8s.gcr.io/busybox
        args:
        - /bin/sh
        - -c
        - while true; do wget -q -O- http://web:8080; done
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/load-generator.yaml -n lab-hpa
```

1. Verifica que los Pods están ejecutándose:
```
kubectl get pods -n lab-hpa
```

1. Verifica los detalles del Despliegue:
```
kubectl describe deployments load-generator -n lab-hpa
```

    Una vez que el generador de carga comienza a generar tráfico, el Despliegue de la aplicación comienza a incrementarse.

1. Inspecciona el HorizontalPodAutoscaler y comprueba durante un par de minutos cómo varía su carga:
```
kubectl get hpa demo-app -n lab-hpa -w
```

1. Para detener la carga, elimina el Despliegue del generador de carga:
```
kubectl delete deployment load-generator -n lab-hpa
```

1. Verifica que se ha eliminado correctamente:
```
kubectl get deployments -n lab-hpa
```

1. Comprueba como el número de réplicas del Despliegue disminuye tras unos momentos:
```
kubectl get deployment demo-app -n lab-hpa -w
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-hpa
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo configurar el autoescalado de Pods