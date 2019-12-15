author: Marcos Manuel Ortega
summary: Desplegando Jobs y CronJobs
id: lab-jobs-cronjobs
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Desplegando Jobs y CronJobs

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo desplegar Jobs
- Cómo desplegar CronJobs

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
kubectl create namespace lab-jobs-cronjobs
```

## Desplegar un Job
Vamos a desplegar un Job con una tarea contenerizada determinada.

1. Crea el archivo `jobs/demo-job.yaml` con este manifiesto:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: demo-job
spec:
  template:
    metadata:
      name: demo-job
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl"]
        args: ["-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
```

1. Crea el Job:
```
kubectl apply -f jobs/demo-job.yaml -n lab-jobs-cronjobs
```

1. Verifica que los Pods están ejecutándose:
```
kubectl get pods -n lab-jobs-cronjobs
```

1. Espera unos segundos y comprueba como el estado cambia a `Completed` cuando el Job finaliza su ejecución:
```
kubectl get pods -n lab-jobs-cronjobs -w
```

    Cancela el comando "watch" con `CTRL + C`.

1. Verifica los detalles del Job y su estado:
```
kubectl describe jobs demo-job -n lab-jobs-cronjobs
```

    Cuando el Job acaba, el controlador elimina los Pods creados, pero no elimina el objeto Job, para poder comprobar sus logs y estado.

1. Encuentra el nombre del Pod creado por el Job y apúntalo, [NOMBRE_POD]:
```
kubectl get pods -n lab-jobs-cronjobs
```

1. Recupera los logs del Job:
```
kubectl logs [NOMBRE_POD] -n lab-jobs-cronjobs
```

## Desplegar un CronJob
Vamos a desplegar un CronJob con una tarea cron contenerizada determinada.

1. Crea el archivo `jobs/demo-cronjob.yaml` con este manifiesto:
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: demo-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello-world
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo "Hello world from a CronJob!"
          restartPolicy: OnFailure
```

1. Crea el CronJob:
```
kubectl apply -f jobs/demo-cronjob.yaml -n lab-jobs-cronjobs
```

1. Verifica que el CronJob ha sido creado correctamente:
```
kubectl get jobs -n lab-jobs-cronjobs
```

1. Verifica los detalles del CronJob y su estado:
```
kubectl describe jobs demo-cronjob -n lab-jobs-cronjobs
```

    Al desplegarse, el CronJob crea un Pod para ejecutar su tarea cron. Encuentra el nombre del Pod creado en el anterior comando y apúntalo, [NOMBRE_POD]

1. Recupera los logs del Pod creado por el CronJob:
```
kubectl logs [NOMBRE_POD] -n lab-jobs-cronjobs
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-jobs-cronjobs
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo desplegar Jobs
- Cómo desplegar CronJobs