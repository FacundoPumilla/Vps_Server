# Guia de configuracion base de servidor VPS contratado

# Autor
- [Pumilla Facundo](https://github.com/FacundoPumilla)

__Ultima actualizacion 2023/05/25__

    - La configuracion esta escrita para hacer uso exclusivamente de la consola.
    - La instalacion esta basado en un servidor VPS GNU/Linux Debian 11 "bullseye"
    - Una vez contradado el servicio, la empresa enviara los datos de conexion mediante mail.
    - Tales datos deberan ser usados a continuacion.

## Requisitos de maquina cliente
- Tener instalados un cliente ssh
- Windows: PuTTy, Kitty o CMDER por nombrar algunos. 
- GNU/Linux: En algunas vesiones viene instalado por defecto, pueden verificar la version con el comando: `ssh -V`

## SHH
- Conectarse al servidor VPS mediante SSH utillizando la siguiente orden: `ssh root@W.X.Y.Z -p port`. 
- Siendo `W.X.Y.Z` la IP otorgada por el servicio y port el puerto asignado.
- El servidor respondera esperando que ingreses la clave otorgada para root.

## Seteando TimeZone
- Para verigicar la zona horaria en el servidor `timedatectl`
- En caso de necesitar cambiarle `sudo timedatectl set-timezone ZONAHORARIA`
- La lista de zona se vizualisa con `timedatectl list-timezones`

## Usuario nuevo y SUDOER
- Crearemos un nuevo usuario con el comando `adduser USUARIO` responderemos los requisitos y listo.
- Verificamos que este instalado sudo mediante la orden `sudo ls -la /root` en caso que nos responda _bash: sudo: command not found,_ pasaremos a instalarlo mediante `apt install sudo`.
- Asignar permisos al usuario creado mediante la orden `usermod -aG sudo FACUNDO`
- Verficamos el cambio con la orden `groups USUARIO` si USUARIO pertence al grupo sudo ya podemos loggearnos con ese usuario.
- Salimos del servidor SSH con el comando `exit`. 

## Loguearse con USUARIO e instalar requisitos basicos
- Loguearse con la orden `ssh USUARIO@W.X.Y.Z -p port`
- Instalamos **Midnight Commander** con la orden `sudo apt install mc` nos pedira nuestra contraseña de usuario FACUNDO.
- Al ejecutar _sudo_ por primera vez nos dara un par de consejos haciendo enfasis en la privacidad de los demas.
- ingresamos a `mc` y dentro de nuestro directorio encontrara el archivo _.bashrc_ y lo editaremos presionando **F2**
- Dentro del archivo _.bashrc_ al final encontraremos las lineas **#alias ll='ls -l'** y **#alias la='ls -A'** las descomentaremos sacandoles el **#**. De esta manera la proxima vez al ingresar a nuestra consola esta, tendra un aspecto mas agradable con colores para identificar los archivos y directorios.
## APT
- Actualizar lista de paquete necesarios ejecutando `sudo apt update`
- Actualizar paquetes necesarios con `sudo apt upgrade -y`
- Buscar y ver version de paquete `sudo apt search PAQUETE`

## HOSTNAME
- Editar archivo _/etc/hostname_ y en una sola linea incluir el nombre de la maquina por ejemplo `server.local`
- Editart el archivo _/etc/hosts_ y modificarlo convenientemente a:
```
127.0.0.1	localhost localhost.localdomain
::1	localhost localhost.localdomain
IP_SERVIDOR	DOMINIO.CONTRATADO.ALGO
```
## Instalar BIND9
- Instalar el servidor de nombres DNS Bind9 con la orden `apt-get install bind9 bind9utils bind9-dnsutils bind9-doc bind9-host -y` el flag -y es para intalar sin pedir confirmacion.
- Verificamos la version instalada ejecutando `sudo named -v` (al momento de escribir esta guia esta en la version _BIND 9.16.37-Debian_)
- Verificamos el funcionamiento del servicio instalado ejecutando `sudo systemctl status named`
## Configurar Bind9
- Ingresando con `sudo mc` buscaremos el directorio `/etc/bin/`
- Editamos ahora el archivo _named.conf.options_ descomentaremos las lineas de __forwarders__ dejandola asi: (puede usar su servidor DNS preferido)
```
forwarders {
    208.67.222.222; #OpenDNS
    8.8.8.8;        #Google
    151.202.0.85;   #Verizon
}
```
- Editamos ahora el archivo _named.conf.local_ agregando 2 zonas de la siguiente manera __Atencion no usar _ (guion bajo) en los nombres__
- Son tantas zonas como dominios alojados.
```
zone "DOMINIO.CONTRATADO.ALGO" { ; (ATENCION PONER MISMO NOMBRE QUE DOMINIO)
 type master;
 allow-transfer {none;};
 file "/etc/bind/forward.DOMINIO.CONTRATADO.ALGO";
};

zone "OTRO.DOMINIO.CONTRATADO.ALGO" { ; (ATENCION PONER MISMO NOMBRE QUE DOMINIO)
 type master;
 allow-transfer {none;};
 file "/etc/bind/forward.OTRO.DOMINIO.CONTRATADO.ALGO";
};

zone "IP_SERVIDOR.in-addr.arpa" { ; (ejemplo "0.168.192.in-addr.arpa")
 type master;
 file "/etc/bind/reverse.DOMINIO.CONTRATADO.ALGO";
};
```
- Salimos de _mc_  y ejecutamos `sudo named-checkconf` si no hay resultado quiere decir que todo esta OK.
- Entramos nuevamente en `sudo mc` nos posicionaremos nuevamente en _/etc/bind/_ en un panel y con el atajo __Atl+i__ cambiara el panel opuesto con el actual.
- No posicionamos sobre el archivo _db.local_ y con __F5__ lo copiamos el mismo directorio pero con el nombre antes sugerido __forward.DOMINIO.CONTRATADO.ALGO__, hacemos lo mismo con el archivo _db.127_ hacia el nombre __reverse.DOMINIO.CONTRATADO.ALGO__ 
- __Atencion no usar _ en los nombres__

### Edicion de archivos de configuracion de BIND9
- Editamos el archivo _forward.DOMINIO.CONTRATADO.ALGO_ dejandolo convenientemente similar a:
```
$TTL    3D
@       IN      SOA     DOMINIO.CONTRATADO.ALGO. mail.responsable. (
                        2          ; Serial // incrementar en cada modificacion
                        8H         ; Refresh
                        2H         ; Retry
                        4W         ; Expire
                        1D )       ; Negative Cache TTL

;Definition of name server for DOMINIO.CONTRATADO.ALGO
        NS      ns1.DOMINIO.CONTRATADO.ALGO
        NS      ns2.DOMINIO.CONTRATADO.ALGO
        TXT     "zona DNS-> DOMINIO.CONTRATADO.ALGO!!"
        HINFO   "VPS Server"    "Debian 11"

;Resolved dns to IP address
localhost   IN  A   127.0.0.1
ns1         IN  A   IP_SERVIDOR
ns2         IN  A   IP_SERVIDOR
@           IN  A   IP_SERVIDOR

;Other domains for DOMINIO.CONTRATADO.ALGO
www         IN  A   IP_SERVIDOR
test        IN  A   IP_SERVIDOR
```

- Ahora editaremos el archivo reverse.DOMINIO.CONTRATADO.ALGO_ dejandolo convenientemente similar a:
```
$TTL    3D
@       IN      SOA     DOMINIO.CONTRATADO.ALGO. mail.responsable. (
                        2          ; Serial // incrementar en cada modificacion
                        8H         ; Refresh
                        2H         ; Retry
                        4W         ; Expire
                        1D )       ; Negative Cache TTL

;Name Server INFO for DOMINIO.CONTRATADO.ALGO
@       IN      NS      ns1.DOMINIO.CONTRATADO.ALGO.
                NS      ns2.DOMINIO.CONTRATADO.ALGO.
                TXT     "Reverse Zone-> DOMINIO.CONTRATADO.ALGO"

;Reverse DNS or PTR Record for DOMINIO.CONTRATADO.ALGO
zz      IN      PTR     ns1.DOMINIO.CONTRATADO.ALGO.
zz      IN      PTR     DOMINIO.CONTRATADO.ALGO.
zz      IN      PTR     www.DOMINIO.CONTRATADO.ALGO.
```
- Editamos el archivo _/etc/resolv.conf_ de la siguiente manera
```
domain DOMINIO.CONTRATADO.ALGO
search DOMINIO.CONTRATADO.ALGO
nameserver IP_SERVIDOR
```
- Chequeamos los archivos _/etc/forward.DOMINIO.CONTRATADO.ALGO_ de la siguiente manera `sudo named-checkzone zonename /etc/bind/zonename`
- Chequeamos toda la configuracion con `sudo named-checkconf -z`
```
sudo named-checkzone DOMINIO.CONTRATADO.ALGO /etc/bind/forward.DOMINIO.CONTRATADO.ALGO
sudo named-checkzone IP_SERVIDOR.in-addr.arpa /etc/bind/reverse.DOMINIO.CONTRATADO.ALGO
```
## Delegacion de dominios en NIC.AR
- Ingresamos con nuestra clave de AFIP y en el apartado TRAMITES a distancia NIA y delegaciones ingresamos a cada dominio y autodelegamos a:
```
ns1.dominio.contradado IP_SERVIDOR_VPS #ns1 y ns2 corresponde con la direccion del DNS Server
ns2.dominio.contradado IP_SERVIDOR_VPS
```
## Comprobacion de propagacion DNS
- https://dnspropagation.net/
- O desde cualquier consola linux `nslookup DOMINIO.CONTRATADO.ALGO`
- O `dig @IP_SERVIDOR DOMINIO.CONTRATADO.ALGO`
- O `dig DOMINIO.CONTRATADO.ALGO`

## Instalar Nginx
- Actualizamos el sistema en busca de cambios `sudo apt update`
- Instalamos nginx `sudo apt install nginx`
- Comandos necesarios 
```
sudo systemctl stop nginx   #Detener servidor
sudo systemctl start nginx  #Iniciar servidor
sudo systemctl reload nginx #Recargar servidor
```
- Creamos un directorio de prueba `sudo mkdir -p /var/www/DOMINIO.CONTRATADO.ALGO`
- Asignamos permisos al usuario actual `sudo chown -R $USER:$USER /var/www/DOMINIO.CONTRATADO.ALGO`
- Cambiamos permisos `sudo chmod -R 755 /var/www/DOMINIO.CONTRATADO.ALGO`
- Creamos un archivo `touch /var/www/DOMINIO.CONTRATADO.ALGO/index.html`
- Editamos el archivo con `mcedit /var/www/DOMINIO.CONTRATADO.ALGO/index.html`, y le colocamos dentro un mensaje de bienvenida.

## Configurar Nginx
- Creamos un archivo `touch /etc/nginx/sites-availaible/DOMINIO.CONTRATADO.ALGO`
- y lo editamos con la configuracion basica de nginx:
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/DOMINIO.CONTRATADO.ALGO;
        index index.html index.htm index.nginx-debian.html;

        server_name DOMINIO.CONTRATADO.ALGO www.DOMINIO.CONTRATADO.ALGO;

        location / {
                try_files $uri $uri/ =404;
        }
}
```
- Creamos un enlace simbolico del archivo `sudo ln -s /etc/nginx/sites-available/DOMINIO.CONTRATADO.ALGO /etc/nginx/sites-enabled/`
- Descomentamos la linea __server_names_hash_bucket_size 64;__ para evitar problemas al agregar mas nombres de dominio.
- Testeamos que la configuracion de nginx sea correcta `sudo nginx -t`
- Reiniciamos nginx `sudo systemctl restart nginx`
- Testeamos el dominio agregado que muestre la pagina de pruebas creada. __DOMINIO.CONTRATADO.ALGO__

## Configuracion de acces.log de Nginx
- https://juantrucupei.wordpress.com/2020/11/03/configurar-personalizar-access-logs-en-nginx/

## Instalacion de PHP
- Actualizamos paquetes `sudo apt update`
- Descargamos la clave de Sury `sudo curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg`
- Sumamos el repositorio de Sury `sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'`
- Volvemos a actualizar paquetes
- Instalamos php y extensiones `sudo sudo apt install php php-cli php-bcmath php-zip php-mysql php-imagick php-memcache php-memcached php-xml php-common php-curl php-gd php-intl php-json php-mbstring php-mysql`
- En este punto se instala por defecto apache2, pasaremos a desactivarlo `sudo systemctl disable --now apache2`
- y removeremos todo sus dependencias `sudo apt remove apache2.*`
- Borramos el dirtectorio de contenido `sudo rm -rf /etc/apache2`
- Y purgamos cualquier residuo de apache `sudo apt-get purge apache2`
- En caso de contar con distintas versiones de PHP, existe una herramienta muy util `sudo update-alternatives --config php`
- O se puede hacer directamente mediante `sudo update-alternatives --set php /usr/bin/phpY.Z` siendo Y.Z la version de php requerida e instalada previamente.
- Reiniciamos el servicio de __php_fpm__ `sudo service php-fpm restart`

