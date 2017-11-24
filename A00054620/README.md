### Examen 3
**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Docente:** Daniel Barragán C.  
**Tema:** Descubrimiento de servicios, Microservicios  
**Correo:** daniel.barragan at correo.icesi.edu.co
**Estudiante:** Juan Camilo Swan
**Código:** A00054620
**Correo:** juan.swan at correo.icesi.edu.co

### Objetivos
* Implementar servicios web que puedan ser consumidos por usuarios o aplicaciones
* Conocer y emplear tecnologías de descubrimiento de servicio

### Prerrequisitos
* Virtualbox o WMWare
* Máquina virtual con sistema operativo CentOS7
* Framework consul, zookeper o etcd

### Descripción
El tercer parcial del curso sistemas operativos trata sobre la creación de servicios web y el uso de tecnologías para el descubrimiento de servicio

![][1]
**Figura 1.** Despliegue básico de microservicios

### Actividades
1. Incluir nombre, código (5%)
2. Ortografía y redacción cuando sea necesario (5%)
3. Despliegue un esquema como el mostrado en la **figura 1**. Empleen un servicio web de su preferencia (puede usar alguno de los ejemplos de clase). No es necesario incluir los componentes para monitoreo (Elasticsearch, Kibana, Logstash) (30%)
4. Adicione un microservicio igual al ya desplegado. Muestre a través de evidencias como las peticiones realizadas al balanceador son dirigidas a la replica del microservicio (30%)
5. Describa los cambios o adiciones necesarias en el diagrama de la **figura 1** para adicionar un microservicio diferente al ya desplegado en el ambiente, tenga en cuenta los siguientes conceptos en su descripción: API Gateway, paradigma reactivo, load balancer, protocolo publicador/suscriptor (interconexión de microservicios) (20%)
6. El informe debe ser entregado en formato pdf a través del moodle y el informe en formato README.md debe ser subido a un repositorio de github. El repositorio de github debe ser un fork de https://github.com/ICESI-Training/so-exam3 y para la entrega deberá hacer un Pull Request (PR) respetando la estructura definida. El código fuente y la url de github deben incluirse en el informe (10%)  

### Desarrollo

**3.** Para el desarrollo de este parcial se implementó el esquema de la figura 1 utilizando un **balanceador de carga**, un servidor consul de **discovery service**
y por último se utilizaron tres servidores ejecutando un servicio(aunque era el mismo servicio, se desplegó un mensaje diferente para lograr evidenciar el
Funcionamiento del balanceador de carga). Los tres servidores inician sus servicios y se subscriben a los members del consul (discovery service). 
Posteriormente el balanceador de carga hace repetidas solicitudes al discovery service para enterarse de los nuevos servidores que estan prestando el servicio.
  
Cuando un cliente, en este caso es un navegador web, hace realiza una solicitud a cierto servicio, esta  no es realizada directamente sobre el servidor que se encarga
de dicho servicio. La solicitud se realiza sobre el balanceador de carga; quien se encarga de reconocer cuantos servidores se encuentran ejecutando el servicio y distribuir las peticiones
entre todos los servidores. Cuando un nuevo servidor es agregado al discovery service, mediante los **consul-templates**, el balanceador de carga es capaz de escribir automaticamente 
sobre su archivo de configuraciones y agregar este nuevo servidor a su lista de servidores disponibles.
  
A continuación se mostrará la forma de realizar el montaje de cada parte de la infraestructura (servidores, discovery service, balanceador de carga)

**Servidores:** Para realizar el montaje de los servidores se debe primero realizar instalar las dependencias necesarias, realizar la programación de un servicio, iniciar el agente de consul, abrir
los puertos necesarios en el firewall que se requieren para acceder al microservicio y los puertos necesarios en el firewall del agente consul.  
Ahora se debe crear un ambiente con el nombre del microservicio que se va a ejecutar, instalar el agente flask en el ambiente y en este momento ya 
se puede ejecutar el microservicio en el ambiente.  
Una vez que se está ejecutando el microservicio y se ha iniciado el agente consul, se debe agregar el servidor al consul members. Con seguir estos pasos se tiene un servidor ejecutando un servicio 
que se encuentra reportado en el sistema de discovery service.  
  
**Código del microservicio**  
  ![][2]  
  
**Creación del ambiente**  
  ![][3]  
  
**Puertos firewall**  
  ![][4]
  
**Servicio corriendo**  
  ![][5]

