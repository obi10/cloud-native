# Cloud Native Fast Track - Oracle
Microservicios - DevOps - IaC

'Hands on' en las soluciones del portafolio de 'Cloud Native Services' de Oracle.
Soluciones: Oracle Kubernetes Engine (OKE), Oracle Cloud Infrastructure Registry (OCIR), Developer Cloud Service, Resource Manager.
Tecnologías: Docker, Kubernetes, Git, Terraform.


## Estructura de directorios

* `microservices`
    * `microservices` Aquí están los archivos manifest .yaml.
    * Guía: Despliegue de un Apache HTTP Server en OKE.
* `devops`
    * Guía: Construcción y despliegue de una aplicación Node.js usando Developer Cloud Service.
* `iac`
    * `terraform` Aquí están los archivos de configuración de terraform.
    * Guía: Despliegue de una arquitectura de desarrollo usando Terraform.

### Prerrequisitos

1. Computadora con acceso a internet y herramientas de conexión SSH (putty, puttygen)
2. Cuenta trial OCI (https://profile.oracle.com/myprofile/account/create-account.jspx)

### Setup - Running the test

__Arquitecture of the solution__
![Oracle logo](https://github.com/obi10/cloud-native/blob/master/images/oracle_logo.png)

__Raspberry Pi - sensors__
![Oracle Developers logo](https://github.com/obi10/cloud-native/blob/master/images/oracle_developers_logo.png)

__Raspberry Pi__
1. Clone the repository `git clone https://github.com/obi10/iot_workshop.git`
2. Install java JDK, JRE (jdk-8u211-linux-x64.tar.gz)
3. Go to raspi_cliente folder and run the java program 'socket_client.java' specifying the number of connections `java socket_client <number_conn>`

__Computer__
1. Sign in to [OCI account](https://cloud.oracle.com)
2. Enter to Oracle Data Visualization Cloud Service
