author: Marcos Manuel Ortega
summary: Asignar Pods a Nodos concretos
id: lab-node-selection
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Asignar Pods a Nodos concretos

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo añadir etiquetas a un Nodo
- Cómo crear un Pod en un tipo de Nodo específico
- Cómo crear un Pod en un Nodo en concreto

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
kubectl create namespace lab-node-selection
```

## Añadir etiquetas a Nodos
Podemos añadir etiquetas a Nodos de la misma forma que a cualquier otro objeto en K8s, permitiéndonos así referenciarlo en el futuro.

1. Lista los Nodos disponibles en tu clúster y sus etiquetas por defecto:
```
kubectl get nodes --show-labels
```

    *¿Qué etiquetas asigna K8s por defecto a estos Nodos?*

1. Elige uno de los Nodos con nombre `[NOMBRE_NODO]` y añádele una etiqueta:
```
kubectl label nodes [NOMBRE_NODO] disktype=ssd
```

1. Verifica que la etiqueta se ha asignado correctamente:
```
kubectl get nodes [NOMBRE_NODO] --show-labels
```

## Crea un Pod en un tipo de Nodo específico
Vamos a crear un Pod en un tipo de Nodo específico, usando una etiqueta asignada al Nodo, `disktype=ssd`.

1. Crea el archivo `pods/pod-node-selection.yaml` con este manifiesto:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd       
```

1. Crea el Pod:
```
kubectl apply -f pods/pod-node-selection.yaml -n lab-node-selection
```

1. Verifica que está ejecutándose en el Nodo correcto:
```
kubectl get pods nginx -n lab-node-selection -o wide
```

## Crea un Pod en un Nodo en concreto
En este caso vamos a crear un Pod en un Nodo en concreto, usando el nombre del Nodo.

1. Vuelve a listar los Nodos disponibles y escoge uno de ellos:
```
kubectl get nodes
```

1. Crea el archivo `pods/pod-node-selection2.yaml` con este manifiesto, sustituyendo `[NOMBRE_NODO]` por el nombre del Nodo seleccionado:
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeName: [NOMBRE_NODO]
```

1. Crea el Pod:
```
kubectl apply -f pods/pod-node-selection2.yaml -n lab-node-selection
```

1. Verifica que está ejecutándose en el Nodo correcto:
```
kubectl get pods nginx-2 -n lab-node-selection -o wide
```

## Para terminar

### Eliminar los recursos usados
Elimina el namespace, lo cual eliminará todos los recursos:
```
kubectl delete namespace lab-node-selection
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo añadir etiquetas a un Nodo
- Cómo crear un Pod en un tipo de Nodo específico
- Cómo crear un Pod en un Nodo en concreto