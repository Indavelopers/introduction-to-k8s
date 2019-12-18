author: Marcos Manuel Ortega
summary: Desplegando Zookeeper con un StatefulSet
id: lab-statefulsets-zookeeper
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Desplegando Zookeeper con un StatefulSet

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo desplegar un conjunto de Zookeeper usando un StatefulSet
- Cómo configurar el conjunto usando ConfigMaps
- Cómo expandir el despliegue de servidores de Zookeeper en el conjunto
- Cómo usar PodDisruptionBudgets para asegurar la disponibilidad del servicio durante el mantenimiento programado

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

### Crea un nuevo clúster
Vamos a crear un nuevo clúster de K8s con más recursos para poder desplegar Zookeeper:

1. Despliega un clúster de K8s en Google Kubernetes Engine:
```
gcloud container clusters create lab-zookeeper --num-nodes 4 --machine-type n1-standard-2 --zone europe-west1-c
```

1. Configura el acceso a dicho clúster para kubectl:
```
gcloud container clusters get-credentials lab-zookeeper --zone europe-west1-c
```

1. Comprueba que estás conectado a dicho clúster:
```
kubectl cluster-info
```

## Configurar aprovisionamiento dinámico de almacenamiento
Para configurarlo, debemos crear una StorageClass disponible en el clúster.

1. Crea el archivo `volumes/fast-storageclass.yaml` con este manifiesto:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

1. Crea la Clase de Almacenamiento:
```
kubectl apply -f volumes/fast-storageclass.yaml
```

## Crear un conjunto de Zookeeper
Vamos a utilizar un manifiesto compuesto para desplegar múltiples objetos a la misma vez: un Servicio "headless", un Servicio, un PodDisruptionBudget y un StatefulSet.

