# Despliegue de un Apache HTTP Server en Oracle Kubernetes Engine (OKE)

## Introducción
En este documento se expone una serie de pasos para realizar el despliegue de un Apache HTTP Server en Oracle Kubernetes Engine (OKE), con el fin de obtener un servicio contenerizado y orquestado por la solución de Kubernetes disponible en Oracle Cloud Infrastructure (OCI).

Obs: Para el correcto entendimiento del ejercicio se debe tener claro conceptos básicos sobre: Oracle Cloud Infrastructure (OCI), Apache HTTP Server, Microservicios, Docker y Kubernetes.

## Visión General
En el siguiente gráfico se puede observar la arquitectura a implementar a alto nivel:
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Todos los componentes deben ser creados en la infraestructura cloud proporcionada por Oracle.
A continuación, se presentan las secciones en las que está dividido este artículo:
* __Aprovisionamiento y configuración del OKE en OCI__
* __Despliegue del Apache HTTP Server en el clúster Kubernetes (Cloud Shell)__

## Desarrollo
#### 1. Aprovisionamiento y configuración del OKE en OCI
Primero, se debe agregar la siguiente política en el compartimiento raíz:
*allow service OKE to manage all-resources in tenancy*
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Después, se crea una instancia de OKE.
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->



#### 2. Despliegue del Apache HTTP Server en el clúster Kubernetes (Cloud Shell)
Para el despliegue del servidor de aplicaciones en el clúster de OKE aprovisionado, se usa la nueva función de línea de comandos de OCI, llamado Cloud Shell. Cloud Shell básicamente es el terminal de una máquina virtual que tiene instalado *oci cli*, *git*, *docker* y *kubectl*.
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

En el terminal se procede a bajar la imagen docker *httpd* y se verifican que images de docker se tiene.
```sh
$ docker pull httpd
$ docker images
```
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Seguidamente, se sube la imagen al repositorio local imágenes docker en OCI, conocido com Oracle Cloud Infrastructure Registry (OCIR).
*Object Storage Namespace ​ : idu2plmeyir7*
*Auth Token ​ : W<Qoe#QNs}OdOvA5BCD4*

```sh
$ docker login -u idu2plmeyir7/junior.palomino@oracle.com iad.ocir.io ​ (password es el auth token)
$ docker tag httpd iad.ocir.io/idu2plmeyir7/httpd-k8s:v1
$ docker push iad.ocir.io/idu2plmeyir7/httpd-k8s:v1
```

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Se verifica que la imagen se subió correctamente al OCIR y se cambia a público.

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Después, se tiene que crear el archivo ​ __*Kubeconfig*__​, archivo de acceso al clúster de Kubernetes. El instructivo se encuentra en la propia consola, específicamente en la interfaz​ de ​la instancia OKE creada.

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Se descarga el repositorio de github que alberga los archivos .​ yaml ​ necesarios para desplegar la aplicación.
```sh
$ git clone https://github.com/obi10/httpd_k8s.git
$ cd http _k8s
```

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Se debe modificar el nombre del namespace (idu2plmeyir7) en el siguiente archivo:
*app_kubernetes.yaml*

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
Luego ejecutar los siguientes comandos:
```sh
$ kubectl get all
$ kubectl apply -f app_kubernetes.yaml
```

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
```sh
$ kubectl apply -f service.yaml
$ kubectl get all
```

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
ip pública del servicio: 150.136.0.161
Introducir “​ <ip pública de su servicio>:8080 ” ​ en el browser para visualizar el mensaje de “It works”.

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->