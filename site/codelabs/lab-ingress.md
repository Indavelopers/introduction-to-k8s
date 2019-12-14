author: Marcos Manuel Ortega
summary: Usar Ingress como LB L7 externo
id: lab-ingress
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Usar Ingress como balanceador de carga L7 externo

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo crear Servicios internos y externos
- Comprobar la resolución de DNS interna para los Servicios
- Cómo usar un Ingress para redirigir peticiones a diferentes Servicios

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
kubectl create namespace lab-ingress
```

## Crear un Servicio interno
Vamos a crear un Servicio interno de tipo ClusterIp para exponer un Despliegue internamente dentro del clúster.

1. Crea el archivo `deployments/demo-ingress-v1.yaml` con este manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-ingress
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-ingress
      version: v1
  template:
    metadata:
      labels:
        app: demo-ingress
        version: v1
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        name: hello-world
        ports:
        - containerPort: 8080
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-ingress-v1.yaml -n lab-ingress
```

1. Verifica que los Pods están ejecutándose:
```
kubectl get pods -n lab-ingress
```

1. Verifica los detalles del Despliegue:
```
kubectl describe deployments demo-ingress -n lab-ingress
```

1. Crea el archivo `services/demo-ingress-v1.yaml` con este manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  name: demo-service
  labels:
    version: v1
spec:
  type: ClusterIP
  selector:
    app: demo-ingress
    version: v1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

1. Crea el Servicio:
```
kubectl apply -f services/demo-ingress-v1.yaml -n lab-ingress
```

1. Verifica que el Servicio está en ejecución:
```
kubectl get services -n lab-ingress
```

1. Verifica los detalles del Servicio:
```
kubectl describe services demo-service -n lab-ingress
```

1. Anota la IP interna del Servicio, [SERVICIO_IP]

### Comprueba el Servicio interno
1. Despliega un Pod para comprobar la conexión al Servicio interno y la resolución de DNS:
```
kubectl run service-test-pod --image gcr.io/google-samples/node-hello:1.0 --generator=run-pod/v1 -n lab-ingress
```

1. Conéctate al Pod ejecutando una sesión shell:
```
kubectl exec -it service-test-pod -n lab-ingress -- sh
```

1. Comprueba la conexión al Servicio interno y la resolución de DNS:
```
apt update
apt install curl -y
curl [SERVICIO_IP]
curl demo-service.lab-ingress.svc.cluster.local
```

1. Sal de la sesión shell:
```
exit
```

## Convierte el Servicio en externo
Ahora convierte el Servicio en uno accesible desde fuera del Clúster usando una nueva versión del manifiesto con un Servicio de tipo NodePort.

1. Crea el archivo `services/demo-ingress-v1-nodeport.yaml` con este manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  name: demo-service
  labels:
    version: v1
spec:
  type: NodePort
  selector:
    app: demo-ingress
    version: v1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

    Fíjate que únicamente cambiamos el tipo de Servicio, manteniendo el nombre y el resto de características.

1. Actualiza el Servicio:
```
kubectl apply -f services/demo-ingress-v1-nodeport.yaml -n lab-ingress
```

1. Verifica los detalles del Servicio:
```
kubectl describe services demo-service -n lab-ingress
```

    Fíjate en el tipo de Servicio empleado.

1. Anota el puerto del nodo del Servicio, [PUERTO_NODO]

1. Anota la IP externa de uno de los Nodos:
```
gcloud compute instances list
```

1. Comprueba la conexión desde fuera del clúster al Servicio a través de la IP de uno de los Nodos, [IP_NODO] y el puerto del Servicio NodePort, [PUERTO_NODO]:
```
curl [IP_NODO]:[PUERTO_NODO]
```

## Despliega un nuevo Despliegue y Servicio de tipo LoadBalancer
Vamos a desplegar un nuevo Despliegue de Pods y un Servicio asociado a los mismos, esta vez de tipo LoadBalancer.

1. Crea el archivo `deployments/demo-ingress-v2.yaml` con este manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-ingress-v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo-ingress
      version: v2
  template:
    metadata:
      labels:
        app: demo-ingress
        version: v2
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:2.0
        name: hello-world
        ports:
        - containerPort: 8080
```

    Este manifiesto corresponde a otro Despliegue diferente, con una nueva versión de la misma imagen de Contenedor.

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-ingress-v2.yaml -n lab-ingress
```

1. Verifica que los Pods están ejecutándose:
```
kubectl get pods -n lab-ingress
```

1. Verifica los detalles del Despliegue:
```
kubectl describe deployments demo-ingress-v2 -n lab-ingress
```

1. Crea el archivo `services/demo-ingress-v2.yaml` con este manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  name: demo-service-v2
  labels:
    version: v2
spec:
  type: LoadBalancer
  selector:
    app: demo-ingress
    version: v2
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

1. Crea el Servicio:
```
kubectl apply -f services/demo-ingress-v2.yaml -n lab-ingress
```

1. Verifica que el Servicio está en ejecución. Espera a que tenga una IP externa asignada (puede tardar hasta 5 minutos):
```
kubectl get services -n lab-ingress -w
```

1. Verifica los detalles del Servicio:
```
kubectl describe services demo-service-v2 -n lab-ingress
```

1. Anota la IP externa del LoadBalancer del Servicio, [SERVICIO_IP].

1. Comprueba la conexión desde fuera del clúster al Servicio a través de la IP del balanceador de carga creado, [SERVICIO_IP]:
```
curl [SERVICIO_IP]
```

## Despliega un Ingress que redireccione a ambos Servicios
Vamos a desplegar un recurso Ingress que redireccionará internamente a ambos Servicios en función de la URL de la petición web.

1. Crea el archivo `services/demo-ingress-ingress.yaml` con este manifiesto:
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: demo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
    paths:
    - path: /v1
      backend:
        serviceName: demo-service
        servicePort: 80
    - path: /v2
      backend:
        serviceName: demo-service-v2
        servicePort: 80
```

1. Crea el Ingress:
```
kubectl apply -f services/demo-ingress-ingress.yaml -n lab-ingress
```

1. Verifica que el Ingress está en ejecución. Espera a que tenga una IP externa asignada (puede tardar hasta 5 minutos):
```
kubectl get ingress -n lab-ingress -w
```

1. Verifica los detalles del Ingress:
```
kubectl describe ingress demo-service-ingress -n lab-ingress
```

1. Anota la IP externa del LoadBalancer L7 del Ingress, [INGRESS_IP].

1. Comprueba la conexión desde fuera del clúster a ambos servicios a través de la IP del Ingress, [INGRESS_IP]:
```
curl [INGRESS_IP]/v1
curl [INGRESS_IP]/v2
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-ingress
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo crear Servicios internos y externos
- Comprobar la resolución de DNS interna para los Servicios
- Cómo usar un Ingress para redirigir peticiones a diferentes Servicios