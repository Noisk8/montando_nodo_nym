# montando_nodo_nym


## instalar dependencias 


~~~
sudo apt install pkg-config build-essential libssl-dev curl jq git
~~~

## Instalar rust 

~~~
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
~~~

mover el binario 

~~~
sudo cp mix-node root@200.58.105.37
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


~~~
./nym-mixnode init --id  cypherplatxs --host $(curl ifconfig.me) --wallet-address n1eufxdlgt0puwrwptgjfqne8pj4nhy2u5ft62uq
~~~


Hacer el bonding desde la wallet 


~~~
./nym-mixnode node-details --id cypherplatxs
~~~

H

sudo ufw allow 1789,1790,8000,22,80,443/tcp
# check the status of the firewall
sudo ufw status





## servicios de vps

https://www.hostinger.co/vps-servidor-web

