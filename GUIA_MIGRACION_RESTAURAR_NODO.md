# Guía Completa: Migración y Restauración de Nym Node desde Backup

Esta guía cubre el proceso completo para migrar un Nym Node desde un VPS existente a uno nuevo, o restaurar un nodo desde un backup existente.

---

## Tabla de Contenidos

1. [Requisitos Previos](#1-requisitos-previos)
2. [Transferencia del Backup al VPS](#2-transferencia-del-backup-al-vps)
3. [Preparación del VPS Destino](#3-preparación-del-vps-destino)
4. [Descompresión del Backup](#4-descompresión-del-backup)
5. [Verificación de la Estructura del Backup](#5-verificación-de-la-estructura-del-backup)
6. [Configuración de IP Pública](#6-configuración-de-ip-pública)
7. [Configuración de Hostname (Opcional)](#7-configuración-de-hostname-opcional)
8. [Restauración de la Base de Datos](#8-restauración-de-la-base-de-datos)
9. [Configuración de systemd](#9-configuración-de-systemd)
10. [Configuración de Reverse Proxy y WSS (Opcional)](#10-configuración-de-reverse-proxy-y-wss-opcional)
11. [Configuración del Tunnel Manager](#11-configuración-del-tunnel-manager)
12. [Verificación Final](#12-verificación-final)
13. [Actualización en Nym Wallet](#13-actualización-en-nym-wallet)
14. [Backup del Nodo Restaurado](#14-backup-del-nodo-restaurado)

---

## 1. Requisitos Previos

### 1.1. Backup existente
- Archivo `.tar.gz` con el backup del nodo
- El backup debe contener:
  - `~/.nym/nym-nodes/<ID>/config/config.toml`
  - `~/.nym/nym-nodes/<ID>/data/` (claves, mnemónico, bases de datos)

### 1.2. VPS destino
- Debian 12+ o Ubuntu 22.04+
- IPv4 e IPv6 estáticas
- Mínimo 2 GB RAM, 2 vCPU, 20 GB SSD
- Acceso SSH con usuario root

### 1.3. Conocimientos
- SSH básico
- Edición de archivos con `nano` o `vim`

---

## 2. Transferencia del Backup al VPS

### 2.1. Desde tu máquina local

```bash
# Reemplaza con tu IP del VPS y ruta del backup
scp /ruta/a/tu/backup.tar.gz root@<IP_VPS>:/root/
```

Ejemplo:
```bash
scp /home/noisk8/Documentos/NYM/bumayplato2026.tar.gz root@88.210.37.199:/root/
```

### 2.2. Verificar transferencia

En el VPS:
```bash
ls -lh /root/*.tar.gz
```

---

## 3. Preparación del VPS Destino

### 3.1. Actualizar el sistema

```bash
apt update -y && apt upgrade -y
```

### 3.2. Instalar dependencias

```bash
apt -y install ca-certificates jq curl wget ufw tmux pkg-config build-essential libssl-dev git ntp ntpdate sqlite3 unzip nano
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

### 3.3. Sincronizar tiempo

```bash
ntpdate -q pool.ntp.org
systemctl enable --now ntp
# o ntpsec según tu distribución
systemctl status ntpsec.service
```

### 3.4. Configurar firewall

```bash
ufw allow 22/tcp
ufw allow 1789/tcp
ufw allow 1790/tcp
ufw allow 8080/tcp
ufw allow 9000/tcp
ufw allow 9001/tcp
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 51822/udp
ufw enable
```

### 3.5. Configurar ulimit

```bash
echo "DefaultLimitNOFILE=65535" >> /etc/systemd/system.conf
echo "* soft nofile 65535" >> /etc/security/limits.conf
echo "* hard nofile 65535" >> /etc/security/limits.conf
sysctl --system
```

### 3.6. Habilitar IP forwarding

```bash
cat > /etc/sysctl.d/99-nym-forwarding.conf <<EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
EOF
sysctl --system
```

### 3.7. Descargar binario nym-node

```bash
cd /root
curl -L https://github.com/nymtech/nym/releases/latest/download/nym-node -o nym-node
chmod +x nym-node
./nym-node --version
```

---

## 4. Descompresión del Backup

### 4.1. Descomprimir en /root

```bash
cd /root
tar -xvzf backup.tar.gz
```

### 4.2. Verificar estructura

```bash
# Ver qué se descomprimió
ls -la /root/

# Buscar la carpeta .nym
find /root -type d -name ".nym" 2>/dev/null
```

### 4.3. Si el backup no está en la ubicación correcta

Si el backup descomprimió en una carpeta diferente (ej: `/root/backup/.nym/`), muévela:

```bash
# Asegurar que existe la carpeta base
mkdir -pv ~/.nym/nym-nodes

# Copiar el backup al lugar correcto
cp -r /root/backup/.nym/nym-nodes/<ID> ~/.nym/nym-nodes/

# O mover si prefieres
mv /root/backup/.nym ~/.nym
```

---

## 5. Verificación de la Estructura del Backup

### 5.1. Identificar el ID del nodo

```bash
ls ~/.nym/nym-nodes/
```

Deberías ver el ID del nodo (ej: `default-nym-node`, `platohedrito`, etc.)

### 5.2. Verificar archivos críticos

```bash
# Configuración
ls -la ~/.nym/nym-nodes/<ID>/config/config.toml

# Claves y datos
ls -la ~/.nym/nym-nodes/<ID>/data/

# Verificar que existan las claves
ls -la ~/.nym/nym-nodes/<ID>/data/ed25519_identity
ls -la ~/.nym/nym-nodes/<ID>/data/x25519_sphinx
ls -la ~/.nym/nym-nodes/<ID>/data/cosmos_mnemonic
```

### 5.3. Verificar base de datos

```bash
ls -la ~/.nym/nym-nodes/<ID>/data/*.sqlite
```

Deberías ver `clients.sqlite` y posiblemente `clients_backup.sqlite`.

---

## 6. Configuración de IP Pública

### 6.1. Obtener la nueva IP del VPS

```bash
NUEVA_IP=$(curl -4 https://ifconfig.me)
echo "Nueva IP: $NUEVA_IP"
```

### 6.2. Editar config.toml

```bash
nano ~/.nym/nym-nodes/<ID>/config/config.toml
```

### 6.3. Actualizar sección [host]

Bajo `[host]`:
```toml
[host]
public_ips = ["<NUEVA_IP>"]
```

Reemplaza `<NUEVA_IP>` con la IP actual del VPS.

### 6.4. Actualizar location (opcional)

Si cambió la ubicación geográfica:
```toml
[host]
location = "<NUEVA_UBICACION>"
```

Códigos de país: `ES`, `USA`, `250`, `Spain`, etc.

---

## 7. Configuración de Hostname (Opcional)

Si tienes un dominio para el nodo (recomendado para WSS):

### 7.1. Crear subdominio en tu proveedor de dominio

En tu panel de dominio (ej: Hostinguer, Namecheap, etc.):
- Tipo: A
- Nombre: `nym` (o el nombre que quieras)
- Valor: La IP pública del VPS

### 7.2. Esperar propagación DNS

```bash
# Desde tu máquina local
nslookup nym.tudominio.com
```

### 7.3. Editar config.toml

```bash
nano ~/.nym/nym-nodes/<ID>/config/config.toml
```

Bajo `[host]`:
```toml
[host]
hostname = 'nym.tudominio.com'
```

---

## 8. Restauración de la Base de Datos

### 8.1. Verificar si existe backup de la base de datos

```bash
ls -la ~/.nym/nym-nodes/<ID>/data/clients_backup.sqlite
```

### 8.2. Si existe backup, restaurarlo

```bash
# Hacer backup de la base de datos actual por seguridad
cp ~/.nym/nym-nodes/<ID>/data/clients.sqlite ~/.nym/nym-nodes/<ID>/data/clients_old.sqlite

# Restaurar desde backup
sqlite3 ~/.nym/nym-nodes/<ID>/data/clients.sqlite <<EOF
.restore ~/.nym/nym-nodes/<ID>/data/clients_backup.sqlite
.exit
EOF
```

### 8.3. Verificar integridad de la base de datos

```bash
sqlite3 ~/.nym/nym-nodes/<ID>/data/clients.sqlite "PRAGMA integrity_check;"
```

Debería devolver `ok`.

---

## 9. Configuración de systemd

### 9.1. Verificar si ya existe servicio

```bash
cat /etc/systemd/system/nym-node.service
```

### 9.2. Crear o editar el servicio

```bash
nano /etc/systemd/system/nym-node.service
```

Contenido (ajusta `<ID>` y `--mode` según tu caso):

```ini
[Unit]
Description=Nym Node
StartLimitInterval=350
StartLimitBurst=10

[Service]
User=root
LimitNOFILE=65536
ExecStart=/root/nym-node run --id <ID> --mode exit-gateway --accept-operator-terms-and-conditions --wireguard-enabled true
KillSignal=SIGINT
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

**Parámetros:**
- `--id <ID>`: El ID de tu nodo (ej: `default-nym-node`, `platohedrito`)
- `--mode`: `entry-gateway` o `exit-gateway` (debe coincidir con el modo original)

### 9.3. Recargar systemd y arrancar

```bash
systemctl daemon-reload
systemctl enable nym-node.service
service nym-node start
journalctl -u nym-node -f
```

### 9.4. Verificar que inicie correctamente

Deberías ver:
```
Started NymNodeHTTPServer on 0.0.0.0:8080
```

Si hay errores, revisa los logs y verifica que `public_ips` esté correcto en `config.toml`.

---

## 10. Configuración de Reverse Proxy y WSS (Opcional)

Si configuraste un dominio y quieres WSS:

### 10.1. Instalar Nginx y Certbot

```bash
apt install -y nginx certbot python3-certbot-nginx
```

### 10.2. Crear landing page

```bash
mkdir -p /var/www/nym.tudominio.com
curl -L https://raw.githubusercontent.com/nymtech/nym/refs/heads/develop/scripts/nym-node-setup/landing-page.html \
  -o /var/www/nym.tudominio.com/index.html
nano /var/www/nym.tudominio.com/index.html
```

### 10.3. Configurar Nginx — Reverse Proxy

```bash
nano /etc/nginx/sites-available/nym.tudominio.com
```

Contenido:
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name nym.tudominio.com;

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
ln -s /etc/nginx/sites-available/nym.tudominio.com /etc/nginx/sites-enabled/
unlink /etc/nginx/sites-enabled/default 2>/dev/null || true
nginx -t
systemctl restart nginx
```

### 10.4. Obtener certificado SSL

```bash
certbot --nginx --non-interactive --agree-tos --redirect \
  -m tu-email@ejemplo.com -d nym.tudominio.com
```

### 10.5. Configurar WSS

```bash
nano /etc/nginx/sites-available/wss-config-nym
```

Contenido:
```nginx
server {
    listen 9001 ssl http2;
    listen [::]:9001 ssl http2;
    server_name nym.tudominio.com;

    ssl_certificate /etc/letsencrypt/live/nym.tudominio.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/nym.tudominio.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

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

### 10.6. Actualizar config.toml para WSS

```bash
nano ~/.nym/nym-nodes/<ID>/config/config.toml
```

Bajo `[entry_gateway]`:
```toml
[entry_gateway]
announce_wss_port = 9001
```

Bajo `[http]`:
```toml
[http]
landing_page_assets_path = '/var/www/nym.tudominio.com'
```

### 10.7. Reiniciar nym-node

```bash
service nym-node restart
journalctl -u nym-node -f
```

---

## 11. Configuración del Tunnel Manager

### 11.1. Descargar el script

```bash
cd /root
rm -f network-tunnel-manager.sh
curl -L "https://raw.githubusercontent.com/nymtech/nym/refs/heads/develop/scripts/nym-node-setup/network-tunnel-manager.sh" \
  -o network-tunnel-manager.sh
chmod +x ./network-tunnel-manager.sh
```

### 11.2. Ejecutar configuración completa

```bash
./network-tunnel-manager.sh complete_networking_configuration
```

### 11.3. Verificar estado

```bash
./network-tunnel-manager.sh exit_policy_status
./network-tunnel-manager.sh check_firewall_setup
./network-tunnel-manager.sh check_nymtun_iptables
./network-tunnel-manager.sh check_ipv6_ipv4_forwarding
```

---

## 12. Verificación Final

### 12.1. Checklist de verificación

| # | Verificación | Comando |
|---|-------------|---------|
| 1 | Nym-node corriendo | `systemctl status nym-node` |
| 2 | Interfaz nymtun0 existe | `ip link show nymtun0` |
| 3 | IP forwarding activo | `cat /proc/sys/net/ipv4/ip_forward` → `1` |
| 4 | Tunnel Manager OK | `./network-tunnel-manager.sh exit_policy_status` |
| 5 | Landing page HTTPS | Abre `https://nym.tudominio.com` en navegador |
| 6 | API Swagger | `https://nym.tudominio.com/api/v1/swagger/#/` |

### 12.2. Obtener información de bonding

```bash
./nym-node bonding-information --id <ID>
```

---

## 13. Actualización en Nym Wallet

### 13.1. Importante: Actualizar host/IP

Cuando migras un nodo, **debes actualizar la información en Nym Wallet**:

1. Abre la **Nym Wallet**
2. Ve a **Bonding** → Encuentra tu nodo
3. Click **Update** o **Edit**
4. Actualiza:
   - **Host**: Nueva IP o dominio
   - **Custom HTTP port**: `8080` (si aplica)
5. Firma y envía la transacción

### 13.2. Verificar en explorador

Tu nodo debería aparecer en:
- [Nym Explorer](https://explorer.nymtech.net)
- [Nym Harbourmaster](https://harbourmaster.nymtech.net)

---

## 14. Backup del Nodo Restaurado

### 14.1. Crear backup del nodo restaurado

```bash
cd /root
tar -czf nym-backup-restaurado-$(date +%Y-%m-%d).tar.gz .nym/nym-nodes/<ID>
```

### 14.2. Descargar a tu máquina local

```bash
# Desde tu máquina local
scp root@<IP_VPS>:/root/nym-backup-restaurado-*.tar.gz /ruta/de/backups/
```

---

## Troubleshooting

### Error: "this node hasn't set any valid public addresses"

Edita `config.toml`:
```bash
nano ~/.nym/nym-nodes/<ID>/config/config.toml
```

Bajo `[host]`:
```toml
public_ips = ["<NUEVA_IP>"]
```

### Error: "Device nymtun0 does not exist"

El nodo debe estar corriendo primero:
```bash
service nym-node start
# Esperar unos segundos
ip link show nymtun0
./network-tunnel-manager.sh complete_networking_configuration
```

### Error: Base de datos corrupta

```bash
sqlite3 ~/.nym/nym-nodes/<ID>/data/clients.sqlite "PRAGMA integrity_check;"
```

Si no devuelve `ok`, restaura desde backup:
```bash
cp ~/.nym/nym-nodes/<ID>/data/clients_backup.sqlite ~/.nym/nym-nodes/<ID>/data/clients.sqlite
```

### El nodo no aparece en el explorador

1. Verifica que el nodo esté corriendo: `systemctl status nym-node`
2. Verifica que esté bonded en la wallet
3. Verifica que la IP/Host esté actualizada en la wallet
4. Espera unos minutos para que el explorador se actualice

---

## Referencias

- [Documentación oficial Nym](https://nym.com/docs/operators/introduction)
- [Guía de instalación desde cero](./GUIA_COMPLETA_GATEWAY_DESDE_CERO.md)

---

**Última actualización**: Junio 2026
