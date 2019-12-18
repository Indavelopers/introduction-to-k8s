author: Marcos Manuel Ortega
summary: Crear configMaps y Secrets
id: lab-configmaps-secrets
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Crear ConfigMaps y Secretos

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo crear ConfigMaps y Secretos de forma imperativa y con manifiestos
- Cómo consumir Secretos y ConfigMaps en Pods en variables de entorno o como Volúmenes montados

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
kubectl create namespace lab-configmaps-secrets
```

## Crear ConfigMaps
Vamos a crear ConfigMaps de 3 formas diferentes:
1. Usando `kubectl`, desde una clave-valor literal
2. Usando `kubectl`, desde un archivo
3. Usando un manifiesto YAML

### Usando kubectl, desde una clave-valor literal
1. Crea un ConfigMap desde una clave-valor literal:
```
kubectl create configmap demo-cm-literal --from-literal message=hello_world -n lab-configmaps-secrets
```

1. Comprueba la descripción del ConfigMap:
```
kubectl describe configmaps demo-cm-literal -n lab-configmaps-secrets
```

### Usando kubectl, desde un archivo
1. Crea el archivo `demo.properties` con el siguiente contenido:
```
foo=bar
meaning_of_life=42
song=always_look_on_the_bright_side_of_life
```

1. Crea un ConfigMap con el contenido del archivo:
```
kubectl create configmap demo-cm-file --from-file=demo.properties -n lab-configmaps-secrets
```

1. Comprueba la descripción del ConfigMap:
```
kubectl describe configmaps demo-cm-file -n lab-configmaps-secrets
```

### Usando un manifiesto YAML
1. Crea el archivo `volumes/demo-cm.yaml` con este manifiesto:
```
apiVersion: v1
kind: ConfigMap
metadata:
    name: demo-cm-manifest
data:
    spam: eggs
    knight: black
```

1. Crea un ConfigMap con el contenido del manifiesto:
```
kubectl apply -f volumes/demo-cm.yaml -n lab-configmaps-secrets
```

1. Comprueba la descripción del ConfigMap:
```
kubectl describe configmaps demo-cm-manifest -n lab-configmaps-secrets
```

1. Lista todos los ConfigMaps creados:
```
kubectl get configmaps -n lab-configmaps-secrets
```

## Usar los ConfigMaps en Contenedores

Vamos a crear un Despliegue con Pods donde montaremos la información contenida en uno de los ConfigMaps como un Volumen y la de otro ConfigMap como variables de entorno.

1. Crea el archivo `deployments/demo-cm.yaml` con este manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-configmap
spec:
  selector:
    matchLabels:
      app: demo-configmap
  template:
    metadata:
      labels:
        app: demo-configmap
    spec:
      containers:
      - name: hello-world
        image: gcr.io/google-samples/node-hello:1.0
        volumeMounts:
        - name: properties
          mountPath: /etc/config
        env:
        - name: SPAM
          valueFrom:
            configMapKeyRef:
              name: demo-cm-manifest
              key: spam
      volumes:
      - name: properties
        configMap:
          name: demo-cm-file
```

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-cm.yaml -n lab-configmaps-secrets
```

1. Verifica que el Pod está ejecutándose:
```
kubectl get pods -n lab-configmaps-secrets
```

1. Anota el nombre del Pod y verifica sus detalles:
```
kubectl describe pod [NOMBRE_POD] -n lab-configmaps-secrets
```

    Fíjate en sus Volúmenes y variables de entorno.

### Comprobar el contenido de los ConfigMap consumidos por el Contenedor
1. Conéctate al Pod ejecutando una sesión shell:
```
kubectl exec -it [NOMBRE_POD] -n lab-configmaps-secrets -- sh
```

1. Lista los contenidos del directorio del Volumen montado:
```
ls /etc/config
```

    Las claves del archivo con el que creamos el ConfigMap aparecerán como archivos separados.

1. Comprueba el contenido de dichos archivos:
```
cat /etc/config/demo.properties
```

1. Comprueba la variable de entorno asignada:
```
echo $SPAM
```

1. Sal de la sesión shell:
```
exit
```

## Crear Secretos
Del mismo modo, vamos a crear Secrets de las mismas 3 formas distintas:
1. Usando `kubectl`, desde una clave-valor literal
2. Usando `kubectl`, desde un archivo
3. Usando un manifiesto YAML

### Usando kubectl, desde una clave-valor literal
1. Crea un Secreto desde una clave-valor literal:
```
kubectl create secret generic demo-secret-literal --from-literal secret=secret_message -n lab-configmaps-secrets
```

1. Comprueba la descripción del Secreto:
```
kubectl describe secret demo-secret-literal -n lab-configmaps-secrets
```

### Usando kubectl, desde un archivo
1. Crea el archivo `secret.txt` con el siguiente contenido:
```
secret_key=QWERTYUIOPASDFGHJKLZXCVBNM
```

1. Crea un Secreto con el contenido del archivo:
```
kubectl create secret generic demo-secret-file --from-file=secret.txt -n lab-configmaps-secrets
```

1. Comprueba la descripción del Secreto:
```
kubectl describe secret demo-secret-file -n lab-configmaps-secrets
```

### Usando un manifiesto YAML
1. Crea el archivo `volumes/demo-secret.yaml` con este manifiesto:
```
apiVersion: v1
kind: Secret
metadata:
    name: demo-secret-manifest
