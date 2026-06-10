# Guía Completa: Montar un Nym Gateway desde Cero

Esta guía cubre el proceso completo para configurar un Nym Gateway (Entry o Exit) desde cero, incluyendo preparación del VPS, instalación de dependencias, configuración del nodo, WSS, reverse proxy, tunnel manager y bonding.

---

## Tabla de Contenidos

1. [Requisitos Previos](#1-requisitos-previos)
2. [Preparación del VPS](#2-preparación-del-vps)
3. [Instalación del Binario nym-node](#3-instalación-del-binario-nym-node)
4. [Inicialización del Nodo](#4-inicialización-del-nodo)
5. [Configuración de systemd](#5-configuración-de-systemd)
6. [Configuración de Reverse Proxy y WSS](#6-configuración-de-reverse-proxy-y-wss)
7. [Configuración del Tunnel Manager](#7-configuración-del-tunnel-manager)
8. [Bonding del Nodo](#8-bonding-del-nodo)
9. [Verificación Final](#9-verificación-final)
10. [Backup del Nodo](#10-backup-del-nodo)

---

## 1. Requisitos Previos

### 1.1. Conocimientos
- Experiencia con Linux, SSH y Bash
- Capacidad para administrar servidores remotos

### 1.2. VPS Requerimientos
- **Sistema Operativo**: Debian 12+ o Ubuntu 22.04+ (obligatorio para binarios pre-compilados)
- **IPv4 e IPv6 estáticas**: El nodo debe comunicarse con ambos protocolos
- **RAM**: Mínimo 2 GB (recomendado 4 GB)
- **CPU**: 2 vCPU mínimo
- **Almacenamiento**: 20 GB SSD mínimo
- **ISP**: Consulta [ISP recomendados](https://nym.com/docs/operators/community-counsel/isp-list)

### 1.3. Wallet NYM
- Descargar [Nym Wallet](https://nym.com/wallet)
- Crear cuenta y financiarla con al menos **101 NYM tokens nativos (Cosmos NYM)**
- **Importante**: No usar tokens ERC20, solo Cosmos NYM

### 1.4. Tiempo
- Setup inicial: 45-120 minutos
- Mantenimiento: unos minutos cada 2-4 semanas para actualizaciones

---

## 2. Preparación del VPS

### 2.1. Actualizar el sistema

```bash
apt update -y && apt upgrade -y
```

### 2.2. Instalar dependencias esenciales

```bash
apt -y install ca-certificates jq curl wget ufw tmux pkg-config build-essential libssl-dev git ntp ntpdate sqlite3 unzip nano
```

### 2.3. Sincronizar tiempo del servidor

```bash
ntpdate -q pool.ntp.org
systemctl enable --now ntp
service ntp start
service ntp status
```



### Hacer el test de vulnerabilidades

```bash

apt update && \
apt install --only-upgrade -y kmod && \
echo "install algif_aead /bin/false" > /etc/modprobe.d/disable-algif_aead.conf && \
cat > /etc/modprobe.d/dirtyfrag.conf <<'EOF'
install esp4 /bin/false
install esp6 /bin/false
install rxrpc /bin/false
EOF

rmmod algif_aead esp4 esp6 rxrpc 2>/dev/null || true

echo 3 > /proc/sys/vm/drop_caches

modprobe algif_aead 2>/dev/null || echo "[OK] algif_aead blocked"
modprobe esp4 2>/dev/null || echo "[OK] esp4 blocked"
modprobe esp6 2>/dev/null || echo "[OK] esp6 blocked"
modprobe rxrpc 2>/dev/null || echo "[OK] rxrpc blocked"

```



### 2.4. Configurar firewall (ufw)

Abre los puertos necesarios:

```bash
# Puertos básicos
ufw allow 22/tcp              
ufw allow 1789/tcp           
ufw allow 1790/tcp            
ufw allow 8080/tcp           
ufw allow 9000/tcp           
ufw allow 9001/tcp            
ufw allow 80/tcp              
ufw allow 443/tcp             
ufw allow 51822/udp           

# Habilitar firewall
ufw enable
ufw status
```

### 2.5. Configurar ulimit (archivos abiertos)

```bash
# Verificar límite actual
ulimit -n

# Configurar globalmente
echo "DefaultLimitNOFILE=65535" >> /etc/systemd/system.conf

# También en limits.conf
echo "* soft nofile 65535" >> /etc/security/limits.conf
echo "* hard nofile 65535" >> /etc/security/limits.conf

# Aplicar cambios
sysctl --system
```

### 2.6. Habilitar IP forwarding

```bash
# Crear archivo de configuración
cat > /etc/sysctl.d/99-nym-forwarding.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF

# Aplicar
sysctl --system

# Verificar
cat /proc/sys/net/ipv4/ip_forward
cat /proc/sys/net/ipv6/conf/all/forwarding
```

Ambos deben retornar `1`.

---

## 3. Instalación del Binario nym-node

### Opción A: Descargar binario pre-compilado (Recomendado)

```bash

service nym-node stop && \
rm -f nym-node hashes.json && \
wget https://github.com/nymtech/nym/releases/download/nym-binaries-v2026.10-waterloo/hashes.json && \
wget https://github.com/nymtech/nym/releases/download/nym-binaries-v2026.10-waterloo/nym-node && \
chmod +x nym-node && \
./nym-node --version && \
systemctl daemon-reload && \
service nym-node start && \
journalctl -f -u nym-node.service

```

---

## 4. Inicialización del Nodo

### 4.1. Obtener IP pública del VPS

```bash
IP_PUBLICA=$(curl -4 https://ifconfig.me)
echo "IP pública: $IP_PUBLICA"
```

### 4.2. Inicializar el nodo

**Para Entry Gateway:**

```bash
./nym-node run --id default-nym-node \
  --mode entry-gateway \
  --public-ips "$IP_PUBLICA" \
  --location "<UBICACION>" \
  --accept-operator-terms-and-conditions \
  --wireguard-enabled true
```

**Para Exit Gateway:**

```bash
./nym-node run --id default-nym-node \
  --mode exit-gateway \
  --public-ips "$IP_PUBLICA" \
  --location "<UBICACION>" \
  --accept-operator-terms-and-conditions \
  --wireguard-enabled true
```

**Parámetros:**
- `--id default-nym-node`: Identificador local (puede ser personalizado)
- `--mode`: `entry-gateway` o `exit-gateway` (solo uno a la vez)
- `--public-ips`: IP pública del VPS (obligatorio)
- `--location`: Código de país (ej: `ES`, `USA`, `250` o `Spain`)
- `--accept-operator-terms-and-conditions`: Obligatorio desde v1.1.3
- `--wireguard-enabled true`: Activa routing dVPN

### 4.3. Solo inicializar sin correr (opcional)

```bash
./nym-node run --id default-nym-node \
  --init-only \
  --mode exit-gateway \
  --public-ips "$IP_PUBLICA" \
  --location "<UBICACION>" \
  --wireguard-enabled true
```

Esto crea el archivo de configuración sin iniciar el nodo.

### 4.4. Verificar configuración

```bash
cat ~/.nym/nym-nodes/default-nym-node/config/config.toml
```

Verificar que `public_ips` tenga la IP correcta bajo `[host]`.

---

## 5. Configuración de systemd

### 5.1. Crear archivo de servicio

```bash
nano /etc/systemd/system/nym-node.service
```

Contenido:

```ini
[Unit]
Description=Nym Node
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=root
LimitNOFILE=65536
ExecStart=/root/nym-node run --id default-nym-node --mode exit-gateway --accept-operator-terms-and-conditions --wireguard-enabled true
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

**Nota**: Ajusta `--mode` según tu caso (`entry-gateway` o `exit-gateway`).

### 5.2. Activar y arrancar el servicio

```bash
systemctl daemon-reload
systemctl enable nym-node.service
service nym-node start
journalctl -u nym-node -f
```

Verificar que el nodo inicie sin errores. Deberías ver:
```
Started NymNodeHTTPServer on 0.0.0.0:8080
```

### 5.3. Comandos útiles de systemd

```bash
# Ver estado
systemctl status nym-node.service

# Ver logs en vivo
journalctl -u nym-node -f

# Reiniciar
service nym-node restart

# Detener
service nym-node stop
```

---

## 6. Configuración de Reverse Proxy y WSS

### 6.1. Requisito previo: Dominio

Necesitas un dominio apuntando a la IP del VPS. Los certificados SSL solo se emiten para dominios, no IPs.

### 6.2. Instalar Nginx y Certbot

```bash
apt install -y nginx certbot python3-certbot-nginx
```

### 6.3. Abrir puertos para Nginx

```bash
ufw allow 'Nginx Full'
ufw reload
```

### 6.4. Crear landing page

```bash
# Reemplaza nodo.noisk8.xyz con tu dominio real
mkdir -p /var/www/nodo.noisk8.xyz

# Descargar plantilla oficial
curl -L https://raw.githubusercontent.com/nymtech/nym/refs/heads/develop/scripts/nym-node-setup/landing-page.html \
  -o /var/www/nodo.noisk8.xyz/index.html

# Editar email de contacto
nano /var/www/nodo.noisk8.xyz/index.html
```

### 6.5. Configurar Nginx — Reverse Proxy HTTP

```bash
nano /etc/nginx/sites-available/nodo.noisk8.xyz
```

Contenido:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name nodo.noisk8.xyz;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Activar:

```bash
ln -s /etc/nginx/sites-available/undergroundresistence.noisk8.xyz /etc/nginx/sites-enabled/
unlink /etc/nginx/sites-enabled/default 2>/dev/null || true
nginx -t
systemctl restart nginx
```

### 6.6. Obtener certificado SSL

```bash
certbot --nginx --non-interactive --agree-tos --redirect \
  -m tu-email@ejemplo.com -d nodo.noisk8.xyz
```

### 6.7. Configurar Nginx — WSS (WebSocket Secure)

```bash
nano /etc/nginx/sites-available/wss-config-nym
```

Contenido (reemplaza `nodo.noisk8.xyz` y `<PUERTO_WSS>`, ej: `9001`):

```nginx
server {
    listen <PUERTO_WSS> ssl http2;
    listen [::]:<PUERTO_WSS> ssl http2;
    server_name undergroundresistance.noisk8.xyz;

    ssl_certificate /etc/letsencrypt/live/undergroundresistance.noisk8.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/undergroundresistance.noisk8.xyz/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    location /favicon.ico {
        return 204;
        access_log off;
        log_not_found off;
    }

    location / {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Credentials' 'true';
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, HEAD';
        add_header 'Access-Control-Allow-Headers' '*';
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://localhost:9000;
        proxy_intercept_errors on;
    }
}
```

Activar:

```bash
ln -s /etc/nginx/sites-available/wss-config-nym /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx
```

### 6.8. Configurar nym-node para anunciar WSS

```bash
nano ~/.nym/nym-nodes/default-nym-node/config/config.toml
```

Bajo `[host]`:
```toml
[host]
public_ips = ["<TU_IP_PUBLICA>"]
hostname = 'nodo.noisk8.xyz'
```

Bajo `[entry_gateway]`:
```toml
[entry_gateway]
announce_wss_port = <PUERTO_WSS>   # ej: 9001
```

Bajo `[http]`:
```toml
[http]
landing_page_assets_path = '/var/www/nodo.noisk8.xyz'
```

### 6.9. Reiniciar nym-node

```bash
systemctl daemon-reload
service nym-node restart
journalctl -u nym-node -f
```

---

## 7. Configuración del Tunnel Manager

### 7.1. Descargar el script

```bash
cd /root
rm -f network-tunnel-manager.sh
curl -L "https://raw.githubusercontent.com/nymtech/nym/refs/heads/develop/scripts/nym-node-setup/network-tunnel-manager.sh" \
  -o network-tunnel-manager.sh
chmod +x ./network-tunnel-manager.sh
```

### 7.2. Ejecutar configuración completa

```bash
./network-tunnel-manager.sh complete_networking_configuration
```

Este comando hace todo de una vez:
- Configura interfaces de túnel (`nymtun0`, `nymwg`)
- Aplica reglas NAT/forwarding
- Configura firewall para servicios nym
- Instala exit policy (filtrado de tráfico)
- Ejecuta tests de conectividad

### 7.3. Verificar estado

```bash

./network-tunnel-manager.sh exit_policy_status


./network-tunnel-manager.sh check_firewall_setup


./network-tunnel-manager.sh check_nymtun_iptables


./network-tunnel-manager.sh check_nym_wg_tun


./network-tunnel-manager.sh check_ipv6_ipv4_forwarding


./network-tunnel-manager.sh joke_through_the_mixnet
./network-tunnel-manager.sh joke_through_wg_tunnel
```

### 7.4. Si algo falla, limpiar y reinstalar

```bash
./network-tunnel-manager.sh exit_policy_clear
./network-tunnel-manager.sh exit_policy_install
```

---

## 8. Bonding del Nodo

### 8.1. Obtener información de bonding

```bash
./nym-node bonding-information --id default-nym-node
```

Esto te dará:
- `identity_key`
- `sphinx_key`
- Otros datos necesarios

### 8.2. Bonding vía Nym Wallet (Recomendado)

1. Abre la **Nym Wallet**
2. Ve a **Bonding** → Click **Bond**
3. Ingresa:
   - **ID Key** (del comando anterior)
   - **Host** (tu IP o dominio)
   - **Custom HTTP port**: `8080` (recomendado)
4. Ingresa:
   - **Amount** (mínimo 100 NYM)
   - **Operating cost**
   - **Profit margin**
5. La wallet generará un payload. Cópialo.
6. En el VPS, firma el payload:
```bash
./nym-node sign --id default-nym-node --contract-msg <PAYLOAD_GENERADO_POR_WALLET>
```
7. Pega la firma base58 en la wallet, confirma la transacción.

### 8.3. Verificar en explorador

Tu nodo debería aparecer en:
- [Nym Explorer](https://explorer.nymtech.net) (mainnet)
- [Nym Harbourmaster](https://harbourmaster.nymtech.net) (más actualizado)

---

## 9. Verificación Final

### 9.1. Checklist de verificación

| # | Verificación | Comando |
|---|-------------|---------|
| 1 | Nym-node corriendo | `systemctl status nym-node` |
| 2 | Interfaz nymtun0 existe | `ip link show nymtun0` |
| 3 | Nginx corriendo | `systemctl status nginx` |
| 4 | Certificado SSL válido | `certbot certificates` |
| 5 | Landing page HTTPS | Abre `https://nodo.noisk8.xyz` en navegador |
| 6 | API Swagger | `https://nodo.noisk8.xyz/api/v1/swagger/#/` |
| 7 | WSS accesible | Desde otra máquina: `wscat -c wss://nodo.noisk8.xyz:9001` |
| 8 | IP forwarding activo | `cat /proc/sys/net/ipv4/ip_forward` → `1` |
| 9 | Tunnel Manager OK | `./network-tunnel-manager.sh exit_policy_status` |
| 10 | Conectividad túnel | `./network-tunnel-manager.sh joke_through_the_mixnet` |

### 9.2. Logs importantes

```bash
# Logs del nodo
journalctl -u nym-node -f

# Logs de Nginx
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log

# Logs del Tunnel Manager
cat /var/log/nym/network_tunnel_manager.log
```

---

## 10. Backup del Nodo

### 10.1. Backup de claves y configuración

Desde tu **máquina local**:

```bash
mkdir -pv $HOME/backups/nym-node
scp -r root@<IP_VPS>:~/.nym/nym-nodes/default-nym-node $HOME/backups/nym-node/
```

### 10.2. Backup de clients.sqlite

En el VPS:

```bash
service nym-node stop
sqlite3 ~/.nym/nym-nodes/default-nym-node/data/clients.sqlite
sqlite> .backup ~/.nym/nym-nodes/default-nym-node/data/clients_backup.sqlite
sqlite> .exit
service nym-node start
```

### 10.3. Backup de configuración de proxy (opcional)

```bash
# Desde tu máquina local
mkdir -pv $HOME/backups/nym-proxy
scp -r root@<IP_VPS>:/var/www $HOME/backups/nym-proxy/
scp -r root@<IP_VPS>:/etc/nginx/sites-available $HOME/backups/nym-proxy/
```

### 10.4. Crear backup comprimido

En el VPS:

```bash
cd /root
tar -czf nym-backup-$(date +%Y-%m-%d).tar.gz .nym/nym-nodes/default-nym-node
```

---

## Troubleshooting Común

### Error: "Too many open files"

```bash
# Verificar ulimit
ulimit -n

# Si es 1024, aumentar
echo "DefaultLimitNOFILE=65535" >> /etc/systemd/system.conf
sysctl --system
service nym-node restart
```

### Error: "Device nymtun0 does not exist"

Normal al ejecutar el Tunnel Manager antes de iniciar el nodo. Solución:
```bash
service nym-node start
# Esperar unos segundos
ip link show nymtun0
./network-tunnel-manager.sh complete_networking_configuration
```

### Error: "this node hasn't set any valid public addresses"

Editar `config.toml`:
```bash
nano ~/.nym/nym-nodes/default-nym-node/config/config.toml
```

Bajo `[host]`:
```toml
public_ips = ["<TU_IP_PUBLICA>"]
```

### Certificado SSL expirado

```bash
certbot renew
systemctl restart nginx
```

---

## Referencias

- [Documentación oficial Nym](https://nym.com/docs/operators/introduction)
- [Nym Node Setup](https://nym.com/docs/operators/nodes/nym-node/setup)
- [Reverse Proxy & WSS](https://nym.com/docs/operators/nodes/nym-node/configuration/proxy-configuration)
- [Tunnel Manager](https://github.com/nymtech/nym/blob/develop/scripts/nym-node-setup/network-tunnel-manager.sh)
- [Bonding Guide](https://nym.com/docs/operators/nodes/nym-node/bonding)

---

**Última actualización**: Junio 2026
**Versión nym-node**: 1.32.0
