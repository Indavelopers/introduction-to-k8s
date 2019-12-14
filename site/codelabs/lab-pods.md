author: Marcos Manuel Ortega
summary: Desplegar pods en K8s
id: lab-pods
categories: introduction-to-k8s
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Desplegar pods en K8s

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo ejecutar pods de forma imperativa
- Cómo desplegar pods de forma declarativa
- Cómo comprobar la respuesta del puerto de un pod
- Cómo eliminar pods

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

## Desplegar un pod
1. En primer lugar, explora la documentación de los pods:
```
kubectl explain pods
kubectl explain pods.spec
kubectl explain pods.spec.containers
```

1. Ejecuta un pod de NGINX:
```
kubectl run nginx --image nginx --generator=run-pod/v1
```

1. Lista los pods en ejecución:
```
kubectl get pods
```

1. Ejecuta un nuevo pod de NGINX con el puerto 80 expuesto:
```
kubectl run nginx-open-port --image nginx --generator=run-pod/v1 --port 80
```

1. Lista los pods en ejecución de nuevo:
```
kubectl get pods
```

1. Abre otra nueva sesión en una nueva pestaña de Cloud Shell.

1. En la nueva pestaña, activa la redirección de puertos al puerto local 8080:
```
kubectl port-forward nginx-open-port 8080:80
```

1. Vuelve a la pestaña anterior y comprueba la respuesta del pod en dicho puerto con `curl` o en una pestaña de tu navegador (con la función "web preview" de Cloud Shell):
```
curl localhost:8080
```

    Debes poder acceder a la página de muestra de NGINX.

## Crear pods a través de un manifiesto

1. Copia la siguiente definición de Pod en un archivo `pods/nginx.yaml`:
```
mkdir pods
nano pods/nginx.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    -  containerPort: 80
```

1. Crea el pod de forma declarativa:
```
kubectl apply -f pods/nginx.yaml
```

1. Lista los pods en ejecución de nuevo:
```
kubectl get pods
```

1. Elimina los pods en ejecución:
```
kubectl delete pod nginx nginx-open-port nginx-app
```

## Reto
**¿Serías capaz de desplegar 3 réplicas de la imagen de [Apache 2](https://hub.docker.com/_/httpd)?**

**¿Y hacerlo a partir de un manifiesto?**

## Para terminar

En esta práctica hemos visto cómo:

### What we've covered
- Cómo ejecutar pods de forma imperativa
- Cómo desplegar pods de forma declarativa
- Cómo comprobar la respuesta del puerto de un pod
- Cómo eliminar pods