author: Marcos Manuel Ortega
summary: Asignar recursos de memoria a pods
id: lab-recursos-pods
categories: introduction-to-k8s
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Asignar recursos a pods

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo especificar una petición y límite de memoria
- Qué ocurre al exceder el límite de memoria del pod
- Qué ocurre al especificar una petición de memoria demasiado grande para los nodos

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
kubectl create namespace lab-recursos-pod
```

## Especifica una petición y límite de memoria
Vamos a crear un Pod con un Contenedor. El Contenedor tiene una petición de memoria de 100 MiB y un límite de memoria de 200 MiB.

1. Crea el archivo `pods/memory-request-limit.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

1. Crea el Pod:
```
kubectl apply -f pods/memory-request-limit.yaml -n lab-recursos-pod
```

1. Verifica que está ejecutándose:
```
kubectl get pods -n lab-recursos-pod
```

1. Comprueba las métricas de uso de recursos del Pod:
```
kubectl top pod memory-demo -n lab-recursos-pod
```

## Excediendo el límite de memoria de un Contenedor
Si un Contenedor del Pod excede su límite de memoria, se convierte en candidato para ser eliminado y reiniciado.

1. Crea el archivo `pods/memory-request-limit2.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
```

1. Crea el Pod:
```
kubectl apply -f pods/memory-request-limit2.yaml -n lab-recursos-pod
```

1. Verifica que está ejecutándose:
```
kubectl get pods -n lab-recursos-pod
```

1. Comprueba las métricas de uso de recursos del Pod:
```
kubectl top pod memory-demo-2 -n lab-recursos-pod
```

1. Vigila la ejecución del Pod hasta que sea eliminado:
```
kubectl get pods memory-demo-2 -n lab-recursos-pod -w
```

    Si ya ha sido eliminado, recréalo de nuevo.

    Una vez sea eliminado, se reiniciará automáticamente, y será eliminado de nuevo.

## Especifica una petición de memoria demasiado grande para los nodos
La asignación y ejecución de Pods depende de la petición de recursos del mismo, y los recursos disponibles en los nodos.

1. Crea el archivo `pods/memory-request-limit3.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-3
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-3-ctr
    image: polinux/stress
    resources:
      limits:
        memory: "1000Gi"
      requests:
        memory: "1000Gi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

1. Crea el Pod:
```
kubectl apply -f pods/memory-request-limit3.yaml -n lab-recursos-pod
```

1. Examina el estado del pod:
```
kubectl get pods -n lab-recursos-pod
```

Podrás comprobar que queda en estado `PENDING` indefinidamente.

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-recursos-pod
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo especificar una petición y límite de memoria
- Qué ocurre al exceder el límite de memoria del pod
- Qué ocurre al especificar una petición de memoria demasiado grande para los nodos