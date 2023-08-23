# montando_nodo_nym

### Pasos preliminares

Hay un par de pasos que deben completarse antes de comenzar a configurar su nodo de mezcla:

### preparando tu billetera

Antes de inicializar y ejecutar tu mixnode, visita nuestra página web y descarga el monedero Nym para tu sistema operativo. Si no dispones de los binarios precompilados para tu sistema operativo, puedes crear el monedero tú mismo siguiendo las instrucciones que encontrarás aquí.

Si aún no tienes uno, crea una dirección Nym utilizando el monedero y fúndalo con tokens. La cantidad mínima necesaria para enlazar un mixnode es de 100 NYM, pero asegúrate de tener un poco más para tener en cuenta los costes de gas.

NYM se puede comprar a través de Bity desde la propia cartera con BTC o fiat, y actualmente está presente en varios intercambios.

~~~
Recuerda que sólo puedes utilizar tokens Cosmos NYM para enlazar tu mixnode. No puedes utilizar representaciones ERC20 de NYM para ejecutar un nodo.
~~~

Un VPS ( Servidor privado virtual )
Preparación de la billetera


### Especificaciones de hardware de VPS

Deberá alquilar un VPS para ejecutar su nodo de mezcla. Una razón clave para esto es que tu nodo debe poder enviar datos TCP utilizando IPv4 e IPv6 ( como otros nodos con los que habla pueden usar cualquiera de los protocolos ).

Por el momento, no hemos hecho un gran esfuerzo para optimizar la concurrencia para aumentar el rendimiento, así que no se moleste en aprovisionar un servidor bestial con múltiples núcleos. Esto cambiará cuando tengamos la oportunidad de comenzar a hacer optimizaciones de rendimiento de una manera más seria. El descifrado de paquetes Sphinx está sujeto a la CPU, por lo que una vez que optimicemos, serán mejores núcleos más rápidos.

Por ahora, vea las especificaciones a continuación:

Procesadores: 2 núcleos están bien. Obtenga las CPU más rápidas que pueda pagar.
RAM: los requisitos de memoria son muy bajos; por lo general, un nodo de mezcla puede usar solo unos pocos cientos de MB de RAM.
Discos: los nodos mixtos no requieren espacio en disco más allá de unos pocos bytes para los archivos de configuración.


Sistema operativo Linux  Ubuntu  o debian preferiblemente 


### Descargar el binario 