## Re-configuramos nginx
- En este punto reconfiguramos el arhivo default nginx para ejecutar script de php con php-fpm (si es necesario)
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;index index.html index.php index.htm index.nginx-debian.html;
        server_name _;
        location / {
                try_files $uri $uri/ /index.php;
        }
        location ~ \.php$ {
                add_header Access-Control-Allow-Origin *;
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        }
        location ~ /\.ht {
                deny all;
        }
}
```

## Instalacion de MySql
- Actualizamos paquetes `sudo apt update`
- Instalamos _mariadb-server_ `sudo apt install mariadb-server`
- Aseguramos _mariadb_ `sudo mysql_secure_installation` y seguimos los pasos. 
- Darle acceso al usuario creado al principio.
- Desde la consola ingresamos a _mysql_ con root `sudo mysql -u root -p` (la contraseña expedida al inicio en datos de VPS)
- Dentro de la terminal de _mysql_ mostramos todas las bases de datos con `show databases;`
- Mostramos los usuarios dentro de la tabla __user__ de la base de datos __mysql__ `select user from mysql.user`
- En el resultado figurara el usuario agregado al principio en __adduser__.
- Incorporamos privilegios al usuario `GRANT ALL PRIVILEGES ON * . * TO '__USUARIO__'@'localhost';`
- Recargamos los privilegios `FLUSH PRIVILEGES;`
- Salimos con `exit`

## Instalacion de phpmyadmin
- Actualizamos paquetes `sudo apt update`
- Instalamos _phpmyadmin_ `sudo apt install phpmyadmin -y`, seguimos los pasos indicados.

## 
### Lectura de apoyo
- Usuarios https://www.ionos.es/ayuda/servidores-cloud/administracion-del-servidor/crear-un-usuario-con-permisos-sudo/
- DNS Server https://servidordebian.org/es/buster/intranet/dns/server
- DNS Server https://blyx.com/public/docs/bind.html -> muy util
- Nginx https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-11
- Nginx https://howtoforge.es/como-instalar-phpmyadmin-con-nginx-y-let-s-encrypt-ssl-en-ubuntu-20-04-lts/
- Nginx y php https://josuamarcelc.medium.com/standard-nginx-php7-4-with-mysql-installation-in-digital-ocean-2020-fc27678f1cd7
- Php https://packages.sury.org/php/README.txt
- Mysql https://www.digitalocean.com/community/tutorials/crear-un-nuevo-usuario-y-otorgarle-permisos-en-mysql-es