Para lograr esta implementación se deben seguir estos pasos:  
Instalar las dependencias necesarias  
```
# yum install -y wget unzip
# wget https://bootstrap.pypa.io/get-pip.py -P /tmp
# python /tmp/get-pip.py
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
```
Se recomienda ejecutar el agente de consul como el usuario consul
```
# adduser consul
# passwd consul
# chown -R consul:consul /etc/consul
# chown -R consul:consul /etc/consul.d
```
Abrir los puertos necesarios en el firewall para el agente de consul
```
# firewall-cmd --zone=public --add-port=8301/tcp --permanent
# firewall-cmd --reload
```
Abrir los puertos necesarios en el firewall que se requieren para acceder al microservicio
```
# firewall-cmd --zone=public --add-port=8080/tcp --permanent
# firewall-cmd --reload
```
Cree un usuario microservices para ejecutar el servicio
```
# adduser microservices
# passwd microservices
```
Cree un ambiente de nombre microservice_a y de ser necesario actívelo
```
# su microservices
$ mkvirtualenv microservice_a
$ work-on microservice_a
```
Instale la librería flask en el ambiente, cree y ejectue el script microservice_a.py
```
$ pip install flask
$ vi microservice_a.py
```
Iniciar el microservicio (use una sesión de screen)
```
$ python microservice_a.py
```
Crear un archivo de configuración para el microservicio con un  healthcheck
```
# su consul
$ echo '{"service": {"name": "nombreDelMicroservicio", "tags": ["flask"], "port": 8080,
  "check": {"script": "curl localhost:8080/health >/dev/null 2>&1", "interval": "10s"}}}' >/etc/consul.d/nombreDelMicroservicio.json
```

Iniciar el agente en modo cliente (use una sesión de screen)
```
# su consul
$ consul agent -data-dir=/etc/consul/data -node=agent-one \
    -bind=Mi_IP -enable-script-checks=true -config-dir=/etc/consul.d
```
Una el cliente al ambiente de descubrimiento de servicio
```
$ consul join IP_DiscoveryService
```
Verifique la lista de miembros del ambiente
```
$ consul members
```  
  
**Discovery service:** Para realizar el montaje del discovery service basicamente se requiere instalar todas las depencias, instalar consul y ejecutar consul 
en modo consul server. (los servidores que ejecutan los servicios ejecutaron consul en modo consul client, en este caso ellos son clientes del servidor de discovery service.)  
Los clientes se unen al consul server y por lo tanto estos se ven agrupados aqui. Se pueden visualizar los servidores por medio de "consul members".  
Es importante que se habiliten los puertos por lo que el consul server funciona, de lo contrario habrá inconvenientes de acceso.
  
**Inicialización del consul agent en modo servidor**  
  ![][6]
  
**Consul agent en modo servidor en ejecución**  
  ![][7]
  
**Consul members del montaje**  
  ![][8]
  

Para lograr esta implementación se deben seguir estos pasos:  
Instalar las dependencias necesarias
```
# yum install -y wget unzip
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
```
Se recomienda ejecutar el agente de consul como el usuario consul
```
# adduser consul
# passwd consul
# chown -R consul:consul /etc/consul
# chown -R consul:consul /etc/consul.d
```
Abrir los puertos necesarios en el firewall para el agente de consul
```
# firewall-cmd --zone=public --add-port=8301/tcp --permanent
# firewall-cmd --zone=public --add-port=8300/tcp --permanent
# firewall-cmd --zone=public --add-port=8500/tcp --permanent
# firewall-cmd --reload
```
Iniciar el agente en modo servidor (use una sesión de screen)
```
# su consul
$ consul agent -server -bootstrap-expect=1 \
    -data-dir=/etc/consul/data -node=agent-server -bind=IP_DiscoveryService \
    -enable-script-checks=true -config-dir=/etc/consul.d -client 0.0.0.0
```
Para consultar los miembros del ambiente de descubrimiento de servicio
```
$ consul members
```  
  
**Balanceador de carga:** El balanceador se encarga de registrar los servidores que disponen de cierto servicio y atender las solicitudes de los clientes mediante estos servidores.
Es decir, el balanceador de carga es con quien el cliente se comunica. Para realizar el montaje del balanceador de carga se debe instalar todas las dependencias necesarias de HAProxy y de consul-template.  
Una vez que se han instalado dichas dependencias, se debe configurar los archivos que van a permitir realizar las funciones del balanceador.
Dicho archivo se encuentra ubicado en: "/etc/haproxy/haproxy.cfg", aquí se van a configurar los servidores y sus respectivas IP_Addres.
Sin embargo no es de buenas prácticas tener que configurar dicho archivo de forma manual cada vez que un servidor nuevo sea agregado. Por este motivo se tiene el dynamic service discovery.
Lo que se busca es que mediante el **consul-template**, cuando un servidor se reporte con el service discovery, el balanceador consulte los servidores en el service discovery y actualice automaticamente el archivo
"haproxy.cfg" automaticamente. Esto se logra configurando el consul-template en el archivo de configuración "/etc/consul-template/haproxy.tpl".  
  
**Configuración del balanceador**  
  ![][9]
  
**Balanceador corriendo**  
  ![][10]
  
**Configuración del consul-template**  
  ![][11]
  
