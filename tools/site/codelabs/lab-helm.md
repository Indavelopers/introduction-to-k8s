author: Marcos Manuel Ortega
summary: Desplegando Jobs y CronJobs
id: lab-helm
categories: introduction-to-k8
environments: Web
status: Published
feedback link: https://www.vectoritcgroup.com

# Desplegando charts de Helm

## Resumen del ejercicio

¿Qué vamos a ver?

### What You’ll Learn 
- Cómo instalar Helm en K8s
- Cómo desplegar SW y paquetes con charts de Helm

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

## Instalar Helm
Vamos a descargar el binario de Helm y configurar la cuenta de servicio necesaria e inicializarlo.

1. Descarga el script de instalación de Helm:
```
curl -LO https://git.io/get_helm.sh
```

1. Haz el script ejecutable:
```
chmod 700 get_helm.sh
```

1. Ejecuta el script de instalación:
```
./get_helm.sh
```

1. Asocia tu cuenta al rol de cluster-admin en el clúster:
```
kubectl create clusterrolebinding user-admin-binding --clusterrole cluster-admin --user $(gcloud config get-value account)
```

1. Crea una cuenta de servicio para Tiller. Helm es la herramienta de cliente y Tiller la de servidor para Helm:
```
kubectl create serviceaccount tiller -n kube-system
```

1. Asocia la cuenta de servicio de Tiller al rol de cluster-admin:
```
kubectl  create clusterrolebinding tiller-admin-binding --clusterrole cluster-admin --serviceaccount kube-system:tiller
```

1. Inicializa Helm con la cuenta de servicio de Tiller (ésto puede tardar unos minutos):
```
helm init --service-account tiller
```

1. Verifica la instalación de Helm:
```
helm version
```

1. Actualiza los repositorios de charts de Helm:
```
helm repo update
```

## Despliega Redis con una chart de Helm

1. Inspecciona las plantillas de la chart de Helm antes de desplegarlas:
```
helm install stable/redis --dry-run --debug
```

1. Usando una chart de Helm, despliega una instancia de [Redis](https://github.com/helm/charts/tree/master/stable/redis) en tu clúster de K8s:
```
helm install stable/redis
```

1. Comprueba los recursos desplegados por Helm:
```
kubectl get services
kubectl get statefulsets
kubectl get configmaps
kubectl get secrets
```

## Comprueba el despliegue de Redis
1. Anota la IP del servicio del máster de Redis en una variable de entorno:
```
export IP_REDIS=$(kubectl get services -l app=redis -o json | jq -r '.items[].spec | select(.selector.role=="master")' | jq -r '.clusterIP')
echo $IP_REDIS
```

1. Anota la contraseña del máster de Redis en una variable de entorno:
```
export PASSWORD_REDIS=$(kubectl get secret -l app=redis -o jsonpath="{.items[0].data.redis-password}"  | base64 --decode)
echo $PASSWORD_REDIS
```

1. Ejecuta una sesión shell en un Pod temporal para comprobar el despliegue de Redis:
```
kubectl run redis-test-pod --rm -it --env REDIS_PW=$PASSWORD_REDIS --env REDIS_IP=$IP_REDIS --image docker.io/bitnami/redis:4.0.12 -- bash
```

1. En la sesión shell interactiva, conéctate al clúster Redis:
```
redis-cli -h $REDIS_IP -a $REDIS_PW
```

1. Establece una clave de prueba y su valor:
```
set demo-key demo-value
```

1. Recupera el valor de la clave de prueba:
```
get demo-key
```

## Para terminar

### Eliminar los recursos usados
Elimina los recursos creados por Helm
```
helm delete --purge redis
```

En esta práctica hemos visto cómo:

### What we've covered
- Cómo instalar Helm en K8s
- Cómo desplegar SW y paquetes con charts de Helm