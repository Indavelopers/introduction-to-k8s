author: Marcos Manuel Ortega
summary: Exponer una aplicación externamente con un Servicio
id: lab-external-service
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Exponer una aplicación externamente con un Servicio

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Desplegar una aplicación con múltiples réplicas
- Crear un servicio de tipo NodePort
- Crear un servicio de tipo LoadBalancer

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
kubectl create namespace lab-external-service
```

## Desplegar una aplicación
Vamos a desplegar una aplicación con 5 réplicas de Pods usando un Despliegue.

1. Crea el nuevo directorio `deployments`:
```
mkdir deployments
```

1. Crea el archivo `deployments/demo-deployment.yaml` con este manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: demo-deployment
  template:
    metadata:
      labels:
        app: demo-deployment
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        name: hello-world
        ports:
        - containerPort: 8080
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-deployment.yaml -n lab-external-service
```

1. Verifica que los Pods están ejecutándose:
```
kubectl get pods -n lab-external-service
```

1. Verifica los detalles del Despliegue:
```
kubectl get deployments demo-deployment -n lab-external-service

kubectl describe deployment demo-deployment -n lab-external-service
```

1. Verifica los ReplicaSets creados:
```
kubectl get replicasets -n lab-external-service

kubectl describe replicasets -n lab-external-service
```

1. Expón el Despliegue externamente fuera del clúster:
```
kubectl expose deployment demo-deployment --type NodePort --name demo-service -n lab-external-service
```

1. Muestra la información del Servicio creado:
```
kubectl get services demo-service -n lab-external-service

kubectl describe services demo-service -n lab-external-service
```

1. En la descripción del Servicio anterior, comprueba los Endpoints creados en la línea que comienza por `Endpoints`:
```
kubectl describe services demo-service -n lab-external-service | grep Endpoints
```

1. Puedes comprobar que dichas IPs corresponden a los Pods del Despliegue:
```
kubectl get pods -o wide -n lab-external-service
```

1. Vuelve a comprobar el Servicio y apunta el valor del puerto del servicio, [PUERTO], en la línea `NodePort`:
``` 
kubectl describe services demo-service -n lab-external-service | grep NodePort
```

1. Encuentra la IP externa de uno de los Nodos y apúntala, [IP_NODO]:
```
gcloud compute instances list
```

1. Comprueba la respuesta del Servicio:
```
curl http://[IP_NODO]:[PUERTO]
```

1. Elimina el Servicio y recréalo como LoadBalancer:
```
kubectl delete service demo-service -n lab-external-service

kubectl expose deployment demo-deployment --type LoadBalancer --name demo-service -n lab-external-service
```

1. Tardará unos minutos en crear un balanceador de carga y asginarle una IP externa. Vigila sus detalles hasta que la IP cambie de `PENDING` a una dirección IP (ésto puede tardar hasta 4 minutos):
```
kubectl get service demo-service -n lab-external-service -w
```

1. Cancela el comando de "watch" con `CTRL + C`.

1. Comprueba la respuesta del Servicio de nuevo:
```
curl http://[IP_EXTERNA_LB]:8080
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-external-service
```

En esta práctica hemos visto cómo:

### What we've covered
- Desplegar una aplicación con múltiples réplicas
- Crear un servicio de tipo NodePort
- Crear un servicio de tipo LoadBalancer