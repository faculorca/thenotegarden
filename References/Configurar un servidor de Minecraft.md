# Guía Completa: Servidor de Minecraft con Pterodactyl en Ubuntu Server

## 📋 Tabla de Contenidos
1. [Requisitos Previos](#requisitos-previos)
2. [Instalación de Ubuntu Server](#instalación-de-ubuntu-server)
3. [Configuración Inicial del Sistema](#configuración-inicial-del-sistema)
4. [Instalación de Dependencias](#instalación-de-dependencias)
5. [Instalación de Pterodactyl Panel](#instalación-de-pterodactyl-panel)
6. [Configuración de NGINX](#configuración-de-nginx)
7. [Instalación de Wings](#instalación-de-wings)
8. [Creación del Servidor de Minecraft](#creación-del-servidor-de-minecraft)
9. [Solución para CG-NAT con Playit.gg](#solución-para-cg-nat-con-playitgg)
10. [Comandos Útiles](#comandos-útiles)

---

## Requisitos Previos

### Hardware
- **Laptop o PC** con al menos:
  - 4GB RAM (mínimo), 8GB recomendado
  - 50GB espacio en disco
  - Conexión a internet estable
  - Procesador dual-core o superior

### Software
- **Ubuntu Server 24.04 LTS** (ISO descargable desde ubuntu.com)
- **USB booteable** (usar Rufus en Windows o Etcher)

### Red
- Router con acceso a configuración (generalmente 192.168.1.1 o 192.168.2.1)
- Conexión por cable ethernet recomendada

---

## Instalación de Ubuntu Server

### 1. Crear USB Booteable
1. Descargar Ubuntu Server 24.04 LTS ISO
2. Usar Rufus (Windows) o Etcher para crear USB booteable
3. Bootear desde USB en la laptop

### 2. Proceso de Instalación
- Seleccionar idioma: **English** (recomendado para compatibilidad)
- Configuración de red: **Automática (DHCP)**
- Storage: **Use entire disk**
- Crear usuario (ejemplo: `facu`)
- Instalar **OpenSSH server** ✓ (importante para acceso remoto)
- No instalar paquetes adicionales por ahora

### 3. Primer Acceso
Después de reiniciar, accede con tu usuario y contraseña.

---

## Configuración Inicial del Sistema

### 1. Actualizar el Sistema

```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Configurar Laptop para Permanecer Encendida con Tapa Cerrada

**Editar archivo de configuración:**
```bash
sudo nano /etc/systemd/logind.conf
```

**Buscar y modificar estas líneas** (quitar el `#` si están comentadas):
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

**Guardar:** `Ctrl+O`, `Enter`, `Ctrl+X`

**Aplicar cambios:**
```bash
sudo systemctl restart systemd-logind
```

### 3. Obtener la IP del Servidor

```bash
ip a
```

Busca la IP que comienza con `192.168.x.x` o `10.0.x.x`. Esta es la IP de tu servidor en la red local.

---

## Instalación de Dependencias

### 1. Instalar PHP y Extensiones

```bash
sudo apt install -y php8.3 php8.3-{cli,gd,mysql,pdo,mbstring,tokenizer,bcmath,xml,fpm,curl,zip,redis}
```

**Verificar instalación:**
```bash
php -v
```

### 2. Instalar MySQL

```bash
sudo apt install -y mysql-server
```

**Acceder a MySQL:**
```bash
sudo mysql
```

**Dentro de MySQL, ejecutar:**
```sql
CREATE DATABASE panel;
CREATE USER 'pterodactyl'@'localhost' IDENTIFIED BY 'tu_contraseña_segura';
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Reemplaza `tu_contraseña_segura` con una contraseña real y guárdala.**

### 3. Instalar Redis

```bash
sudo apt install -y redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

**Verificar:**
```bash
redis-cli ping
```
Debe responder: `PONG`

### 4. Instalar Composer

```bash
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

### 5. Instalar NGINX

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
```

---

## Instalación de Pterodactyl Panel

### 1. Crear Directorio y Descargar Pterodactyl

```bash
sudo mkdir -p /var/www/pterodactyl
cd /var/www/pterodactyl
sudo curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
sudo tar -xzvf panel.tar.gz
sudo chmod -R 755 storage/* bootstrap/cache/
```

### 2. Instalar Dependencias de Composer

```bash
cd /var/www/pterodactyl
sudo composer install --no-dev --optimize-autoloader
```

**Si falta alguna extensión PHP, instalar:**
```bash
sudo apt install php8.3-{extensión-faltante}
```

### 3. Configurar Entorno

```bash
sudo cp .env.example .env
sudo php artisan key:generate --force
```

### 4. Configuración de Base de Datos

```bash
sudo php artisan p:environment:database
```

**Completar con los datos:**
- Database Host: `127.0.0.1`
- Database Port: `3306`
- Database Name: `panel`
- Username: `pterodactyl`
- Password: (la que creaste en MySQL)

### 5. Configurar Email (opcional, puedes usar valores por defecto)

```bash
sudo php artisan p:environment:mail
```

Puedes dejar los valores por defecto presionando Enter en todo.

### 6. Ejecutar Migraciones

```bash
sudo php artisan migrate --seed --force
```

### 7. Crear Usuario Administrador

```bash
cd /var/www/pterodactyl
sudo -u www-data php artisan p:user:make
```

**Responder:**
- Is this user an administrator? → `yes`
- Email Address → Tu email
- Username → Tu usuario
- First Name → Tu nombre
- Last Name → Tu apellido
- Password → Tu contraseña (mínimo 8 caracteres)

**Guarda estas credenciales, las necesitarás para acceder al panel.**

### 8. Configurar Permisos

```bash
sudo chown -R www-data:www-data /var/www/pterodactyl/*
sudo chmod -R 755 /var/www/pterodactyl/storage /var/www/pterodactyl/bootstrap/cache
```

### 9. Configurar Queue Worker

```bash
sudo nano /etc/systemd/system/pteroq.service
```

**Pegar este contenido:**
```ini
[Unit]
Description=Pterodactyl Queue Worker
After=redis-server.service

[Service]
User=www-data
Group=www-data
Restart=always
ExecStart=/usr/bin/php /var/www/pterodactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

**Guardar y activar:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now pteroq.service
```

---

## Configuración de NGINX

### 1. Crear Archivo de Configuración

```bash
sudo nano /etc/nginx/sites-available/pterodactyl.conf
```

**Pegar este contenido** (cambiar `192.168.x.x` por tu IP real):

```nginx
server {
    listen 80;
    server_name 192.168.x.x;  # Cambiar por tu IP local
    
    root /var/www/pterodactyl/public;
    index index.php;
    
    access_log /var/log/nginx/pterodactyl.app-access.log;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;
    
    client_max_body_size 100m;
    client_body_timeout 120s;
    sendfile off;
    
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;
    
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }
    
    location ~ /\.ht {
        deny all;
    }
}
```

### 2. Activar Sitio

```bash
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf
sudo nginx -t
sudo systemctl restart nginx
```

**Si `nginx -t` da errores, revisar el archivo de configuración.**

### 3. Acceder al Panel

Abre un navegador y ve a: `http://192.168.x.x` (tu IP local)

Inicia sesión con las credenciales que creaste anteriormente.

---

## Instalación de Wings

### 1. Instalar Docker

```bash
curl -sSL https://get.docker.com/ | CHANNEL=stable bash
sudo systemctl enable --now docker
```

**Verificar:**
```bash
sudo docker --version
```

### 2. Descargar Wings

```bash
sudo mkdir -p /etc/pterodactyl
curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_amd64"
sudo chmod u+x /usr/local/bin/wings
```

**Verificar:**
```bash
wings --version
```

### 3. Crear Location y Node en el Panel Web

**En el navegador (Panel de Pterodactyl):**

1. Click en **Admin Panel** (esquina superior derecha)
2. Ve a **Locations** → **Create New**
   - Short Code: `local`
   - Description: `Local Server`
   - Click **Create**

3. Ve a **Nodes** → **Create New**
   - Name: `Node1`
   - Description: `Servidor local`
   - Location: Selecciona `local`
   - FQDN: Tu IP local (ej: `192.168.2.106`)
   - **Communicate Over SSL:** DESMARCAR ✗
   - Behind Proxy: Dejar sin marcar
   - Daemon Server File Directory: `/var/lib/pterodactyl/volumes`
   - Total Memory: RAM disponible en MB (ej: `4096` para 4GB)
   - Memory Over-Allocation: `0`
   - Total Disk Space: Espacio en MB (ej: `50000` para 50GB)
   - Disk Over-Allocation: `0`
   - Daemon Port: `8080`
   - Daemon SFTP Port: `2022`
   - Click **Create Node**

### 4. Configurar Wings

**En el panel web, dentro del Node que creaste:**

1. Ve a la pestaña **Configuration**
2. **Copia TODO** el contenido del cuadro de texto (es un archivo YAML)

**En el servidor:**
```bash
sudo nano /etc/pterodactyl/config.yml
```

Pega el contenido copiado, guarda (`Ctrl+O`, `Enter`, `Ctrl+X`)

### 5. Crear Servicio de Wings

```bash
sudo nano /etc/systemd/system/wings.service
```

**Pegar:**
```ini
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service
Requires=docker.service
PartOf=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
LimitNOFILE=4096
PIDFile=/var/run/wings/daemon.pid
ExecStart=/usr/local/bin/wings
Restart=on-failure
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

**Activar:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now wings
```

**Verificar:**
```bash
sudo systemctl status wings
```

Debe mostrar "active (running)" en verde.

---

## Creación del Servidor de Minecraft

### 1. Crear Allocations

**En el panel web:**

1. **Admin Panel** → **Nodes** → Click en tu **Node1**
2. Pestaña **Allocation**
3. En **Assign New Allocations:**
   - IP Address: Tu IP local (ej: `192.168.2.106`)
   - Ports: `25565` (o un rango: `25565-25570`)
   - Click **Submit**

### 2. Crear el Servidor

**En el panel web:**

1. **Servers** (menú lateral) → **Create New**
2. Completar:
   - **Server Name:** El nombre que quieras
   - **Server Owner:** Tu usuario
   - **Node:** `Node1`
   - **Default Allocation:** Selecciona `25565`
3. En **Resource Management:**
   - **Memory:** `2048` MB (mínimo para Minecraft)
   - **Disk Space:** `10000` MB
   - **CPU Limit:** `100` (o más si quieres)
4. En **Nest Configuration:**
   - **Nest:** `Minecraft`
   - **Egg:** `Vanilla Minecraft` o `Paper` (Paper es más eficiente)
5. Click **Create Server**

### 3. Iniciar el Servidor

1. Click en el servidor recién creado
2. Click en **Start** (botón verde)
3. Ve a **Console** para ver el progreso

**Si pide aceptar EULA:**
- Ve a **Startup**
- Cambia `EULA` de `false` a `true`
- Reinicia el servidor

**Espera a ver:**
```
[Server thread/INFO]: Done (XXs)! For help, type "help"
```

### 4. Probar Conexión Local

Desde cualquier PC en tu red, abre Minecraft y conecta a:
```
192.168.2.106:25565
```

---

## Solución para CG-NAT con Playit.gg

### ⚠️ Problema del CG-NAT

**Cómo detectar si tienes CG-NAT:**

1. Averigua tu IP pública:
```bash
curl ifconfig.me
```

2. Entra a tu router y busca la **IP WAN**
3. **Si son diferentes**, tienes CG-NAT y el port forwarding NO funcionará

**Solución: Usar Playit.gg (túnel gratuito)**

### 1. Instalar Playit.gg

```bash
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/playit.gpg >/dev/null
echo "deb [signed-by=/etc/apt/trusted.gpg.d/playit.gpg] https://playit-cloud.github.io/ppa/data ./" | sudo tee /etc/apt/sources.list.d/playit-cloud.list
sudo apt update
sudo apt install playit
```

### 2. Configurar Playit

```bash
playit
```

Te mostrará un link tipo:
```
Claim URL: https://playit.gg/claim/XXXXX-XXXXX
```

**Copiar ese link y abrir en navegador:**

1. Crear cuenta gratuita en Playit.gg
2. Autorizar el agente
3. Click en **"Add Tunnel"**
4. Configurar:
   - **Tunnel Type:** TCP
   - **Local Address:** Tu IP local (ej: `192.168.2.106`)
   - **Local Port:** `25565`
   - **Agent:** El que acabas de crear
5. Click **"Create"**

Te asignará un dominio tipo:
```
example-name.gl.at.ply.gg:XXXXX
```

**Verificar en la terminal** que muestre:
```
example-name.gl.at.ply.gg:XXXXX => 192.168.2.106:25565
```

### 3. Probar Conexión

Desde tu celular con **datos móviles** (no WiFi) o un amigo, conectarse a:
```
example-name.gl.at.ply.gg:XXXXX
```

### 4. Configurar Playit como Servicio

**Primero, presiona `Ctrl+C` para detener Playit.**

**Crear directorio de configuración:**
```bash
sudo mkdir -p /etc/playit
sudo cp ~/.config/playit_gg/playit.toml /etc/playit/
sudo chmod 644 /etc/playit/playit.toml
```

**Crear servicio:**
```bash
sudo nano /etc/systemd/system/playit.service
```

**Pegar:**
```ini
[Unit]
Description=Playit.gg Tunnel Service
After=network.target

[Service]
Type=simple
User=facu
WorkingDirectory=/home/facu
ExecStart=/usr/local/bin/playit
Restart=on-failure
RestartSec=5s
Environment="HOME=/home/facu"

[Install]
WantedBy=multi-user.target
```

**IMPORTANTE:** Cambiar `facu` por tu nombre de usuario real.

**Activar servicio:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable playit
sudo systemctl start playit
```

**Verificar:**
```bash
sudo systemctl status playit
```

Debe mostrar "active (running)".

---

## Comandos Útiles

### Gestión del Sistema

```bash
# Ver estado de servicios
sudo systemctl status nginx
sudo systemctl status php8.3-fpm
sudo systemctl status mysql
sudo systemctl status redis-server
sudo systemctl status wings
sudo systemctl status pteroq
sudo systemctl status playit

# Reiniciar servicios
sudo systemctl restart [nombre-servicio]

# Ver logs
sudo journalctl -u [nombre-servicio] -f

# Ver IP del servidor
ip a | grep "inet "

# Monitorear recursos
htop  # (instalar con: sudo apt install htop)
```

### Gestión de Docker

```bash
# Ver contenedores corriendo
sudo docker ps

# Ver logs de contenedor
sudo docker logs [container-id]

# Detener todos los contenedores
sudo docker stop $(sudo docker ps -q)
```

### Diagnóstico de Conexión

```bash
# Probar conexión local
nc -zv 192.168.x.x 25565

# Probar túnel Playit
nc -zv tu-dominio.ply.gg XXXXX

# Ver IP pública
curl ifconfig.me

# Verificar puerto abierto
sudo ss -tulpn | grep :25565
```

### Mantenimiento

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Limpiar paquetes no usados
sudo apt autoremove -y

# Ver espacio en disco
df -h

# Ver uso de RAM
free -h
```

---

## 🎯 Resumen de Direcciones de Conexión

**Para ti (red local):**
```
192.168.x.x:25565
```

**Para amigos (internet con Playit.gg):**
```
tu-dominio.gl.at.ply.gg:XXXXX
```

**Panel de administración:**
```
http://192.168.x.x
```

---

## ⚠️ Notas Importantes

1. **Ventilación:** Si usas una laptop, asegúrate de que tenga buena ventilación incluso con la tapa cerrada

2. **Backups:** Considera hacer backups regulares de:
   - Base de datos: `/var/lib/mysql/`
   - Archivos del servidor: `/var/lib/pterodactyl/volumes/`

3. **Seguridad:** 
   - Usa contraseñas fuertes
   - Mantén el sistema actualizado
   - No compartas acceso SSH públicamente

4. **IP Dinámica:** Si tu ISP cambia tu IP frecuentemente, considera usar un servicio DDNS como DuckDNS

5. **Recursos:** Un servidor de Minecraft vanilla necesita mínimo 2GB RAM, Paper puede funcionar con menos

---

## 🔧 Troubleshooting Común

### "502 Bad Gateway" en NGINX
- Verificar que PHP-FPM esté corriendo: `sudo systemctl status php8.3-fpm`
- Verificar ruta del socket en NGINX config

### "Connection Refused" al conectar
- Verificar que el servidor esté corriendo en Pterodactyl
- Verificar que Wings esté corriendo: `sudo systemctl status wings`
- Verificar que Docker tenga el contenedor: `sudo docker ps`

### Playit no funciona después de reiniciar
- Verificar servicio: `sudo systemctl status playit`
- Ver logs: `sudo journalctl -u playit -n 50`

### Servidor se pausa automáticamente
- Es normal con el Egg de Vanilla
- Se despierta automáticamente cuando alguien se conecta
- O cambiar a Paper que no tiene autopause

---

¡Listo! Ahora tienes un servidor de Minecraft completamente funcional con panel de gestión profesional.