# montando_nodo_nym

### Pasos preliminares

~~~
Tener ipv6 en tu router para conectarte
~~~

[Test de ipv6 ](https://ipv6-test.com/)

Te recomendamos abrir dos terminales para hacer las configuraciones mas agutso, una la usaremos para acceder al servidor vía ssh y la otra terminal la usaremos para enviar el binario del mixnode desde nuestra computadora hasta el servidor donde montaremos el nodo

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


### Especificaciones del VPS

Debes de solicitar un acceso a una ip4 primaria para no tener problemas para acceder

Deberá alquilar un VPS para ejecutar su nodo de mezcla. Una razón clave para esto es que tu nodo debe poder enviar datos TCP utilizando IPv4 e IPv6 ( como otros nodos con los que habla pueden usar cualquiera de los protocolos ).

Por el momento, no hemos hecho un gran esfuerzo para optimizar la concurrencia para aumentar el rendimiento, así que no se moleste en aprovisionar un servidor bestial con múltiples núcleos. Esto cambiará cuando tengamos la oportunidad de comenzar a hacer optimizaciones de rendimiento de una manera más seria. El descifrado de paquetes Sphinx está sujeto a la CPU, por lo que una vez que optimicemos, serán mejores núcleos más rápidos.

Por ahora, vea las especificaciones a continuación:

Procesadores: 2 núcleos están bien. Obtenga las CPU más rápidas que pueda pagar.
RAM: los requisitos de memoria son muy bajos; por lo general, un nodo de mezcla puede usar solo unos pocos cientos de MB de RAM.
Discos: los nodos mixtos no requieren espacio en disco más allá de unos pocos bytes para los archivos de configuración.



Sistema operativo Linux  Ubuntu  > 20.04



## Paso 1 

Abrir la wallet 

## paso 2 

Para usuarios de windows recmendamos usar wsl, terminal de unix en windows

Entrar al root del vps vía ssh 

~~~
ssh root@xxx.xxx.xxx.xx
~~~


## paso 3 

descargar el binario en el vps 

~~~
wget -c https://github.com/nymtech/nym/releases/download/nym-binaries-v2023.5-rolo/nym-mixnode
~~~


## paso 4 

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



## paso 5 

Verificar la version del binario que teenmos 

~~~
./nym-mixnode --version
~~~

## paso 6 
Creamos el archivo de informacion del servicio

~~~
nano /etc/systemd/system/nym-mixnode.service
~~~

ponemos dentro esta información

~~~
[Unit]
Description=Nym Mixnode (v1.1.36)
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=root
LimitNOFILE=65536
ExecStart=/root/nym-node run --mode mixnode --id noisk8 --deny-init --accept-operator-terms-and-conditions
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target

~~~

## paso 7 

Reiniciamos los demonios

~~~
systemctl daemon-reload
~~~


## paso 8 

Iniciamos el nodo 


~~~
./nym-mixnode init --id <nombre-nodo> --host <tuip>
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

## paso 9 

Hacer el bonding desde la wallet 

![doc-mixnode](https://user-images.githubusercontent.com/17709296/235567658-975dacd4-738b-42fe-9f5d-d238985e72c4.png)


## paso 10 

Ahora te pedira una firma 

~~~
./nym-mixnode sign --id cypherplatxs --contract-msg 5XrvVEMzRJk2AcT2h1o6ErZNb8z1ZzD3h7teipBW3NUtrtYq7vu4DRMgzZRTPVPnyr2YWCxpmKCMFaEXvksnJ
4jt7np3NMLxsLMrFjEBhh67Crtjy4868vCzAivUqzdc365RiqxQQKtv4r9eTk9mTbE9JY8U3TxzKJCSGcBqbrb9JX3HrZVWm6tqbUYbsnku9pqnfeyeUiaYKY44Lm72TYrkZfRrMAZLMATiXT1ntmiKqT37HzRxNZjiH8qHeQEoRHkgDsmXDXRbfppGTpPrN7R4sjynJzehzUBZ8Ug7ovT9FoAHb8kuVQhUiMs1js6tdwtthzQMbPi9vwxUtVvjYknN2fnJgMnckEhzJJpJDCNdH7YhpPaWQnGVVS334mskiuqkbRVrFPJN2nnwArHr3L2cLxSMk9toKfw7ViKJ2p5E5JxiSmKY1cFGZ7uRLsuQ833PJN9JE8crPtkBNefqkbFNz68S5jPmzUShSvAc4TqXKeovDASFmmhKaPqLUrfsSWm7nzuKnzJSMADF6xSuwr9cknMoirqkRkLe7ybJ2ERwSdf5cUxMjF7yjS8tW9hZudnTUb1uPNDuSmPPVrCR12XZyFzBvVgxH51ZNJTym46nqnfA881LQcmFMnCwJf39rVJ4ASLnzEzmuwXj75QoB9ce9kiLmoBNLYe4QKSB6gDd858VnBtBNQELVuCCZbrTYuSCeNdUFhvMwD4kryc1pBYUa8Ro81F3QVfiKN
~~~

## paso 11

Copie la firma resultante:

~~~
>>> The base58-encoded signature is:
2GbKcZVKFdpi3sR9xoJWzwPuGdj3bvd7yDtDYVoKfbTWdpjqAeU8KS5bSftD5giVLJC3gZiCg2kmEjNG5jkdjKUt
~~~
![firma](https://nymtech.net/docs/images/wallet-sign.png)


## paso 12 

pon una descripción al nodo 

Acá te va pedir un nombre, una descripción y un dominio el dominio puede ser el contacto con el cual las personas que hagan stake tengan contacto directo 

~~~
./nym-mixnode describe --id winston-smithnode
~~~


## paso 13 

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


## paso 14 

Habilitar los puertos 1789, 1790 8000, 22, 80, 443

~~~
sudo ufw allow 1789,1790,8000,22,80,443/tcp
# revisa el status del firewall
sudo ufw status
~~~


~~~
systemctl enable nym-mixnode.service
~~~

~~~
service nym-mixnode start
~~~

~~~
systemctl daemon-reload
~~~

# Actualizar un nodo 

[Vídeo cómo actualzar nodo](https://archive.org/details/nym-node-update)

Descarga la nueva versión del mix node 

[Nym Components](https://nymtech.net/download-nym-components/)

En una terminal ingresamos al servidor vía ssh 

~~~
ssh root@190.120.xxx.xxx
~~~


Paramos el proceso del nodo 

~~~
service nym-mixnode stop
~~~

Eliminamos el binario de mixnode que vamos a reemplazar 

~~~
rm nym-mixnode
~~~

Descargamos el binario 

~~~
wget -c https://github.com/nymtech/nym/releases/download/nym-binaries-v2023.5-rolo/nym-mixnode
~~~

Damos permisos de ejecucución al binario 

~~~
chmod +x nym-mixnode
~~~

Hacemos el comando init para inicializar el nodo de nuevo 

 ~~~
 ./nym-mixnode init --id  cypherplatxs --host  xxx.xx.xxx.xx 
 ~~~

en los campos de las xx va la ip del nodo 


En la wallet vamos a la configuración del nodo y cambiamos la versión del nodo por la que acabamos de actualizar 

![Screenshot_20230511_225628](https://github.com/Noisk8/montando_nodo_nym/assets/17709296/d729b7ab-aa13-4a79-9210-27b50291b99d)



cambiar el numero de la versión en el script et system 

~~~
cd /etc/systemd/system/

nano nym-mixnode.service
~~~

Altera la 2da linea del script y pon la version del nodo que haz actualizado 

~~~
Description=Nym Mixnode (v1.1.27)
~~~

~~~
systemctl daemon-reload
~~~

Reiniciamos el servicio 

~~~
service nym-mixnode restart
~~~

Para verificar que nuestro nodo quedo activo 

primero podemos hacer un ping al server

~~~
ping xx.xxx.xx.xx
~~~

y luego ver el explorador de [guru](https://mixnet.explorers.guru/) 



este comando para monitorear tu nodo

~~~
journalctl -f -u nym-mixnode.service
~~~


<details>
<summary> <h1><b> Cambiar la descripción del nodo </b> </h1></summary>

~~~
cd .nym/mixnodes/cypherplatxs/

cd config/

~~~


</details>