Ve a [Nym Components](https://nymtech.net/download-nym-components/) y descarga el binario de nym mix-node


mover el binario al vps 

~~~
sudo scp mix-node root@200.58.105.37:/root
~~~

Entrar al root del vps vía ssh 

~~~
ssh root@xxx.xxx.xxx.xx
~~~

las xxx se cambia por la ip del vps que has contratado


actualisamos el sistema con el comando 

~~~
sudo apt update
~~~

Una vez adentro del vps listamos para revisar que si este el  binario de nym-mixnode

~~~
ls -la
~~~


damos permisos de ejecución al binario 

~~~
chmod u+x nym-mixnode
~~~

🍔 Problemas 

Al intentar ejecutar varios usuarios hemos experimentado este error...


*  error al ejecutar el binario de mix-node

~~~
error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory
~~~


Solución 

~~~
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.18_amd64.deb

sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.18_amd64.deb
~~~



Iniciar el nodo 

~~~
./nym-mixnode init --id  cypherplatxs --host $(curl ifconfig.me) --wallet-address n1ntzugxxxxxxxxxxxt4zfkrlu4am3mn2z
~~~



Para ver los detalles del nodo 

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



Ahora te pedira una firma 

~~~
./nym-mixnode sign --id cypherplatxs --contract-msg 5XrvVEMzRJk2AcT2h1o6ErZNb8z1ZzD3h7teipBW3NUtrtYq7vu4DRMgzZRTPVPnyr2YWCxpmKCMFaEXvksnJ
4jt7np3NMLxsLMrFjEBhh67Crtjy4868vCzAivUqzdc365RiqxQQKtv4r9eTk9mTbE9JY8U3TxzKJCSGcBqbrb9JX3HrZVWm6tqbUYbsnku9pqnfeyeUiaYKY44Lm72TYrkZfRrMAZLMATiXT1ntmiKqT37HzRxNZjiH8qHeQEoRHkgDsmXDXRbfppGTpPrN7R4sjynJzehzUBZ8Ug7ovT9FoAHb8kuVQhUiMs1js6tdwtthzQMbPi9vwxUtVvjYknN2fnJgMnckEhzJJpJDCNdH7YhpPaWQnGVVS334mskiuqkbRVrFPJN2nnwArHr3L2cLxSMk9toKfw7ViKJ2p5E5JxiSmKY1cFGZ7uRLsuQ833PJN9JE8crPtkBNefqkbFNz68S5jPmzUShSvAc4TqXKeovDASFmmhKaPqLUrfsSWm7nzuKnzJSMADF6xSuwr9cknMoirqkRkLe7ybJ2ERwSdf5cUxMjF7yjS8tW9hZudnTUb1uPNDuSmPPVrCR12XZyFzBvVgxH51ZNJTym46nqnfA881LQcmFMnCwJf39rVJ4ASLnzEzmuwXj75QoB9ce9kiLmoBNLYe4QKSB6gDd858VnBtBNQELVuCCZbrTYuSCeNdUFhvMwD4kryc1pBYUa8Ro81F3QVfiKN
~~~

Copie la firma resultante:

~~~
>>> The base58-encoded signature is:
2GbKcZVKFdpi3sR9xoJWzwPuGdj3bvd7yDtDYVoKfbTWdpjqAeU8KS5bSftD5giVLJC3gZiCg2kmEjNG5jkdjKUt
~~~
![firma](https://nymtech.net/docs/images/wallet-sign.png)


Ahora ejecuta el nodo 

~~~
./nym-mixnode run --id cypherplatxs
~~~

pon una descripción al nodo 

Acá te va pedir un nombre, una descripción y un dominio el dominio puede ser el contacto con el cual las personas que hagan stake tengan contacto directo 

~~~
./nym-mixnode describe --id winston-smithnode
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
Description=Nym Mixnode (v1.1.26)
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

[Vídeo cómo actualzar nodo](https://archive.org/details/nym-node-update)

Descarga la nueva versión del mix node 

Ingresamos al vps vía ssh 

Paramos el proceso del nodo 

~~~
service nym-mixnode stop
~~~

Eliminamos el binario de mixnode que vamos a reemplazar 

~~~
rm nym-mixnode
~~~


Copiamos el binario del mixnode al vps

~~~
scp nym-mixnode root@xxx.xx.xxx.xx:/root
~~~


luego le das los permisos de ejecución al binario 

~~~
chmod +x nym-mixnode
~~~

 ~~~
 ./nym-mixnode init --id  cypherplatxs --host  xxx.xx.xxx.xx 
 ~~~

en los campos de las xx va la ip del nodo 

### despues del binario 1.1.21 el comando --wallet no funca
 ~~~
 ./nym-mixnode init --id  cypherplatxs --host $(curl ifconfig.me) 
 ~~~

En la wallet vamos a la configuración del nodo y cambiamos la versión del nodo por la que acabamos de actualizar 

![Screenshot_20230511_225628](https://github.com/Noisk8/montando_nodo_nym/assets/17709296/d729b7ab-aa13-4a79-9210-27b50291b99d)



cambiar el numero de la versión en el script et system 

~~~
cd /etc/systemd/system/

nano nym-mixnode.service
~~~

Altera la 2da linea del script y pon la version del nodo que haz actualizado 

~~~
Description=Nym Mixnode (v1.1.21)
~~~

~~~
systemctl daemon-reload
~~~

Reiniciamos el servicio 

~~~
service nym-mixnode restart
~~~


