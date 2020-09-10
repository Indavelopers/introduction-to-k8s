author: Marcos Manuel Ortega
summary: Despliega una app de libro de visitas en PHP con Redis
id: lab-guestbook
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Desplegar una app de libro de visitas en PHP con Redis

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo desplegar una app con un frontend de libro de visitas en PHP con un Servicio de balanceador de carga
- Cómo desplegar un backend con un clúster de Redis con un Servicio interno

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
kubectl create namespace lab-guestbook
```

## Desplegar el clúster de Redis
Vamos a desplegar un clúster de Redis con un despliegue para el máster y otro para los trabajadores.

1. Crea el archivo `deployments/demo-guestbook-redis-master.yaml` con el siguiente manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-redis-master
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
      role: master
      tier: backend
  replicas: 1
  template:
    metadata:
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      containers:
      - name: master
        image: k8s.gcr.io/redis:e2e
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-guestbook-redis-master.yaml -n lab-guestbook
```

1. Verifica que el Despliegue ha sido creado correctamente y anota el nombre del Pod, `[NOMBRE_POD]`:
```
kubectl get pods -n lab-guestbook
```

1. Comprueba los logs del Pod del máster de Redis:
```
kubectl logs -f [NOMBRE_POD] -n lab-guestbook
```

1. Verifica los detalles del Despliegue:
```
kubectl describe deployment demo-redis-master -n lab-guestbook
```

### Desplegar el Servicio interno para el máster de Redis
1. Crea el archivo `services/demo-guestbook-redis-master.yaml` con el siguiente manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  name: demo-redis-master
  labels:
    app: redis
    role: master
    tier: backend
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: master
    tier: backend
```

1. Crea el Servicio:
```
kubectl apply -f services/demo-guestbook-redis-master.yaml -n lab-guestbook
```

1. Verifica que el Servicio ha sido creado correctamente:
```
kubectl get services -n lab-guestbook
```

1. Verifica los detalles del Servicio:
```
kubectl describe services demo-redis-master -n lab-guestbook
```

### Desplegar el Despligue de los trabajadores de Redis
1. Crea el archivo `deployments/demo-guestbook-redis-slaves.yaml` con el siguiente manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-redis-slaves
  labels:
    app: redis
spec:
  selector:
    matchLabels:
      app: redis
      role: slave
      tier: backend
  replicas: 2
  template:
    metadata:
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      containers:
      - name: slave
        image: gcr.io/google_samples/gb-redisslave:v3
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
        ports:
        - containerPort: 6379
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-guestbook-redis-slaves.yaml -n lab-guestbook
```

1. Verifica que el Despliegue ha sido creado correctamente:
```
kubectl get pods -n lab-guestbook
```

    Verás las 2 réplicas de los trabajadores y 1 réplica del máster de Redis.

1. Verifica los detalles del Despliegue:
```
kubectl describe deployment demo-redis-slaves -n lab-guestbook
```

### Desplegar el Servicio interno para el máster de Redis
1. Crea el archivo `services/demo-guestbook-redis-slaves.yaml` con el siguiente manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  name: demo-redis-slaves
  labels:
    app: redis
    role: slave
    tier: backend
spec:
  ports:
  - port: 6379
  selector:
    app: redis
    role: slave
    tier: backend
```

1. Crea el Servicio:
```
kubectl apply -f services/demo-guestbook-redis-slaves.yaml -n lab-guestbook
```

1. Verifica que el Servicio ha sido creado correctamente:
```
kubectl get services -n lab-guestbook
```

    Verás el Servicio del máster y los trabajadores de Redis.

1. Verifica los detalles del Servicio:
```
kubectl describe services demo-redis-slaves -n lab-guestbook
```

## Despliega el frontend del libro de visitas
El frontend de la aplicación del libro de visitras está escrito en PHP y está configurado para usar el máster de Redis para escribir datos y los trabajadores de Redis para las lecturas.

1. Crea el archivo `deployments/demo-guestbook-frontend.yaml` con el siguiente manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-frontend
  labels:
    app: guestbook
spec:
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  replicas: 3
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google-samples/gb-frontend:v4
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
        ports:
        - containerPort: 80
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-guestbook-frontend.yaml -n lab-guestbook
```

1. Verifica que el Despliegue ha sido creado correctamente:
```
kubectl get pods -n lab-guestbook
```

    Verás las 2 réplicas de los trabajadores y 1 réplica del máster de Redis.

1. Verifica los detalles del Despliegue:
```
kubectl describe deployment demo-frontend -n lab-guestbook
```

### Desplegar el Servicio para el frontend
1. Crea el archivo `services/demo-guestbook-frontend.yaml` con el siguiente manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  name: demo-frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: guestbook
    tier: frontend
```

1. Crea el Servicio:
```
kubectl apply -f services/demo-guestbook-frontend.yaml -n lab-guestbook
```

1. Verifica que el Servicio ha sido creado correctamente y anota su IP externa, `[IP_EXTERNA]` (puede tardar hasta 5 minutos):
```
kubectl get services -n lab-guestbook -w
```

    Verás el Servicio del máster y los trabajadores de Redis, junto al del frontend.

1. Verifica los detalles del Servicio:
```
kubectl describe services demo-frontend -n lab-guestbook
```

1. Comprueba la respuesta de la aplicación web en una pestaña de tu navegador o con `curl`:
```
http://[IP_EXTERNA]
```

## Escalar el frontend
Escalar el nº de réplicas del frontend es fácil, puesto que la aplicación se define como un único punto de entrada, el Servicio, que redirige las peticiones a diferentes réplicas del frontend.

1. Escala el número de Pods del frontend:
```
kubectl scale deployment frontend --replicas 5 -n lab-guestbook
```

1. Comprueba el número de Pods del frontend desplegados:
```
kubectl get pods -n lab-guestbook -l app=guestbook
```

1. Para escalarlo de nuevo hacia abajo, usa el mismo comando:
```
kubectl scale deployment frontend --replicas 3 -n lab-guestbook
```

1. Comprueba el número de Pods del frontend desplegados:
```
kubectl get pods -n lab-guestbook -l app=guestbook
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-guestbook
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo desplegar una app con un frontend de guestbook en PHP con un Servicio de balanceador de carga
- Cómo desplegar un backend con un clúster de Redis con un Servicio interno
