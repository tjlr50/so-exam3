# so-exam3

**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Estudiante:** Tomas Lemus  
**Tema:** Descubrimiento de servicios, Microservicios  
**Correo:** tjlr50@gmail.com

**URL:** https://github.com/tjlr50/so-exam3

### Objetivos
* Implementar servicios web que puedan ser consumidos por usuarios o aplicaciones
* Conocer y emplear tecnologías de descubrimiento de servicio

### punto 3

Para el despliegue del esquema mosrado en la figura, fue necesirario implementar un balancador de carga, consul, descrubridor de servicios y tres servidores que ejecutaran un servicio cada uno. Esto se realiza de la siguiente forma:



Al comenzar, para los servidores se debe crear el servicio, ejecutando un agente consul. Para conseguir un funcionnamiento optimo, se requieren las siguientes instalaciones.

![][1]

```
# yum install -y wget unzip
# wget https://bootstrap.pypa.io/get-pip.py -P /tmp
# python /tmp/get-pip.py
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
# adduser consul
# passwd consul
# chown -R consul:consul /etc/consul
# chown -R consul:consul /etc/consul.d
```

Luego, para ejecutar el microservicio, se crea un ambiente asociado y un usuario de microservicio, en el cual se ejecutará un agente-flask; es necesario habilitar ciertos puertos para poder ejecutar al microservicio.

Se habilitan los puertos y se procede a la implementación.

```
$ sudo firewall-cmd --permanent --zone=public --add-port=8181/tcp
$ sudo firewall-cmd --reload

# adduser microservices
# passwd microservices

# su microservices
$ mkvirtualenv microservice_a
$ work-on microservice_a
```

Se crea y se ejectue el archivo en pyhton

Crear un archivo de configuración para el microservicio con un healthcheck
```
# su consul
$ echo '{"service": {"name": "nombreDelMicroservicio", "tags": ["flask"], "port": 8080,
  "check": {"script": "curl localhost:8080/health >/dev/null 2>&1", "interval": "10s"}}}' >/etc/consul.d/nombreDelMicroservicio.json
```

Iniciar el agente en modo cliente (use una sesión de screen)

```
# su consul
$ consul agent -data-dir=/etc/consul/data -node=agent-one \ -bind=Mi_IP -enable-script-checks=true -config-dir=/etc/consul.d
$ consul join IP_DiscoveryService
```

![][2] 
![][4]
 

Para continuar con el descubridor de servicios, se debe realizar la siguiente instalación:

```
# yum install -y wget unzip
# wget https://releases.hashicorp.com/consul/1.0.0/consul_1.0.0_linux_amd64.zip -P /tmp
# unzip /tmp/consul_1.0.0_linux_amd64.zip -d /tmp
# mv /tmp/consul /usr/bin
# mkdir /etc/consul.d
# mkdir -p /etc/consul/data
```

Para la imprementación de los servicios, sSe procede a crear el consul user y habilitar los puertos necesarios

```
# adduser consul
# passwd consul
# chown -R consul:consul /etc/consul
# chown -R consul:consul /etc/consul.d

# firewall-cmd --zone=public --add-port=8301/tcp --permanent
# firewall-cmd --zone=public --add-port=8300/tcp --permanent
# firewall-cmd --zone=public --add-port=8500/tcp --permanent
# firewall-cmd --reload
```

Se continúa con el descubridor de servicios, el cual requiere un consul server (quien administra los clientes), que se ejecuta de la siguiente forma:

![][5] 
![][6]


Se inicia el consul server

```
# su consul
$ consul agent -server -bootstrap-expect=1 \
    -data-dir=/etc/consul/data -node=agent-server -bind=IP_DiscoveryService \
    -enable-script-checks=true -config-dir=/etc/consul.d -client 0.0.0.0
```

Se muestran los consul members del descubridor de servicios

![][7]


Para asignar un servidor a una determinada solicitud de cliente, se configura un balanceador de carga a través de la instalación de HAProxy y la configuracion del archivo haproxy.cfg

```
$ sudo yum info haproxy
$ sudo yum install gcc pcre-static pcre-devel -y
$ wget https://www.haproxy.org/download/1.7/src/haproxy-1.7.8.tar.gz -O ~/haproxy.tar.gz
$ tar xzvf ~/haproxy.tar.gz -C ~/
$ cd ~/haproxy-1.7.8
$ make TARGET=linux2628
$ sudo make install
$ sudo mkdir -p /etc/haproxy
$ sudo mkdir -p /var/lib/haproxy 
$ sudo touch /var/lib/haproxy/stats
$ sudo cp ~/haproxy-1.7.8/examples/haproxy.init /etc/init.d/haproxy
$ sudo chmod 755 /etc/init.d/haproxy
$ sudo systemctl daemon-reload
$ sudo chkconfig haproxy on
$ sudo useradd -r haproxy
```

Ahora, se reinicia el HAProxy para cargar la configuración previa

```
$ sudo vi /etc/haproxy/haproxy.cfg
$ sudo systemctl restart haproxy
```

![][8]
![][9]

Finalmente, para automatizar esta función se configura el consul template al microservisio con su IP correspondiente.

![][10]


4. Al tener varios microservicios al tiempo, el balanceador de cargas, por medio del llamando round robin, que recorre las solicitudes de clientes, asignando un respectivo servidor, uno tras otro, para mantener en lo posible un número de servicios asignados a cada servidor. 

![][11]

![][13]

![][15]

5. SI lo que se busca es una escalabilidad para futuros posibles servicios asociables, con el fin de conseguir un manejo autónomo del sistema, recomendaría implementar un API GATEWAY, de tal forma que los servicios accedan a este en busca de un microservicio y consuman el API respectivo. Con esto se obtendría un sistema de gran atonomía y escalabilidad a nuevos microservicios.













[1]: 1.PNG

[2]: 2.PNG

[3]: 3.PNG

[4]: 4.PNG

[5]: 5.PNG


[6]: 6.PNG


[7]: 7.PNG


[8]: 8.PNG


[9]: 9.PNG


[10]: 10.PNG


[11]: 11.PNG

[13]: 13.PNG

[14]: 14.PNG

[15]: 15.PNG
