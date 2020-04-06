# Build and Deploy Node.js Microservice on Docker using Oracle Developer Cloud
Autor: Abhinav Shroff - Principal product manager of Developer Cloud Service
Link: https://blogs.oracle.com/developers/build-deploy-nodejs-microservice-on-docker-using-oracle-developer-cloud

Obs: Se recomienda tener instalado en su PC __Git Bash__

## Introducción
El blog explica la construcción de una imagen Docker que conteneriza una aplicación 'NodeJS REST microservice', para luego ser subida a DockerHub haciendo uso de la solucion Oracle Developer Cloud Service.

<!---imagen de portada--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

A continuación, se presentan las secciones en las que está dividido este artículo:
* __Configurar el entorno de Developer Cloud Service__
* __Crear un nuevo proyecto__
* __Crear un nuevo repositorio Git__
* __Setting up Build VM in Developer Cloud__

## Desarrollo
#### 1. Configurar el entorno de Developer Cloud Service
You must configure Compute & Storage before your projects will be fully functional. OCI Credentials.
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
Para llenar los valores requeridos por el developer cloud service.
Los valores del tenancy_ocid, user_ocid, compartment_ocid y storage namespace se obtienen de la consola de OCI.
Los valores de private key, passphrase y fingerprint son obtenidos al crear las llave .pem y agregarlas en API Key del usuario. Para generar las llaves se usa el Cloud Shell.
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
Cuando se ingresó al Cloud Shell, ejecutar los siguientes comandos para crear las llave pública y privada en formato .pem.
```sh
$ mkdir keys (crear una nueva carpeta en el home)
$ cd keys (ir a la nueva carpeta)
$ openssl genrsa -des3 -out private.pem 2048 (passphrase: private)
$ openssl rsa -in private.pem -outform PEM -pubout -out public.pem
$ ls (observar los archivos)
$ cat public.pem (mostrar el valor del archivo)
```
Copiar el valor de llave con ‘Ctrl + Insert’ en un block de nota (notepad), para después añadirla en API key del usuario.
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Copiar el valor del fingerprint en el block de notas.
fingerprint:
94:ae:b2:42:24:0c:e1:7b:a2:ae:48:7c:7e:38:b3:b4
Asimismo, se debe copiar al block de notas el valor de la llave privada.
```sh
$ cat private.pem (Ctrl +Insert para copiar el valor)
```

Los siguientes valores son requeridos al configurar el Developer Cloud Service.

*tenancy_ocid: ocid1.tenancy.oc1..aaaaaaaal7ryxbp2hgljainhn3xe67m3jec66exupxajvsjcd36y5sfot7kq*
*user_ocid: ocid1.user.oc1..aaaaaaaas2ol4ddk6yzmhabecxetak6cnqejwc7whhb7r567iiqxe7nvmpza*
*private key: <valor>*
*passphrase: private*
*fingerprint: <valor>*
*compartment_ocid: ocid1.compartment.oc1..aaaaaaaa57ptyitppgnwlp4e3dosnqv3ehqyb3cmbmtkcy6rndtj6qjnpzda*
*storage namespace: idu2plmeyir7*
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

#### 2. Crear un nuevo proyecto

<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Seleccionar Markdown y crear un nuevo proyecto. El entorno se aprovisiona exitosamente para empezar a trabajar.
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

#### 3. Crear un nuevo repositorio Git
Project Home > + Create Repository
*Repository Name: NodeJSDocker*
*Description: -*
*Initial content: Empty Repository*
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

#### 4. Crear un VM con Developer Cloud Service
Primero se debe crear un template de VM con los siguientes tools: Docker.
Organization > Virtual Machines Templates > + Create Template
*Name: DockerTemplate*
*Description: -*
*Platform: Oracle Linux 7*
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Click en > Configure Software
Agregar Docker 17.12 > Done
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Luego, aprovisionar una máquina virtual a partir del template creado previamente.
Build Virtual Machines > + Create VM
*Quantity: 1*
*VM Template: DockerTemplate*
*Region: us-ashburn-1*
*Shape: VM.Standard.E2.1*
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->

Start the Build VM
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->


#### 5. Pushing Scripts to Git Repository on Oracle Developer Cloud
En este paso se subirán algunos archivos al repositorio creado en la tercer parte. Estos archivos son propios de la aplicación.
Entonces, en su pc, se abre la consola de Git Bash:
Ctrl+S > escribir 'Git Bash' > Enter
<!---![architecture_using_oke](https://github.com/obi10/cloud-native/blob/master/images/architecture_using_oke.png)--->
Ejecutar los siguientes comandos para inicializar git:
```sh
$ mkdir NodeJSDocker (se crea una nueva carpeta de trabajo)
$ git init
```
En la carpeta de trabajo se crean tres archivos: Main.js, Package.json y Dockerfile.
Las líneas de código de los archivos son las siguientes:
__Main.js__
~~~~
var express = require("express");
var bodyParser = require("body-parser");
var app = express();
app.use(bodyParser.urlencoded());
app.use(bodyParser.json());
var router = express.Router();
router.get('/',function(req,res){
  res.json({"error" : false, "message" : "Hello Abhinav!"});
});
router.post('/add',function(req,res){
  res.json({"error" : false, "message" : "success", "data" : req.body.num1 + req.body.num2});
});
app.use('/',router);
app.listen(80,function(){
  console.log("Listening at PORT 80");
})
~~~~

__Package.json__
~~~~
{
  "name": "NodeJSMicro",
  "version": "0.0.1",
  "scripts": {
    "start": "node Main.js"
  },
  "dependencies": {
    "body-parser": "^1.13.2",
    "express": "^4.13.1"
    
 }
}
~~~~

__Dockerfile__
~~~~
FROM node:6
 
ADD Main.js ./
ADD package.json ./
 
RUN npm install
 
EXPOSE 80
 
CMD [ "npm", "start" ]
~~~~

Para subir los archivos al repositorio:
```sh
$ git add -A
$ git status
$ git commit –m “<some commit message>”
$ git remote add origin <Developer cloud Git repository HTTPS URL>
$ git push -u origin master
```


#### 6. Build Configuration
En este paso se subirán algunos archivos al repositorio creado en la tercer parte. Estos archivos son propios de la aplicación.
Entonces, en su pc, se abre la consola de Git Bash: