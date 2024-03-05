Conseguir el vps 

## acceder al vps

~~~
ssh root@123.123.45
~~~

## descargar el binario 

~~~
wget https://github.com/nymtech/nym/releases/download/nym-binaries-v2024.1-marabou/nym-gateway
~~~
## permiso al binario 

~~~
chmod +x nym-gateway
~~~


## init

~~~
./nym-gateway init --id <ID> --listening-address 0.0.0.0 --public-ips "$(curl -4 https://ifconfig.me)" --with-network-requester --with-exit-policy true

~~~

## run 

~~~
./nym-gateway run --id <ID>
~~~


## bond

Abre tu wallet y ve a bond, selecciona gate way y pon los datos 
~~~
./nym-gateway sign --id <YOUR_ID> --contract-msg <PAYLOAD_GENERATED_BY_THE_WALLET>

~~~

## servicio 

~~~
nano /etc/systemd/system/nym-gateway.service
~~~

pega esto 

~~~
[Unit]
Description=Nym Gateway 1.1.33
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=root
LimitNOFILE=65536
ExecStart=/root/nym-gateway run --id <YOUR_ID>
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
~~~


~~~
systemctl daemon-reload
~~~

~~~
systemctl enable nym-gateway.service
~~~

~~~
service nym-gateway start
~~~

## testea el gateway


~~~
journalctl -f -u nym-gateway.service
~~~
