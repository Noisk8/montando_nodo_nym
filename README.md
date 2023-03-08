# montando_nodo_nym








## instalar dependencias 


~~~
sudo apt install pkg-config build-essential libssl-dev curl jq git


~~~

## Instalar rust 

~~~
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
~~

Preconstruir el binario

~~~
rustup update
git clone https://github.com/nymtech/nym.git
cd nym

git reset --hard # in case you made any changes on your branch
git pull # in case you've checked it out before

git checkout release/v1.1.12 # checkout to the latest release branch: `develop` will most likely be incompatible with deployed public networks

cargo build --release # build your binaries with **mainnet** configuration
NETWORK=sandbox cargo build --release # build your binaries with **sandbox** configuration

~~~


Descargar los componenstes de https://nymtech.net/download/




*  error al ejecutar el binario de mix-node


~~~
error while loading shared libraries: libssl.so.1.1: cannot open shared object file: No such file or directory
~~~

Solución 

~~~

wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2.17_amd64.deb

sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2.17_amd64.deb

~~~


