# Guia de instalacion de Odoo 15.0
- 

### Actualizar el sistema
- `sudo apt update`
- `sudo apt upgrade`

### Descarga de paquetes
- `wget https://github.com/ctmil/odoo-argentina/archive/refs/heads/15.0.zip`
- `wget https://github.com/regaby/odoo-custom/archive/refs/heads/15.0-moldeo.zip`








### Instalar Git, PIP, NodeJS
- `sudo apt install git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less virtualenv`

### Instalar dependencias Pillow
- `sudo apt-get install libtiff5-dev libjpeg8-dev libopenjp2-7-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python3-tk libharfbuzz-dev libfribidi-dev libxcb1-dev`

### Creamos usuario Odoo15
- Creamos el usuario y la carpeta que usara `sudo useradd -m -d /odoo15 -U -r -s /bin/bash odoo15`

### Instalamos y configuramos PostGreSQL
- `sudo apt install libpq-dev postgresql-client postgresql-client-common python3-psycopg2 postgresql postgresql-contrib`

### Crear usuario Postgresql odoo15
- `sudo su - postgres -c "createuser -s odoo15"`

### Instalar Wkhtmltopdf
- Buscar el repositorio adecuado en `https://wkhtmltopdf.org/downloads.html`
- Descargarlo en una carpeta con (ejemplo) `wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.bullseye_amd64.deb`
- Instalar wkhtmltopdf (ejemplo cambia version) `sudo apt install ./wkhtmltox_0.12.6.1-2.bullseye_amd64.deb`
- Corroborar version `wkhtmltopdf --version`

### Instalar y configurar Odoo 15
- Cambiamos al iusuario odoo15 creado anteriormente `sudo su - odoo15`
- Clonamos el proyecto desde git `git clone https://www.github.com/odoo/odoo --depth 1 --branch 15.0 /odoo15/odoo15-server/`
- Creamos un entorno virtual de Python para la instalacion de Odoo 15 `cd /odoo15-server` y  `virtualenv - p python3 env`
- Siguiente, activamos el entorno `source env/bin/activate`
- Instalamos los requerimientos `pip install wheel %% pip install -r requeriments.txt`
- Finalizada la instalacion desactivamos el entorno `deactivate`
- Creamos un directorio para los addons personalizados `mkdir /odoo15/odoo15-server/custom`
- Salimos al usuario sudo `exit`
- Creamos un archivo de configuracion basado en un ejemplo `sudo cp /odoo15/odoo15-server/odoo/debian/odoo.conf /odoo15/odoo15-server/odoo15-server.conf`
- Editamos el archivo:
```
[options]
; This is the password that allows database operations:
admin_passwd = ACA_UN_PASSWORD_FUERTE
db_host = localhost
db_port = 5432
db_user = False
db_password = False
addons_path = /odoo15/odoo15-server/addons,/odoo15/odoo15-server/custom
```

### Creando el servicio Odoo
- Creamos el archivo a editar `sudo touch /etc/systemd/system/odoo15.service`
- Y lo editamos desde `sudo mc` y agregamos el siguiente codigo:
```
[Unit]
Description=Odoo14 Community Edition
Requires=postgresql.service
After=network.target postgresql.service

[Service]
Type=simple
SyslogIdentifier=odoo14
PermissionsStartOnly=true
User=odoo14
Group=odoo14
ExecStart=/odoo14/odoo14-server/env/bin/python3 /odoo14/odoo14-server/odoo-bin -c /odoo14/odoo14-server/odoo14-server.conf
StandardOutput=journal+console

[Install]
WantedBy=multi-user.target
```
- Reiniciar los servicios `sudo systemctl daemon-reload`
- Iniciar odoo `sudo systecmtl start odoo15`
- Habilitar odoo para que inicie automaticamente `sudo systemctl enable odoo15`
- Verificamos el estado de odoo `sudo systemctl status odoo15`
- Verificamos mensajes de log en el servicio ` sudo journalctl -u odoo15`
- Verificamos la carga del servicio desde un explorador apuntando a `http://DOMINIO.CONTRATADO.ALGO:8069`
 
## Sigue en punto 6:
- https://gist.github.com/101t/3982252740ce44d3affba37edea6ed33