type: Opaque
data:
    user: YWRtaW4=
    password: MWYyZDFlMmU2N2Rm
```

1. Crea un Secreto con el contenido del manifiesto:
```
kubectl apply -f volumes/demo-secret.yaml -n lab-configmaps-secrets
```

1. Comprueba la descripción del Secreto:
```
kubectl describe secret demo-secret-manifest -n lab-configmaps-secrets
```

1. Lista todos los Sescretos creados:
```
kubectl get secrets -n lab-configmaps-secrets
```

## Usar los Secretos en Contenedores
Del mismo modo que para los ConfigMaps, vamos a crear un Despliegue con Pods donde montaremos la información contenida en uno de los Secretos como un Volumen y la de otro Secreto como variables de entorno.

1. Crea el archivo `deployments/demo-secret.yaml` con este manifiesto:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-secret
spec:
  selector:
    matchLabels:
      app: demo-secret
  template:
    metadata:
      labels:
        app: demo-secret
    spec:
      containers:
      - name: hello-world
        image: gcr.io/google-samples/node-hello:1.0
        volumeMounts:
        - name: secrets
          mountPath: /etc/secrets
        env:
        - name: USER
          valueFrom:
            secretKeyRef:
              name: demo-secret-manifest
              key: user
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: demo-secret-manifest
              key: password             
      volumes:
      - name: secrets
        secret:
          secretName: demo-secret-file
```

    *No hay que decir que no debes usar un nombre de usuario y contraseña en un proyecto real de esta manera, ¿verdad?*

1. Crea el Despliegue:
```
kubectl apply -f deployments/demo-secret.yaml -n lab-configmaps-secrets
```

1. Verifica que el Pod está ejecutándose:
```
kubectl get pods -n lab-configmaps-secrets
```

1. Anota el nombre del Pod que consume los secretos y verifica sus detalles:
```
kubectl describe pod [NOMBRE_POD] -n lab-configmaps-secrets
```

    Fíjate en sus Volúmenes y variables de entorno.

### Comprobar el contenido de los Secretos consumidos por el Contenedor
1. Conéctate al Pod que consume los secretos ejecutando una sesión shell:
```
kubectl exec -it [NOMBRE_POD] -n lab-configmaps-secrets -- sh
```

1. Lista los contenidos del directorio del Volumen montado:
```
ls /etc/secrets
```

    Las claves del archivo con el que creamos el ConfigMap aparecerán como archivos separados.

1. Comprueba el contenido de dichos archivos:
```
cat /etc/secrets/secret.txt
```

1. Comprueba la variable de entorno asignada:
```
echo $USER
echo $PASSWORD
```

1. Sal de la sesión shell:
```
exit
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-configmaps-secrets
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo crear ConfigMaps y Secrets de forma imperativa y con manifiestos
- Cómo consumir Secretos y ConfigMaps en Pods en variables de entorno o como Volúmenes montados