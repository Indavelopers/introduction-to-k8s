author: Marcos Manuel Ortega
summary: Configure Pod initialization
id: lab-pod-initialization
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Configurar un Contenedor de inicialización para un Pod

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Crear un Pod con un Contenedor de inicialización

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
kubectl create namespace lab-pod-initialization
```

## Crear un Pod con un Contenedor de inicialización
Vamos a crear un Pod con un Contenedor de aplicación y otro Contenedor de inicialización, que se ejecutará hasta completarse antes de que el Contenedor de aplicación comience.

1. Crea el archivo `pods/init-container.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: demo-init
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  initContainers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://kubernetes.io
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```

    En la definición del Pod especificamos un Volumen de tipo `emptyDir` que comparten ambos Contenedores. El contenedor de inicialización crea un archivo HTML en el mismo con la web de [k8s.io](https://k8s.io), que el contenedor NGINX finalmente muestra.

1. Crea el Pod:
```
kubectl apply -f pods/init-container.yaml -n lab-pod-initialization
```

1. Verifica que está ejecutándose:
```
kubectl get pods demo-init -n lab-pod-initialization
```

1. Verifica sus Contenedores y Volumen:
```
kubectl describe pods demo-init -n lab-pod-initialization
```

    *¿Cuántos segundos ha tardado el Contenedor de inicialización en ejecutarse? ¿La imagen de NGINX se ha descargado antes o después de que terminara de ejecutarse?*

1. Establece una shell en el Contenedor NGINX del Pod:
```
kubectl exec -it demo-init -n lab-pod-initialization -- /bin/bash
```

1. Comprueba la respuesta del servidor NGINX:
```
apt update
apt install curl -y
curl localhost
```

    La salida debe mostrar la web del archivo creado por el Contenedor de inicialización.

1. Sal de la shell:
```
exit
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-pod-initialization
```

En esta práctica hemos visto cómo:

### What we've covered
- Crear un Pod con un Contenedor de inicialización