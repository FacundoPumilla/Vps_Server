# Guian de instalacionde PrestaShop 8

## Partiendo de la base de un servidor VPS y o cualquier servidor Debian 11
- Ultima actualizacion `2023/05/31`

## Configuracion de php necesaria

## Configuracion de nginx

```
server {
	root /var/www/viverde.ar;
	index index.php;
	server_name viverde.ar www.viverde.ar viverde.com.ar www.viverde.com.ar;

	# Images.
	rewrite ^/(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$1$2.jpg last;
        rewrite ^/(\d)(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$2/$1$2$3.jpg last;
        rewrite ^/(\d)(\d)(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$2/$3/$1$2$3$4.jpg last;
        rewrite ^/(\d)(\d)(\d)(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$2/$3/$4/$1$2$3$4$5.jpg last;
        rewrite ^/(\d)(\d)(\d)(\d)(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$2/$3/$4/$5/$1$2$3$4$5$6.jpg last;
        rewrite ^/(\d)(\d)(\d)(\d)(\d)(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$2/$3/$4/$5/$6/$1$2$3$4$5$6$7.jpg last;
        rewrite ^/(\d)(\d)(\d)(\d)(\d)(\d)(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$2/$3/$4/$5/$6/$7/$1$2$3$4$5$6$7$8.jpg last;
        rewrite ^/(\d)(\d)(\d)(\d)(\d)(\d)(\d)(\d)(-[\w-]+)?/.+\.jpg$ /img/p/$1/$2/$3/$4/$5/$6/$7/$8/$1$2$3$4$5$6$7$8$9.jpg last;
        rewrite ^/c/([\w.-]+)/.+\.jpg$ /img/c/$1.jpg last;

        # AlphaImageLoader for IE and FancyBox.
        rewrite ^images_ie/?([^/]+)\.(gif|jpe?g|png)$ js/jquery/plugins/fancybox/images/$1.$2 last;

        # Web service API.
	    rewrite ^/api/?(.*)$ /webservice/dispatcher.php?url=$1 last;

        # Installation sandbox.
        rewrite ^(/install(?:-dev)?/sandbox)/.* /$1/test.php last;

	location / {
	    try_files $uri $uri/ /index.php$is_args$args;
	}

        # [EDIT] Replace 'admin-dev' in this block with the name of your admin directory.
        " RANDOM_NUMBER is the generate PrestaShop install.
        location /admin-dev/ {
	    if (!-e $request_filename) {
    	        rewrite ^/.*$ /adminRANDOM_NUMBER/index.php last;
	    }
	}

    # .htaccess, .DS_Store, .htpasswd, etc.
    location ~ /\. {
        deny all;
    }

    # Source code directories.
    location ~ ^/(app|bin|cache|classes|config|controllers|docs|localization|override|src|tests|tools|translations|var|vendor)/ {
        deny all;
    }

    # vendor in modules directory.
    location ~ ^/modules/.*/vendor/ {
        deny all;
    }

    # Prevent exposing other sensitive files.
    location ~ \.(log|tpl|twig|sass|yml)$ {
        deny all;
    }

    # Prevent injection of PHP files.
    location /img {
        location ~ \.php$ { deny all; }
    }

    location /upload {
        location ~ \.php$ { deny all; }
    }

    location ~ [^/]\.php(/|$) {
        # Split $uri to $fastcgi_script_name and $fastcgi_path_info.
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;

        # Ensure that the requested PHP script exists before passing it
        # to the PHP-FPM.
        try_files $fastcgi_script_name =404;

        # Environment variables for PHP.
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $request_filename;

        fastcgi_index index.php;
	    client_max_body_size 2M;
        fastcgi_keep_conn on;
        fastcgi_read_timeout 30s;
        fastcgi_send_timeout 30s;

        # [EDIT] Connection to PHP-FPM unix domain socket.
        fastcgi_pass unix:/run/php/php7.4-fpm.sock;
    }
	
	listen 80;
        listen [::]:443 ssl ipv6only=on; # managed by Certbot
	listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/viverde.ar/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/viverde.ar/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = www.viverde.com.ar) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host = www.viverde.ar) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host = viverde.com.ar) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    if ($host = viverde.ar) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

	listen 80;
	listen [::]:80;

	server_name viverde.ar www.viverde.ar viverde.com.ar www.viverde.com.ar;
    return 404; # managed by Certbot
}
```
## Configuracion de carpetas

## Descarga, Copiado y descomprension de archivos

## Usuarios y base de datos en MySQL

## Instalacion de Certbot
- Instalamos paquetes necesarios `sudo apt install certbot python3-certbot-nginx`

## Lectura de apoyo
- Let's encript  https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-debian-11