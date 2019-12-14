author: Marcos Manuel Ortega
summary: Configurar Calidad del Servicio para los Pods
id: lab-quality-of-service
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Configurar Calidad del Servicio para los Pods

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo configurar un Pod con una Calidad de Servicio de "garantizado"
- Cómo configurar un Pod con una Calidad de Servicio de "burstable/flexible"
- Cómo configurar un Pod con una Calidad de Servicio de "mejor esfuerzo"

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
kubectl create namespace lab-quality-of-service
```

## Asignar la Calidad de Servicio de "garantizado" a un Pod
Cuando K8s crea un Pod, le asigna una de las siguientes clases de Calidad de Servicio (QoS):
1. Garantizado
2. "Burstable"/flexible
3. "Best effort"/Mejor esfuerzo

Para obtener la clase de QoS de "garantizado" para un Pod, cada Contenedor del mismo:
1. Debe tener una petición y límite de memoria especificada, y deben coincidir.
2. Debe tener una petición y límite de CPU especificada, y deben coincidir.

Para crear un Pod con dicha clase de QoS:

1. Crea el archivo `pods/qos-pod-guaranteed.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-guaranteed
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
        cpu: "700m"
      requests:
        memory: "200Mi"
        cpu: "700m"
```

1. Crea el Pod:
```
kubectl apply -f pods/qos-pod-guaranteed.yaml -n lab-quality-of-service
```

1. Verifica que está ejecutándose:
```
kubectl get pods -n lab-quality-of-service
```

1. Comprueba las métricas de uso de recursos del Pod:
```
kubectl top pod qos-demo-guaranteed -n lab-quality-of-service
```

1. Comprueba la clase de QoS asignada:
```
kubectl get pods qos-demo-guaranteed -n lab-quality-of-service -o yaml
```

Busca la línea que comienza por "qosClass"

```
kubectl get pods qos-demo-guaranteed -n lab-quality-of-service -o yaml | grep qosClass
```

## Asignar la Calidad de Servicio de "burstable" a un Pod
Un Pod obtendrá la clase de QoS de "burstable" si:
1. No cumple los criterios para la clase de QoS de "garantizado".
2. Al menos un Contenedor en el Pod tiene una petición de memoria o CPU.

Para crear un Pod con dicha clase de QoS:

1. Crea el archivo `pods/qos-pod-burstable.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-burstable
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "100Mi"
```

1. Crea el Pod:
```
kubectl apply -f pods/qos-pod-burstable.yaml -n lab-quality-of-service
```

1. Verifica que está ejecutándose:
```
kubectl get pods -n lab-quality-of-service
```

1. Comprueba las métricas de uso de recursos del Pod:
```
kubectl top pod qos-demo-burstable -n lab-quality-of-service
```

1. Comprueba la clase de QoS asignada:
```
kubectl get pods qos-demo-burstable -n lab-quality-of-service -o yaml
```

Busca la línea que comienza por "qosClass"

```
kubectl get pods qos-demo-burstable -n lab-quality-of-service -o yaml | grep qosClass
```

## Asignar la Calidad de Servicio de "mejor esfuerzo" a un Pod
Un Pod obtendrá la clase de QoS de "mejor esfuerzo" si no cumple los requisitos para el resto de clases, ésto es, si sus Contenedores no tienen ninguna petición o límite de recursos.

Para crear un Pod con dicha clase de QoS:

1. Crea el archivo `pods/qos-pod-besteffort.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-besteffort
spec:
  containers:
  - name: qos-demo-ctr
    image: nginx
```

1. Crea el Pod:
```
kubectl apply -f pods/qos-pod-besteffort.yaml -n lab-quality-of-service
```

1. Verifica que está ejecutándose:
```
kubectl get pods -n lab-quality-of-service
```

1. Comprueba las métricas de uso de recursos del Pod:
```
kubectl top pod qos-demo-besteffort -n lab-quality-of-service
```

1. Comprueba la clase de QoS asignada:
```
kubectl get pods qos-demo-besteffort -n lab-quality-of-service -o yaml
```

Busca la línea que comienza por "qosClass"

```
kubectl get pods qos-demo-besteffort -n lab-quality-of-service -o yaml | grep qosClass
```

## Reto

*Crea un Pod con 2 Contenedores con la siguiente configuración. ¿Qué clase de QoS tendrá asignada?*

Archivo `pods/qos-pod-2containers.yaml`:
```
apiVersion: v1
kind: Pod
metadata:
  name: qos-demo-challenge
spec:
  containers:
  - name: qos-demo-ctr-1
    image: nginx
    resources:
      requests:
        memory: "200Mi"
  - name: qos-demo-ctr-2
    image: redis
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-quality-of-service
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo configurar un Pod con una Calidad de Servicio de "garantizado"
- Cómo configurar un Pod con una Calidad de Servicio de "burstable/flexible"
- Cómo configurar un Pod con una Calidad de Servicio de "mejor esfuerzo"