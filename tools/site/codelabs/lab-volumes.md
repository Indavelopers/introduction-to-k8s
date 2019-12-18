author: Marcos Manuel Ortega
summary: Configurar un Pod para usar un Volumen
id: lab-volumes
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Configurar un Pod para usar un Volumen como almacenamiento

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo configurar un Pod para usar un Volumen como almacenamiento

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
kubectl create namespace lab-volumes
```

## Configurar un Volumen para un Pod
Vamos a crear un Pod que ejecuta un Contenedor y tiene asignado un Volumen de tipo emptyDir que comparte ciclo de vida con el Pod, incluso si el Contenedor se reinicia.

Para crear un Pod con dicho Volumen:

1. Crea el archivo `pods/redis.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: redis-storage
      mountPath: /data/redis
  volumes:
  - name: redis-storage
    emptyDir: {}
```

1. Crea el Pod:
```
kubectl apply -f pods/redis.yaml -n lab-volumes
```

1. Espera a que esté disponible, en estado `Running`, y continúa vigilando su estado:
```
kubectl get pods -n lab-volumes -w
```

1. Abre otra pestaña de sesión en Cloud Shell.

1. En la nueva pestaña, ejecuta una shell en el Contenedor en ejecución:
```
kubectl exec -it redis -- /bin/bash
```

1. Ve a `/data/redis` y crea un archivo:
```
cd /data/redis/
echo "Hello world!" > hello_world.txt
```

1. En la misma shell del contenedor, lista los procesos en ejecución instalando procps:
```
apt-get update
apt-get install procps
ps aux
```

    Comprueba que el proceso `redis` está en ejecución.

1. Elimina el proceso `redis` para forzar el reinicio del Contenedor usando el `[PID]` correspondiente:
```
kill [PID]
```

1. En la pestaña de Cloud Shell original, vigila los cambios en el Pod Redis. Debes poder comprobar como el Pod se ha reiniciado y ha vuelto al estado de "Running".

1. Pulsa `CTRL + C` para detener el comando.

1. Ejecuta una nueva shell en el Contenedor reiniciado, en cualquiera de las dos pestañas de Cloud Shell.

1. Ve a `/data/redis/` y verifica que el archivo continua ahí:
```
cd /data/redis
ls
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-volumes
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo configurar un Pod para usar un Volumen como almacenamiento