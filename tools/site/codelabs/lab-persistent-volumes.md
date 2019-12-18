author: Marcos Manuel Ortega
summary: Configurar un Pod para usar un Volumen
id: lab-persistent-volumes
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Configurar un Pod para usar un Volumen Persistente como almacenamiento

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo crear un Volumen Persistente
- Cómo crear un PersistentVolumeClaim
- Cómo configurar un Pod para usar un Volumen Persistente como almacenamiento

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
kubectl create namespace lab-persistent-volumes
```

## Resumen del proceso
1. El administrador del clúster crea un Volumen Persistente soportado por un sistema de almacenamiento, sin asociarlo con ningún Pod.
2. El desarrollador crea un PersistentVolumeClaim que se asocia automáticamente a un Volumen Persistente compatible.
3. El desarrollador crea un Pod que usa el PersistentVolumeClaim anterior para almacenar archivos.


## Crea un disco
Necesitamos crear un disco persistente de Compute Engine previamente para que K8s pueda montarlo en un Nodo:

```
gcloud compute disks create pd-disk-1 --size 15Gb --zone europe-west1-c
```

## Crea un Volumen Persistente
En este ejercicio vamos a crear un PersistentVolume de tipo *gcePersistentDisk*. Este tipo de PersistentVolume usa un disco persistente de Google Compute Engine.

1. Crea el archivo `persistent-volumes/pv-volume.yaml` con este manifiesto:
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo-volume
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  gcePersistentDisk:
    fsType: ext4
    pdName: pd-disk-1
```

1. Crea el Volumen Persistente:
```
kubectl apply -f persistent-volumes/pv-volume.yaml
```

1. Verifica su estado:
```
kubectl get pv pv-demo-volume
```

    La respuesta muestra que el PersistentVolume tiene un estado de `AVAILABLE` o disponible, lo cual significa que aún no ha sido asociado a un PersistentVolumeClaim.

## Crea un PersistentVolumeClaim
El siguiente paso es crear un PersistentVolumeClaim, que será usado por el Pod para solicitar almacenamiento.

1. Crea el archivo `persistent-volumes/pv-claim.yaml` con este manifiesto:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-demo-claim
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```

1. Crea el PersistentVolumeClaim:
```
kubectl apply -f persistent-volumes/pv-claim.yaml -n lab-persistent-volumes
```

    Al ser creado, el PersistentVolumeClaim será asociado a un Volumen Persistente que satisfaga sus requisitos y la StorageClass seleccionada.

1. Verifica el estado del Volumen Persistente de nuevo:
```
kubectl get pv pv-demo-volume
```

    Ahora el estado muestra `BOUND` o asociado.

1. Verifica el estado del PersistentVolumeClaim:
```
kubectl get pvc pv-demo-claim -n lab-persistent-volumes
```

    La salida muestra que el PersistentVolumeClaim está asociado al Volumen Persistente `pv-demo-volume`.

Positive
: *¿Por qué crees que a veces usamos la flag `-n` para indicar el namespace y a veces no?*

## Crear un Pod que usa dicho PersistentVolumeClaim
El siguiente paso es crear un Pod que use dicho PersistentVolumeClaim como volumen de almacenamiento.

1. Crea el archivo `pods/pv-pod.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: pv-demo-pod
spec:
  containers:
    - name: pv-demo-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: pv-storage
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-demo-claim          
```

1. Crea el Pod:
```
kubectl apply -f pods/pv-pod.yaml -n lab-persistent-volumes
```

1. Verifica que está ejecutándose:
```
kubectl get pods pv-demo-pod -n lab-persistent-volumes
```

1. Verifica que tiene el Volumen pv-demo-volume asignado:
```
kubectl describe pod pv-demo-pod -n lab-persistent-volumes
```

## Para terminar

### Eliminar los recursos usados
1. Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-persistent-volumes
```

1. Elimina el disco persistente creado:
```
gcloud compute disks delete pd-disk-1 --zone europe-west1-c
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo configurar un Pod para usar un Volumen como almacenamiento