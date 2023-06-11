# Guia de instalacion de Odoo 15.0
- 

### Actualizar el sistema
- `sudo apt update`
- `sudo apt upgrade`

### Instalar Git, PIP, NodeJS
- `sudo apt install git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less virtualenv`

### Instalar dependencias Pillow
- `sudo apt-get install libtiff5-dev libjpeg8-dev libopenjp2-7-dev zlib1g-dev libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python3-tk libharfbuzz-dev libfribidi-dev libxcb1-dev`

### Creamos usuario Odoo15
- `sudo useradd -m -d /odoo14 -U -r -s /bin/bash odoo15`

### Instalamos y configuramos PostGreSQL
- `sudo apt install libpq-dev postgresql-client postgresql-client-common python3-psycopg2 postgresql postgresql-contrib`

### Crear usuario Postgresql odoo15
- `sudo su - postgres -c "createuser -s odoo15"`

### Instalar Wkhtmltopdf
- Buscar el repositorio adecuado en `https://wkhtmltopdf.org/downloads.html`
- Descargarlo en una carpeta con (ejemplo) `wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.bullseye_amd64.deb`
- Instalar wkhtmltopdf (ejemplo cambia version) `sudo apt install ./wkhtmltox_0.12.6.1-2.bullseye_amd64.deb`
- Corroborar version `wkhtmltopdf --version`



## Sigue en punto 5:
- https://gist.github.com/101t/3982252740ce44d3affba37edea6ed33