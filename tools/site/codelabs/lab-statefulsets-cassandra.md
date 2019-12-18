author: Marcos Manuel Ortega
summary: Desplegando Cassandra con un StatefulSet
id: lab-statefulsets-cassandra
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Desplegando Cassandra con un StatefulSet

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Crear un Servicio "headless" para Cassandra
- Desplegar un anillo de Cassandra con un StatefulSet y validarlo
- Modificar el StatefulSet

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
kubectl create namespace lab-statefulsets-cassandra
```

### Configuración de Cassandra
Vamos a desplegar un anillo de [Cassandra]() usando la imagen [gcr.io/google-samples/cassandra:v13](https://github.com/kubernetes/examples/blob/master/cassandra/image/Dockerfile) de Google Container Registry, basada en Debian e incluyendo OpenJDK 8.

La imagen incluye una instalación estándar de Cassandra del repositorio de Debian que puedes configurar con las siguientes variables de entorno, insertadas en `cassandra.yaml`:
- CASSANDRA_CLUSTER_NAME = 'Test Cluster'
- CASSANDRA_NUM_TOKENS = 32
- CASSANDRA_RPC_ADDRESS = 0.0.0.0

*Se muestran sus valores por defecto*.

## Creando un Servicio "headless" para Cassandra
Este Servicio es usado para resolución de DNS entre los Pods de Cassandra y sus clientes en el clúster de K8s.

1. Crea el archivo `services/cassandra.yaml` con este manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  labels:
    app: cassandra
  name: cassandra
spec:
  clusterIP: None
  ports:
  - port: 9042
  selector:
    app: cassandra
```

1. Crea el Servicio:
```
kubectl apply -f services/cassandra.yaml -n lab-statefulsets-cassandra
```

1. Verifica que se ha creado correctamente:
```
kubectl get services -n lab-statefulsets-cassandra
```

1. Verifica los detalles del Servicio:
```
kubectl describe services cassandra -n lab-statefulsets-cassandra
```

## Creando un anillo de Cassandra
Usaremos un StatefulSet para desplegar un anillo con 3 Pods de Cassandra.

También declararemos una StorageClass para los volúmenes en el mismo manifiesto.

1. Crea el archivo `deployments/cassandra.yaml` con este manifiesto:
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra
  labels:
    app: cassandra
spec:
  serviceName: cassandra
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      terminationGracePeriodSeconds: 1800
      containers:
      - name: cassandra
        image: gcr.io/google-samples/cassandra:v13
        imagePullPolicy: Always
        ports:
        - containerPort: 7000
          name: intra-node
        - containerPort: 7001
          name: tls-intra-node
        - containerPort: 7199
          name: jmx
        - containerPort: 9042
          name: cql
        resources:
          limits:
            cpu: "500m"
            memory: 1Gi
          requests:
            cpu: "500m"
            memory: 1Gi
        securityContext:
          capabilities:
            add:
              - IPC_LOCK
        lifecycle:
          preStop:
            exec:
              command: 
              - /bin/sh
              - -c
              - nodetool drain
        env:
          - name: MAX_HEAP_SIZE
            value: 512M
          - name: HEAP_NEWSIZE
            value: 100M
          - name: CASSANDRA_SEEDS
            value: "cassandra-0.cassandra.statefulsets-cassandra.svc.cluster.local"
          - name: CASSANDRA_CLUSTER_NAME
            value: "demo-cassandra"
          - name: CASSANDRA_DC
            value: "DC1-demo-cassandra"
          - name: CASSANDRA_RACK
            value: "Rack1-demo-cassandra"
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
        readinessProbe:
          exec:
            command:
            - /bin/bash
            - -c
            - /ready-probe.sh
          initialDelaySeconds: 15
          timeoutSeconds: 5
        volumeMounts:
        - name: cassandra-data
          mountPath: /cassandra_data
  volumeClaimTemplates:
  - metadata:
      name: cassandra-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: fast
      resources:
        requests:
          storage: 1Gi
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

1. Crea el StatefulSet y la StorageClass:
```
kubectl apply -f deployments/cassandra.yaml -n lab-statefulsets-cassandra
```

1. Verifica que se ha creado correctamente:
```
kubectl get statefulsets -n lab-statefulsets-cassandra
```

1. Verifica el estado del orden de creación de los Pods (puede tardar unos minutos en completarse):
```
kubectl get pods -l app=cassandra -n lab-statefulsets-cassandra -w
```

1. Verifica los detalles del StatefulSet:
```
kubectl describe statefulsets cassandra -n lab-statefulsets-cassandra
```

1. Usa el comando `nodetool` de Cassandra para mostrar el estado del anillo:
```
kubectl exec -it cassandra-0 -n lab-statefulsets-cassandra -- nodetool status
```

## Modificar el StatefulSet de Cassandra
Vamos a modificar el número de Pods del anillo de Cassandra.

1. Edita el StatefulSet con el siguiente comando:
```
kubectl edit statefulsets cassandra -n lab-statefulsets-cassandra
```

    Se abrirá un editor por defecto.

1. Busca la línea con el campo `replicas`, cámbialo a 4 y guarda el manifiesto.

1. Verifica que el StatefulSet está creando una nueva réplica:
```
kubectl get statefulsets cassandra -n lab-statefulsets-cassandra
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-statefulsets-cassandra
```

En esta práctica hemos visto cómo:

### What we've covered
- Crear un Servicio headless para Cassandra
- Desplegar un anillo de Cassandra con un StatefulSet y validarlo
- Modificar el StatefulSet