Para realizar la implementación del balanceador de carga se debe seguir lo siguiente:  
Instalar dependencias
``` 
$ sudo yum info haproxy
$ sudo yum install gcc pcre-static pcre-devel -y
$ wget https://www.haproxy.org/download/1.7/src/haproxy-1.7.8.tar.gz -O ~/haproxy.tar.gz
$ tar xzvf ~/haproxy.tar.gz -C ~/
$ cd ~/haproxy-1.7.8
$ make TARGET=linux2628
$ sudo make install
```  
  
Configurar HAproxy:
``` 
$ sudo mkdir -p /etc/haproxy
$ sudo mkdir -p /var/lib/haproxy 
$ sudo touch /var/lib/haproxy/stats
$ sudo cp ~/haproxy-1.7.8/examples/haproxy.init /etc/init.d/haproxy
$ sudo chmod 755 /etc/init.d/haproxy
$ sudo systemctl daemon-reload
$ sudo chkconfig haproxy on
$ sudo useradd -r haproxy
```  
Abrir puertos
```
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --permanent --zone=public --add-port=8181/tcp
$ sudo firewall-cmd --reload
```  
configure archivos y reload service
```
$ sudo vi /etc/haproxy/haproxy.cfg
$ sudo systemctl restart haproxy
```  
  
  
**4.** Desde un inicio se han tenido varios microservicios. Como se pudo observar en la imagen del archivo de configuración del balanceador,
este ya tiene consigo inscritos por medio del discovery service todos los servidores que prestan el servicio. Tambien se puede observar que el
balanceador está utilizando el método de balanceo **roundrobin**. Este método consiste en que si la anterior solicitud fue al servidor 1, la próxima será al 2, la próxima al 3
y así sucesivamente hasta dar la vuelta y volver al servidor 1. Existen varios tipos de balanceo y para este caso se utilizó el roundrobin, pero
tambien se pudó haber utilizado: **Weighted Round-Robin**, **LeastConnection**, **Weighted LeastConnection**, entre otros.

**visualización de peticiones y balanceo roundrobin**  
  ![][12]  
  
 
| ![][13] | ![][14] | ![][15] |
|------|------|------|  
  
  
**5.** Si se desea agregar un servicio diferente a la infraestructura de que se resolvio durante todo este informe se debe primero tener claro algunos conceptos que permiten 
generar buenas prácticas en el montaje de los microservicios.  
Para empezar sería parte de las buenas prácticas agregar un API-Gateway, con esto se logra la optimización en la codificación, ya que los servicios no deben ser
codificados en cada servidor. Sino que el API-Gateway tiene todos los códigos y los servidores lo que hacen es ir a el y consumir el API del microservicio.
Esto genera que los microservicios sean más fáciles de manejar, ya que si por ejemplo un servicio se encuentra en 5 servidores; no se va a necesitar codificar 5 veces el microservicio, 
se codifica una sola vez en el API-Gateway y los servidores consumen los API de los servicios que requieren. Por lo tanto
se implementa como un único punto de entrada para que todas las aplicaciones cliente consuman los APIs de backend y los servicios.  
Protocolo publicador/suscriptor, que es algo similar a lo que hace se hace con el servidor de discovery service por lo que es un término a tener en cuenta. Por 
otro lado va a ser necesario tener en cuenta que el balanceador de carga es efectivo para la ejecución de un servicio con varios servidores detras.

Por lo tanto si se quiere agregar un servicio diferente se requiere como minimo un nuevo balanceador de carga. Incluso el nuevo servicio 
se puede desplegar en los mismos servidores existentes pero ejecutandose por un puerto diferente. En este orden de ideas el proceso seria el siguiente:  
1. Implementar un nuevo servicio (sea en los servidores existentes ejecutandose en un puerto diferente, o en un servidor diferente).  
Dicha implementación puede ser directamente o se puede implementar un API-Gateway que realice el manejo de los servicios.  
2. registrar los nuevos servicios ante el consul server.
3. Implementar un nuevo balanceador de carga que se haga cargo del nuevo servicio.




### Referencias
https://github.com/ICESI/so-microservices-python  
http://microservices.io/patterns/microservices.html
https://www.upcloud.com/support/haproxy-load-balancer-centos/
https://github.com/ICESI/so-discovery-service/blob/master/README.md
http://sirile.github.io/2015/05/18/using-haproxy-and-consul-for-dynamic-service-discovery-on-docker.html
https://es.wikipedia.org/wiki/Balanceador_de_carga

[1]: images/1.png
[2]: images/2.png
[3]: images/3.png
[4]: images/4.png
[5]: images/5.jpg
[6]: images/6.png
[7]: images/7.png
[8]: images/8.png
[9]: images/9.png
[10]: images/10.png
[11]: images/11.png
[12]: images/12.png
[13]: images/13.png
[14]: images/14.png
[15]: images/15.png



