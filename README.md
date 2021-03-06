# Kubernetes Jaeger Monitoreo :tophat::mag_right::bar_chart:
Jaeger es un sistema de software open source que sirve para detectar operaciones entre los servicios distribuidos. Se utiliza para supervisar entornos complejos de microservicios y solucionar los problemas asociados a ellos. 

En esta gu铆a se instala el Operador de Jaeger, recomendado para ambientes de producci贸n y se prueba su funcionalidad de monitoreo para aplicaciones de Python, NodeJS y Java.
<br />

## Tabla de contenido 馃搼

1. [Requisitos](#Requisitos-newspaper)
2. [Instalaci贸n del operador de Jaeger](#Instalaci贸n-del-operador-de-Jaeger-computer)
3. [Implementaci贸n y monitoreo de una aplicaci贸n de prueba en Python](#Implementaci贸n-y-monitoreo-de-una-aplicaci贸n-de-prueba-en-Python-1234)
4. [Implementaci贸n y monitoreo de una aplicaci贸n de prueba en Java - NodeJS](#Implementaci贸n-y-monitoreo-de-una-aplicaci贸n-de-prueba-en-Java---NodeJS-a-b)
5. [Referencias](#Referencias-mag)
6. [Autores](#Autores-black_nib)

## Requisitos :newspaper:
- Tener un servicio **[Kubernetes Cluster (IKS)](https://cloud.ibm.com/kubernetes/clusters)** disponible en la cuenta IBM Cloud.

  **Importante:** Debe ser un Cl煤ster **pago** en plan **Standard**, puede ser en infraestructura cl谩sica o *VPC*.

- :cloud: [IBM Cloud CLI](https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started&locale=en)
- :whale: [Docker](https://www.docker.com/products/docker-desktop)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/). La version de esta herramienta debe ser compatible con la version de IKS que se despleg贸 en la cuenta.
- Complemento [container-service/kubernetes-service](https://cloud.ibm.com/docs/cli?topic=cli-install-devtools-manually) para ibmcloud CLI. `ibmcloud plugin install container-service/kubernetes-service`.
- Tener instalado git.

## Instalaci贸n del operador de Jaeger :computer:

Para trabaja Jaeger en un ambiente de producci贸n, lo m谩s recomendable es instalar el operador Jaeger, a continuaci贸n se detallan los pasos para instalar Jaeger en su cl煤ster de Kubernetes, utilizando las plantillas utilizadas en la documentaci贸n oficial de Jaeger:

1.  Cree la definici贸n de recurso personalizada requerida por el operador de Jaeger:

```
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/crds/jaegertracing.io_jaegers_crd.yaml
```
2. A continuaci贸n, cree una cuenta de servicio , un rol y una vinculaci贸n de roles para el control de acceso basado en roles:

```
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/service_account.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role.yaml
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/role_binding.yaml
```
3. Finalmente, implemente el operador Jaeger:

```
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/master/deploy/operator.yaml
```
4. Aunque ya se tenga instalado el operador, es necesario crear un recurso que describa la instancia de Jaeger:

```
kubectl apply -f - <<EOF
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simplest
EOF
```
5. Para validar que Jaeger est谩 funcionando correctamente, ejecute el siguiente comando y verifique si puede acceder a la interfaz de usuario:

```
kubectl port-forward $(kubectl get pods -l=app="jaeger" -o name) 16686:16686
```

 <p align=center><img width="950" src=".github/openjaeger.PNG"></p>
 <br />
Por ahora el unico servicio que debe ver es el de Jaeger Query, ahora procederemos a implementar la aplicaci贸n que deseamos monitorear.
 

## Implementaci贸n y monitoreo de una aplicaci贸n de prueba en Python :1234:
Dicha aplicaci贸n ha sido copiada del siguiente [tutorial](https://www.digitalocean.com/community/tutorials/how-to-implement-distributed-tracing-with-jaeger-on-kubernetes) y consiste en una aplicaci贸n sencilla que ser谩 un contador de visitas cuyo valor aumenta cada vez que se realiza un llamado al frontend. Para simular problemas de rendimiento, incluye una funci贸n de suspensi贸n aleatoria que se ejecuta cada vez que el frontend env铆a una GET solicitud al backend. Esta aplicaci贸n adem谩s incluye la configuraci贸n necesaria de OpenTracing que necesita Jaeger para monitorear la aplicaci贸n. A continuaci贸n se detallan los pasos para desplegar la aplicaci贸n en el cl煤ster de Kubernetes:

1. Clone el repositorio de donde ser谩 tomada la aplicaci贸n:
```
git clone https://github.com/emeloibmco/Kubernetes-Jaeger-Monitoreo/sammy-jaeger.git
```

2. Dirijase a la carpeta ```sammy-jaeger``` que se encuentra en la carpeta clonada de este repositorio. A continuaci贸n asegurese que haya iniciado sesi贸n en DockerHub con el siguiente comando:

```
docker login --username=your_username --password=your_password
```
3. Ejecute 

```
nano ./frontend/deploy_frontend.yaml
```
O el comando que le permita editar el archivo. Cambie la direcci贸n ```username``` por su usuario de docker.

 <p align=center><img width="950" src=".github/console1.png"></p>
 <br />
 
 Pulse Ctrl+S para guardar los cambios. Y Ctrl+X para salir del editor. 
 
 4. Ejecute 

```
nano ./backend/deploy_backend.yaml
```
O el comando que le permita editar el archivo. Cambie la direcci贸n ```username``` por su usuario de docker.

 <p align=center><img width="950" src=".github/console2.png"></p>
 <br />
 
 5. A continuaci贸n se crear谩 la imagen en Docker, ejecute los siguiente comandos:
(Recuerde cambiar username por su usuario de DockerHub)

Para crear el backend:
 
 ```
docker build -t username/do-visit-counter-backend:v2 ./backend
docker push username/do-visit-counter-backend:v2
 ```
Para crear el frontend:

```
docker build -t username/do-visit-counter-frontend:v2 ./frontend
docker push username/do-visit-counter-frontend:v2

```
 6. Ahora enviaremos la aplicaci贸n al cl煤ster de kubernetes:
 
 ```
kubectl apply -f ./frontend/deploy_frontend.yaml
kubectl apply -f ./backend/deploy_backend.yaml

 ```
7. Para abrir la aplicaci贸n ejecute:

```
kubectl port-forward $(kubectl get pods -l=app="do-visit-counter-frontend" -o name) 8000:8000
```

 <p align=center><img width="950" src=".github/openapp.PNG"></p>
 <br />
 
8. Abra la aplicaci贸n desde el browser con la direcci贸n http://localhost:8000.

 <p align=center><img width="700" src=".github/app.PNG"></p>
 <br />
 
En otra terminal no olvide estar corriendo la aplicaci贸n de Jaeger, abra la consola de Jaeger con la direcci贸n http://localhost:16686. Debes poder visualizar el servicio ```service``` que corresponde a la aplicaci贸n desplegada.

 <p align=center><img width="950" src=".github/jaeger.PNG"></p>
 <br />

9. A continuaci贸n en otra terminal ejecute solicitudes a la aplicaci贸n para posteriormente observar el monitoreo.

```
for i in 0 1 2 3 4 5 6 7 8 9; do curl localhost:8000; done
```
10. Una vez concluido el ciclo del paso anterior, ingrese a la consola de Jaeger y complete lo siguiente:

* ```Service```: service
* ```Operation```: Puede escoger cualquiera de los servicio de la aplicaci贸n ```hello_world``` o ```counter```. O en este caso escogeremos la opci贸n ```all``` para visualizar ambos servicios.
* ```Tags```: Esta opci贸n le permite filtrar por la respuesta que fue entregada al cliente al momento de hacer la solicitud a la aplicaci贸n. Ejm: http.status_code=200.
* ```Lookback```: Puede elegir visualizar las solicitudes hechas en las 煤timas horas, seg煤n necesite puede elegir entre un rango de la 煤ltima hora y las 煤ltimas 48 horas.
* ```Max Duration/Min Duration ```: Debido a que Jaeger le entrega el tiempo en que tardo la aplicaci贸n en responder a la solictud, usted puede filtrar entre los tiempos que haya tardado, que sean de su interes. 
* ```Limit Results```: Con esta opci贸n indique a Jaeger cuantos resultados desea listar en el monitoreo que va a realizar.

Finalmente de click en ``` Find Traces``` para que se muestren los resultados. En el caso de la aplicaci贸n de ejemplo, usted podr谩 visualizar una primera gr谩fica donde se comparan las solicitudes realizadas de acuerdo a su tiempo de respuesta. Y posteriormente un resumen del tiempo y la respuesta de cada solicitud realizada por el cliente.

 <p align=center><img width="950" src=".github/monitoreo.gif"></p>
 <br />

## Implementaci贸n y monitoreo de una aplicaci贸n de prueba en Java - NodeJs :a: :b:
Dicha aplicaci贸n ha sido copiada del siguiente [tutorial](https://tracing.cloudnative101.dev/docs/ocp-jaeger.html) y consiste en una aplicaci贸n que consta de dos servicios (servicio-a y servicio-b), el servicio-a envia una petici贸n de saludo al servicio-b el cual constesta el saludo con el parametro de nombre ingresado por el cliente al momentode realizar la solicitud al servicio-b, esta aplicaci贸n tambien incluye una funci贸n de suspensi贸n que se ejecuta cada tres solicitudes realizadas al servicio-a. Esta aplicaci贸n adem谩s incluye la configuraci贸n necesaria de OpenTracing que necesita Jaeger para monitorear la aplicaci贸n. A continuaci贸n se detallan los pasos para desplegar la aplicaci贸n en el cl煤ster de Kubernetes:

1. Clone el repositorio de donde ser谩 tomada la aplicaci贸n:
```
git clone https://github.com/ibm-cloud-architecture/learning-distributed-tracing-101.git

```
2. Dirijase a la carpeta ```lab-jaeger-java``` o  ```lab-jaeger-nodejs```, dependiendo de cual de las dos quiera ejecutar, que se encuentra en la carpeta clonada de este repositorio. A continuaci贸n asegurese que haya iniciado sesi贸n en DockerHub con el siguiente comando:

```
docker login --username=your_username --password=your_password
```
3. Ejecute 

```
cd solution
```
 
 4. A continuaci贸n se crear谩 la imagen en Docker, ejecute los siguiente comandos:
(Recuerde cambiar username por su usuario de DockerHub)

Para crear el servicio-a:
 
 ```
docker build -t username/service-a:v1 ./service-a
docker push username/service-a:v1
 ```
Para crear el servicio-b:

```
docker build -t username/service-b:v1 ./service-b
docker push username/service-b:v1

```

5. Regrese a la carpeta principal del proyecto y ejecute:

```
cd lab-jaeger-ocp
```
6. A continuaci贸n, ejecute el siguiente comando para editar el archivo de configuraci贸n ```jaeger-java.yaml``` o ```jaeger-nodejs.yaml```:

```
nano jaeger-nodejs.yaml
```
O el comando que le permita editar el archivo.

Una vez ingrese al archivo cambie el ```username``` por su usuario de DockerHub, agregue la extensi贸n :v1 al nombre de la imagen y el ```value``` del ```JAEGER_ENDPOINT``` establezcalo en : "http://simplest-collector:14268/api/traces". Realice esto para el deployment de ambos servicios.

<p align=center><img width="950" src=".github/service-a.PNG"></p>
 <br />

<p align=center><img width="950" src=".github/service-b.PNG"></p>
 <br />

Pulse Ctrl+S para guardar los cambios. Y Ctrl+X para salir del editor. 

7. Ahora enviaremos la aplicaci贸n al cl煤ster de kubernetes:
 
 ```
kubectl apply -f ./jaeger-nodejs.yaml

 ```
8. Para abrir la aplicaci贸n ejecute:

```
kubectl port-forward $(kubectl get pods -l=app="service-a" -o name) 8080:8080
```

 <p align=center><img width="950" src=".github/openapp2.PNG"></p>
 <br />
 
9. Abra la aplicaci贸n desde el browser con la direcci贸n http://localhost:8080/sayHello/nombre.

 <p align=center><img width="700" src=".github/app2.PNG"></p>
 <br />
 
En otra terminal no olvide estar corriendo la aplicaci贸n de Jaeger, abra la consola de Jaeger con la direcci贸n http://localhost:16686. Debes poder visualizar el servicio ```service-a``` y ```service-b``` que corresponde a la aplicaci贸n desplegada. Elija visualizar ```service-a``` y evidencie la traza correspondiente a la solicitud que realizo previamente, observando su comunicaci贸n con el servicio-b, el tiempo de respuesta y tipo de solicitud realizada.
 
 <p align=center><img width="700" src=".github/jaeger1.gif"></p>
 <br />


10. A continuaci贸n ejecute una solicitud de error:
```
 curl http://localhost:8080/error -I HTTP/1.1 500
```
 Y visualice la traza en Jaeger.
 
 <p align=center><img width="700" src=".github/jaeger2.gif"></p>
 <br /> 

 
 
11. Ejecute el siguiente ciclo para enviar varias solicitudes a la aplicaci贸n:

```
for i in 0 1 2 3 4 5 6 7 8 9; do curl localhost:8080/sayHello/<nombre>; done
```
Una vez concluido el ciclo anterior, ingrese a la consola de Jaeger y visualice las distintas trazas, con el tiempo de respuesta y detalle.

 <p align=center><img width="950" src=".github/jaeger3.gif"></p>
 <br />

 12. Finalmente en la consola de Jaeger, ingrese a System Architecure > DAG y visualice la arquitectura de la aplicaci贸n.
 
 <p align=center><img width="950" src=".github/jaeger4.PNG"></p>
 <br />
 
 ## Referencias :mag:

- [Operador Jaeger](https://www.jaegertracing.io/docs/1.26/operator/)
- [How To Implement Distributed Tracing with Jaeger on Kubernetes ](https://www.digitalocean.com/community/tutorials/how-to-implement-distributed-tracing-with-jaeger-on-kubernetes)
- [Using Jaeger in OpenShift/Kubernetes](https://tracing.cloudnative101.dev/docs/ocp-jaeger.html)



<br />

## Autores :black_nib:
Equipo *IBM Cloud Tech Sales Colombia*.