1. Crea el archivo `app/zookeeper.yaml` con este manifiesto:
```
apiVersion: v1
kind: Service
metadata:
  name: zk-hs
  labels:
    app: zk
spec:
  ports:
  - port: 2888
    name: server
  - port: 3888
    name: leader-election
  clusterIP: None
  selector:
    app: zk
---
apiVersion: v1
kind: Service
metadata:
  name: zk-cs
  labels:
    app: zk
spec:
  ports:
  - port: 2181
    name: client
  selector:
    app: zk
---
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: zk-pdb
spec:
  selector:
    matchLabels:
      app: zk
  maxUnavailable: 1
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zk
spec:
  selector:
    matchLabels:
      app: zk
  serviceName: zk-hs
  replicas: 3
  updateStrategy:
    type: RollingUpdate
  podManagementPolicy: OrderedReady
  template:
    metadata:
      labels:
        app: zk
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - zk
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: kubernetes-zookeeper
        imagePullPolicy: Always
        image: "k8s.gcr.io/kubernetes-zookeeper:1.0-3.4.10"
        resources:
          requests:
            memory: "1Gi"
            cpu: "0.5"
        ports:
        - containerPort: 2181
          name: client
        - containerPort: 2888
          name: server
        - containerPort: 3888
          name: leader-election
        command:
        - sh
        - -c
        - "start-zookeeper \
          --servers=3 \
          --data_dir=/var/lib/zookeeper/data \
          --data_log_dir=/var/lib/zookeeper/data/log \
          --conf_dir=/opt/zookeeper/conf \
          --client_port=2181 \
          --election_port=3888 \
          --server_port=2888 \
          --tick_time=2000 \
          --init_limit=10 \
          --sync_limit=5 \
          --heap=512M \
          --max_client_cnxns=60 \
          --snap_retain_count=3 \
          --purge_interval=12 \
          --max_session_timeout=40000 \
          --min_session_timeout=4000 \
          --log_level=INFO"
        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - "zookeeper-ready 2181"
          initialDelaySeconds: 10
          timeoutSeconds: 5
        volumeMounts:
        - name: datadir
          mountPath: /var/lib/zookeeper
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
  volumeClaimTemplates:
  - metadata:
      name: datadir
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

1. Crea los objetos:
```
kubectl apply -f app/zookeeper.yaml
```

1. Verifica que se han creado correctamente:
```
kubectl get services
kubectl get poddisruptionbudget
kubectl get statefulsets
```

1. Verifica que los Pods del StatefulSet se están creando ordenadamente:
```
kubectl get pods -l app=zk -w
```

1. Verifica los detalles del Servicio:
```
kubectl describe services cassandra -n lab-statefulsets-cassandra
```

    Una vez que el pod `zk-2` esté en estado `Running` puedes detener el comando "watch" con `CTRL + C`.

## Facilitar la elección del líder
Cada servidor en el conjunto de Zookeeper necesita un identificador único, asociado con una dirección de red, que todos los servidores necesitan conocer.

Revisa los hostname de los Pods del StatefulSet:

```
for i in 0 1 2; do kubectl exec zk-$i -- hostname; done
```

Los servidores de Zookeeper usan enteros como identificadores únicos y guardan el identificador de cada servidor en un archivo `myid` en el directorio de datos.

Examina dicho archivo en cada servidor:

```
for i in 0 1 2; do echo "myid zk-$i; kubectl exec zk-$i -- cat /var/lib/zookeeper/data/myid; done
```

Lista el FQDN de cada Pod:

```
for in in 0 1 2; do kubectl exec zk-$i -- hostname -f; done
```

El K8s DNS resuelve los FQDN. Si el Pod es redesplegado en otro lugar, los registros A del DNS no cambiarán, sólo serán actualizados con las nuevas direcciones IP de los Pods.

Revisa el archivo de configuración de la aplicación:

```
kubectl exec zk-0 -- cat /opt/zookeeper/conf/zoo.cfg
```

## Alcanzando consenso
Los protocolos de consenso requieren que los identificadores sean únicos para que puedan confirmar qué procesos han enviado qué datos.

Los registros A para cada Pod se crean cuando el Pod llega al estado `Ready`, por lo que los FQDN apuntan a un único endpoint, un único servidor.

Ésto se consigue gracias al despliegue escalonado de Pods del StatefulSet.

### Comprobando el conjunto
Vamos a escribir datos en un servidor Zookeeper y leerlos desde otro para confirmar que todo está desplegado correctamente.

1. Escribe el valor `world` en el path `/hello` en el Pod `zk-0`:
```
kubectl exec zk-0 zkCli.sh create /hello world
```

1. Lee el valor desde el Pod `zk-1`:
```
kubectl exec zk-1 zkCli.sh get /hello
```

Ésto verificará que los datos que hemos escrito a `zk-0` están disponibles en todos los servidores del conjunto.

## Proporcionando almacenamiento duradero
Zookeeper mantiene todas las entradas en un WAL duradero y escribe instantáneas de memoria a un medio de almacenamiento para conseguir replicar el estado.

1. Elimina el StatefulSet:
```
kubectl delete statefulset zk
```

1. Vigila la eliminación de los Pods del StatefulSet:
```
kubectl get pods -l app=zk -w
```

    Usa `CTRL + C` cuando se haya eliminado el último Pod, `zk-0`.

1. Reaplica el manifiesto para recrear el StatefulSet:
```
kubectl apply -f app/zookeeper.yaml
```

1. Vigila los Pods mientras son recreados:
```
kubectl get pods -l app=zk -w
```

1. Recupera el dato que escribiste antes de eliminar el StatefulSet:
```
kubectl exec zk-2 zkCli.sh get /hello
```

    Podrás comprobar como los datos han persistido en el Volumen Persistente.

1. Comprueba el PersistentVolumeClaim de cada Pod:
```
kubectl get pvc -l app=zk
```

Cuando un Pod en el StatefulSet es recreado, mantiene el mismo PersistentVolume asociado montado en el directorio de datos.

## Sobreviviendo a eventos de mantenimiento
Vamos a planificar ante el caso de eventos de mantenimiento que provoquen fallos en un Nodo.

1. Lista los Nodos del clúster:
```
kubectl get nodes
```

1. Revisa el PodDisruptionBudget:
```
kubectl get pdb zk-pdb
```

1. Vigila los Pods del StatefulSet:
```
kubectl get pods l app=zk -w
```

1. Abre otra pestaña de terminal en Cloud Shell y comprueba los Nodos en los que están desplegados los Pods:
```
for i in 0 1 2; do kubectl get pod zk-$i --template {{.spec.nodeName}}; echo ""; done
```

1. En la 2ª terminal, acordona y deshaucia el Nodo en el que está desplegado el Pod `zk-0`:
```
kubectl drain $(kubectl get pod zk-0 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-local-data
```

    El pod `zk-0` será redesplegado en otro de los 3 Nodos disponibles.

1. Continúa vigialndo los Pods en la 1ª terminal y deshaucia el Nodo en el que está desplegado el Pod `zk-1`:
```
kubectl drain $(kubectl get pod zk-0 --template {{.spec.nodeName}}) --ignore-daemonsets --force --delete-local-data
```

    El Pod no puede ser redesplegado por la regla de podAntiAffinity, por lo que queda en estado de `Pending`.

## Para terminar

### Eliminar el clúster utilizado
```
gcloud container clusters delete lab-zookeeper --zone europe-west1-c
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo desplegar un conjunto de Zookeeper usando un StatefulSet
- Cómo configurar el conjunto usando ConfigMaps
- Cómo expandir el despliegue de servidores de Zookeeper en el conjunto
- Cómo usar PodDisruptionBudgets para asegurar la disponibilidad del servicio durante el mantenimiento programado