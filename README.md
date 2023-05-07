# montando_nodo_nym

### Pasos preliminares

Hay un par de pasos que deben completarse antes de comenzar a configurar su nodo de mezcla:

### preparando tu billetera

Un VPS ( Servidor privado virtual )
Preparación de la billetera


### Especificaciones de hardware de VPS

Deberá alquilar un VPS para ejecutar su nodo de mezcla. Una razón clave para esto es que tu nodo debe poder enviar datos TCP utilizando IPv4 e IPv6 ( como otros nodos con los que habla pueden usar cualquiera de los protocolos ).

Por el momento, no hemos hecho un gran esfuerzo para optimizar la concurrencia para aumentar el rendimiento, así que no se moleste en aprovisionar un servidor bestial con múltiples núcleos. Esto cambiará cuando tengamos la oportunidad de comenzar a hacer optimizaciones de rendimiento de una manera más seria. El descifrado de paquetes Sphinx está sujeto a la CPU, por lo que una vez que optimicemos, serán mejores núcleos más rápidos.

Por ahora, vea las especificaciones a continuación:

Procesadores: 2 núcleos están bien. Obtenga las CPU más rápidas que pueda pagar.
RAM: los requisitos de memoria son muy bajos; por lo general, un nodo de mezcla puede usar solo unos pocos cientos de MB de RAM.
Discos: los nodos mixtos no requieren espacio en disco más allá de unos pocos bytes para los archivos de configuración.



### Descargar el binario 

Ve a [Nym Components](https://nymtech.net/download-nym-components/) y descarga el binario de nym mix-node


mover el binario al vps 

~~~
sudo scp mix-node root@200.58.105.37/root
~~~

Entrar al root del vps vía ssh 

~~~
ssh root@200.58.105.37
~~~

actualisamos el sistema 

~~~
sudo apt update
~~~

Una vez adentro del vps listamos para revisar que si este el  binario de nym-node

~~~
ls -la
~~~


damos permisos de ejecución al binario 

~~~
chmod u+x nym-mixnode
~~~



*  error al ejecutar el binario de mix-node

~~~
error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory
~~~


Solución 

~~~
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.17_amd64.deb

sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.17_amd64.deb
~~~



Iniciar el nodo 

~~~
./nym-mixnode init --id  cypherplatxs --host $(curl ifconfig.me) --wallet-address n1ntzug277pnc5pge07d8882t4zfkrlu4am3mn2z
~~~


~~~
./nym-mixnode node-details --id cypherplatxs
~~~



Debes de obteneter el Ident Key y el  Sphinx Key para ponerlos en la wallet y hacer el bonding

~~~
 Identity Key: DMkSeKp3Zq5nC7SH1fiwDQmS1LFdxq2oEr2ZPeCSxxGG
Sphinx Key: A6BFTQsuTWFuQhnY4k7d6ehGgPRfAa3cSEk3cTpbK6As
Host: 65.108.110.214 (bind address: 65.108.110.214)
Version: 1.1.17
Mix Port: 1789, Verloc port: 1790, Http Port: 8000


You are bonding to wallet address: n1eufxdlgt0puwrwptgjfqne8pj4nhy2u5ft62uq
~~~

Hacer el bonding desde la wallet 

![doc-mixnode](https://user-images.githubusercontent.com/17709296/235567658-975dacd4-738b-42fe-9f5d-d238985e72c4.png)

~~~
./nym-mixnode run --id cypherplatxs
~~~


### Configura tu firewall

Los siguientes comandos le permitirán configurar un firewall usando ufw.

~~~
# revisa si tienes instalado  ufw 
ufw version
# si no está instalado, instalalo con
sudo apt install ufw -y
# habilita ufw
sudo ufw enable
# revisa el status del firewall
sudo ufw status
~~~



Habilitar los puertos 1789, 1790 8000, 22, 80, 443

~~~
sudo ufw allow 1789,1790,8000,22,80,443/tcp
# revisa el status del firewall
sudo ufw status
~~~


### Automatizar su nodo de mezcla con systemd

Es útil que el nodo de mezcla se inicie automáticamente en el momento del arranque del sistema. Aquí hay un archivo de servicio systemd para hacer eso:

~~~

cd /etc/systemd/system/

sudo nano nym-mixnode.service
~~~

pega el sigueinte codigo dentro de este archivo 

~~~
[Unit]
Description=Nym Mixnode (v1.1.17)
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=root
LimitNOFILE=65536
ExecStart=/root/nym-mixnode run --id cypherplatxs
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
~~~


en nano guarda con ctrl + o y cierra con ctrl x, y luego habilita el servicio con ↓↓

~~~
systemctl enable nym-mixnode.service
~~~

~~~
service nym-mixnode start
~~~

~~~
systemctl daemon-reload
~~~

### Actualizar un nodo 

Descarga la nueva versión del mix node 

Ingresamos al vps vía ssh 




Paramos el proceso del nodo 

~~~
service nym-mixnode stop
~~~


Copiamos el bibnario del mixnode al vps

~~~
sudo scp mix-node root@200.58.105.37/root
~~~

En la wallet vamos a la configuración del nodo y cambiamos la versión del nodo por la que acabamos de actualizar 



Reiniciamos el servicio 

~~~
service nym-mixnode restart
~~~


