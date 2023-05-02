# montando_nodo_nym


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


~~~
./nym-mixnode node-details --id cypherplatxs
~~~

Habilitar los puertos 1789, 1790 8000, 22, 80, 443

~~~
sudo ufw allow 1789,1790,8000,22,80,443/tcp
# check the status of the firewall
sudo ufw status
~~~





## servicios de vps

https://www.hostinger.co/vps-servidor-web



