# Construcción y despliegue de una aplicación Golang usando Oracle Developer Cloud Service (DevCS)
Workshop basado en el blog titulado [Build and Deploy a Golang Application Using Oracle Developer Cloud]( https://blogs.oracle.com/developers/build-and-deploy-a-golang-application-using-oracle-developer-cloud).<br/>
Obs: Se asume que ya se tiene una instancia de DevCS previamente creada. Asimismo, se recomienda tener instalado __Git Bash__ en su PC.

## Introducción
Esta guía muestra los pasos de desarrollo, construcción y despliegue de su primera aplicación REST basada en Golang usando Docker y Kubernetes en Oracle Developer Cloud Service.
<!---imagen de portada--->
![devcs_ws_portada](https://github.com/obi10/cloud-native/blob/master/images/devcs/devcs_ws_portada.png)

A continuación, se presentan las secciones en las que está dividido el presente workshop:
* __Configurar el entorno de Developer Cloud Service__
* __Crear un nuevo proyecto__
* __Crear un nuevo repositorio Git__
* __Crear un VM con Developer Cloud Service__
* __Subir archivos al repositorio Git__
* __Crear Jobs__
* __Crear un Pipeline__

## Desarrollo
#### 1. Configurar el entorno de Developer Cloud Service
Primero se configura el entorno de Developer Cloud Service con parámetros relacionados a las credenciales de sus cuenta OCI y a la autenticación del usuario administrador.<br/>
Organization > OCI Account
![oci_account](https://github.com/obi10/cloud-native/blob/master/images/devcs/oci_account.png)
Los valores de *tenancy_ocid*, *user_ocid*, *compartment_ocid* y *storage namespace* se obtienen de la consola de OCI. Por otro lado, los valores de *private key*, *passphrase* y *fingerprint* son obtenidos al crear las llave .pem y agregarla la llave pública como API Key del usuario (para generar las llaves se usa la herramienta Cloud Shell).<br/>
Tip: Se recomienda copiar estos valores en un block de notas (Notepad).
![cloud_shell](https://github.com/obi10/cloud-native/blob/master/images/devcs/cloud_shell.png)
Dentro del Cloud Shell, ejecutar los siguientes comandos para crear las llave pública y privada en formato .pem.
```sh
$ mkdir keys (crear una nueva carpeta en el home)
$ cd keys_aux (ir a la nueva carpeta)
$ openssl genrsa -out private.pem 2048 (creación de la llave privada .pem sin contraseña)
$ openssl rsa -in private.pem -outform PEM -pubout -out public.pem (obtención de la llave pública .pem)
$ ls (observar los archivos)
$ cat public.pem (mostrar el valor del archivo public.pem)
```
El valor de llave public.pem se añade como API key del usuario.<br/>
![api_key_user](https://github.com/obi10/cloud-native/blob/master/images/devcs/api_key_user.png)
Se debe tener en cuentra que la llave creada no tiene contraseña, ya que en la configuración del ocicli (job: DeployGoRESTAppl) no es permitido ingresar el parámetro de contraseña de la llave.

Los siguientes valores son requeridos al configurar el Developer Cloud Service:<br/>
*__tenancy_ocid__: ocid1.tenancy.oc1..aaaaaaaal7ryxbp2hgljainhn3xe67m3jec66exupxajvsjcd36y5sfot7kq*<br/>
*__user_ocid__: ocid1.user.oc1..aaaaaaaas2ol4ddk6yzmhabecxetak6cnqejwc7whhb7r567iiqxe7nvmpza*<br/>
*__private key__: <valor>*<br/>
*__passphrase__:*<br/>
*__fingerprint__: b6:d5:1d:9f:3e:b8:93:fc:d1:35:09:bf:67:e5:0c:48*<br/>
*__compartment_ocid__: ocid1.compartment.oc1..aaaaaaaa57ptyitppgnwlp4e3dosnqv3ehqyb3cmbmtkcy6rndtj6qjnpzda*<br/>
*__storage namespace__: idu2plmeyir7*

#### 2. Crear un nuevo proyecto
Se crea un nuevo proyecto:<br/>
Organization > Projects > + Create<br/>
*Name: __ProyectoDevCS__*
![new_project_1](https://github.com/obi10/cloud-native/blob/master/images/devcs/new_project_1.png)
*Template: Empty Project*
![new_project_2](https://github.com/obi10/cloud-native/blob/master/images/devcs/new_project_2.png)
*Properties: Markdown*
![new_project_3](https://github.com/obi10/cloud-native/blob/master/images/devcs/new_project_3.png)
Click en Create y se aprovisiona el entorno exitosamente para empezar a trabajar.

#### 3. Crear un nuevo repositorio Git
Ingresar al proyecto > Project Home > + Create Repository<br/>
*Repository Name: __GoREST__*<br/>
*Description:*<br/>
*Initial content: Empty Repository*
![new_repository](https://github.com/obi10/cloud-native/blob/master/images/devcs/new_repository.png)
Click en Create y se obtiene un nuevo repositorio en Oracle Cloud.

#### 4. Crear un VM con Developer Cloud Service
Primero se debe crear un template de VM con los siguientes tools: Docker, OCIcli (requiere Python3 3.6), Kubectl.<br/>
Organization > Virtual Machines Templates > + Create Template<br/>
*Name: __MicroserviceTemplate__*<br/>
*Description:*<br/>
*Platform: Oracle Linux 7*
![vm_template](https://github.com/obi10/cloud-native/blob/master/images/devcs/vm_template.png)
Click en Configure Software.<br/>
Agregar Docker 17.12 > Agregar OCIcli > Agregar Python3 3.6 > Agregar Kubectl > Done
![vm_template_configure_software](https://github.com/obi10/cloud-native/blob/master/images/devcs/vm_template_configure_software.png)

Luego, aprovisionar una máquina virtual a partir del template creado previamente.<br/>
Build Virtual Machines > + Create VM<br/>
*Quantity: 1*<br/>
*VM Template: MicroserviceTemplate*<br/>
*Region: us-ashburn-1*<br/>
*Shape: VM.Standard.E2.2*
![build_vm](https://github.com/obi10/cloud-native/blob/master/images/devcs/build_vm.png)
Start the Build VM.
![build_vm_start](https://github.com/obi10/cloud-native/blob/master/images/devcs/build_vm_start.png)

#### 5. Subir archivos al repositorio Git
En este paso se subirán algunos archivos al repositorio creado en la tercera parte. Estos archivos son propios de la aplicación y construcción de la imagen Docker.<br/>
Entonces, en su PC, se abre la consola de Git Bash:<br/>
Ctrl+S > escribir 'Git Bash' > Enter<br/>
En mi caso, haré uso del propio terminal de Linux Ubuntu.
Ejecutar los siguientes comandos para clonar el repositorio y entrar en este localmente:
```sh
$ git clone https://junior.palomino%40oracle.com@devops-okepro.developer.ocp.oraclecloud.com/devops-okepro/s/devops-okepro_proyecto1_16911/scm/GoREST.git
$ cd GoREST
```
![terminal_1](https://github.com/obi10/cloud-native/blob/master/images/devcs/terminal_1.png)
En la carpeta GoREST se crean tres archivos: main.go, Dockerfile y gorest.yml.
![terminal_2](https://github.com/obi10/cloud-native/blob/master/images/devcs/terminal_2.png)
Las líneas de código de los archivos son las siguientes:<br/>
__main.go__
~~~~
package main
import (
    "fmt"
    "log"
    "net/http"
    "os"
)
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello %s!", r.URL.Path[1:])
    fmt.Println("RESTfulServ. on:8093, Controller:",r.URL.Path[1:])
}
func main() {
    http.HandleFunc("/", handler)
    fmt.Println("Starting Restful services...")
    fmt.Println("Using port:8093")
    err := http.ListenAndServe(":8093", nil)
    log.Print(err)
    errorHandler(err)
}
func errorHandler(err error){
if err!=nil {
    fmt.Println(err)
    os.Exit(1)
}
}
~~~~

__Dockerfile__
~~~~
FROM golang:latest 
RUN mkdir /app 
ADD . /app/ 
WORKDIR /app 
RUN go build -o main . 
CMD ["/app/main"]
~~~~

__gorest.yml__
~~~~
apiVersion: v1
kind: Service
metadata:
  name: gorest-se
  labels:
    app: gorest-se
spec:
  type: NodePort
  selector:
    app: gorest-se
  ports:
  - port: 8093
    targetPort: 8093
    nodePort: 30091
    name: http
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gorest-se
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: gorest-se
    spec:
      containers:
      - name: gorest-se
        image:  iad.ocir.io/idu2plmeyir7/go-rest:v1
        imagePullPolicy: Always
        ports:
        - containerPort: 8093
      imagePullSecrets:
        - name: ocirsecret
~~~~

Para subir los archivos al repositorio Git del Developer Cloud Service:
```sh
$ git add -A
$ git status
$ git commit –m “first commit”
$ git push -u origin master (password: account password)
```
![terminal_3](https://github.com/obi10/cloud-native/blob/master/images/devcs/terminal_3.png)

#### 6. Conectar a Oracle Cloud Infrastructure Registry (OCIR)
Para observar las imágenes del OCIR desde el DevCS:<br/>
Docker > New External Registry
![ocir](https://github.com/obi10/cloud-native/blob/master/images/devcs/ocir.png)

#### 7. Crear Jobs
Se crea un primer job __BuildGoRESTAppl__<br/>
Builds > + Create Job<br/>
*Name: BuildGoRESTAppl*<br/>
*Template: MicroserviceTemplate*
![job_1](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_1.png)
Git<br/>
*Repository: GoREST.git*
*Branch: Master*
![job_1_git](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_1_git.png)
Steps<br/>
Add Step - __Docker login__, __Docker build__, __Docker push__<br/>
__Docker login__<br/>
*Registry Host: iad.ocir.io*<br/>
*Username: idu2plmeyir7/junior.palomino@oracle.com*<br/>
*Password:*<Auth Token>
![job_1_docker_login](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_1_docker_login.png)
__Docker build__<br/>
*Registry Host: iad.ocir.io*<br/>
*Image Name: idu2plmeyir7/go-rest (verificar el nombre completo de la imagen 'iad.ocir.io/idu2plmeyir7/go-rest:v1' en el archivo gorest.yml)*<br/>
*Version Tag: v1*<br/>
*Source: Context Root in Workspace*
![job_1_docker_build](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_1_docker_build.png)
__Docker push__<br/>
*Registry Host: iad.ocir.io*<br/>
*Image Name: idu2plmeyir7/go-rest*<br/>
*Version Tag: v1*
![job_1_docker_push](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_1_docker_push.png)

Click en Save para guardar las configuraciones del primer job.

Se crea otro job __DeployGoRESTAppl__<br/>
Builds > + Create Job<br/>
*Name: DeployGoRESTAppl*
*Template: MicroserviceTemplate*
![job_2](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_2.png)
Git<br/>
*Repository: GoREST.git*<br/>
*Branch: Master*
![job_2_git](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_2_git.png)
Steps<br/>
Add Step - __OCIcli__, __Unix Shell__<br/>
__OCIcli__<br/>
*User OCID: ocid1.user.oc1..aaaaaaaas2ol4ddk6yzmhabecxetak6cnqejwc7whhb7r567iiqxe7nvmpza*<br/>
*Fingerprint: b6:d5:1d:9f:3e:b8:93:fc:d1:35:09:bf:67:e5:0c:48*<br/>
*Tenancy: ocid1.tenancy.oc1..aaaaaaaal7ryxbp2hgljainhn3xe67m3jec66exupxajvsjcd36y5sfot7kq*<br/>
*Private Key:*<value><br/>
*Region: us-ashburn-1*
![job_2_ocicli](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_2_ocicli.png)
__Unix Shell__
```sh
mkdir -p $HOME/.kube
# Acceder al cluster de Kubernetes a través del 'kubeconfig' (este comando es proporcionado por la consola OCI)
oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.iad.aaaaaaaaaezgcmzwgy4wiyrsg5rgczbzhfstqmjsgy3dkytggcywgzbtmvtg --file $HOME/.kube/config --region us-ashburn-1 --token-version 2.0.0
export KUBECONFIG=$HOME/.kube/config

if kubectl get deployment gorest-se; then
    kubectl set image deployment/gorest-se gorest-se=iad.ocir.io/idu2plmeyir7/go-rest:$tag --record;
else
    # A Kubernetes cluster uses the Secret of docker-registry type to authenticate with a container registry to pull a private image (modificar el comando)
    kubectl create secret docker-registry <secret-name> --docker-server=<region-code>.ocir.io --docker-username='<tenancy-namespace>/<oci-username>' --docker-password='<oci-auth-token>' --docker-email='<email-address>';
fi

kubectl apply -f gorest.yml
sleep 30
kubectl get services gorest-se
kubectl get pods
kubectl describe pods
```
![job_2_unixshell](https://github.com/obi10/cloud-native/blob/master/images/devcs/job_2_unixshell.png)

#### 8. Crear un Pipeline
Builds > Pipeline > + Create Pipeline<br/>
*Name: __GoApplPipeline__*
![create_pipeline](https://github.com/obi10/cloud-native/blob/master/images/devcs/create_pipeline.png)
Arrastrar los jobs BuildGoRESTAppl y DeployGoRESTAppl build jobs y después conectarlos.
![pipeline_drag_drop](https://github.com/obi10/cloud-native/blob/master/images/devcs/pipeline_drag_drop.png)
Doble click en el link que une los jobs y seleccionar Successful. Luego click en Apply.
![pipeline_link](https://github.com/obi10/cloud-native/blob/master/images/devcs/pipeline_link.png)
Click en Save.

Click en el botón de Build, como es mostrado en la figura de abajo, para correr el pipeline. El job BuildGoRESTAppl será ejecutado primero, si la ejecución resulta exitosa, luego el job DeployGoRESTAppl desplegar la aplicación REST escrita en Golang en un contenedor orquestado por Oracle Kubernetes.
![pipeline_build_button](https://github.com/obi10/cloud-native/blob/master/images/devcs/pipeline_build_button.png)

Para observar como funciona el REST Service, ingresar en el browser la siguiente url:
http://<ip_worker_node>:30091/<nombre>
![browser](https://github.com/obi10/cloud-native/blob/master/images/devcs/browser.png)

Listo!! Usted acaba de crear satisfactoriamente un REST Service en Oracke Kubernetes Engine usando Oracle Developer Cloud Service.


#### Posibles errores
Failed to pull image "iad.ocir.io/idu2plmeyir7/go-rest:latest": rpc error: code = Unknown desc = pull access denied for iad.ocir.io/idu2plmeyir7/go-rest, repository does not exist or may require 'docker login'.<br/>
Link: https://github.com/kubernetes/kubernetes/issues/24903<br/>
Tip: en el archivo .yml se recomienda tagear la imagen con una etiqueta diferente a latest.<br/>
![error_1](https://github.com/obi10/cloud-native/blob/master/images/devcs/error_1.png)
