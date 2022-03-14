---
tags:
  - Amazon
  - Etiqueta 1
---

...


# IAW-WP-CLI
Esta práctica trata sobre el proceso de instalación de un CMS como Wordpress a través de un Command Line Interface creado para él mismo.

Nuestro script sigue basandose en dos partes `software` e `infraestructura`.
- `Infraestructura`: Se basa en la destruccion de instancias previas, grupos de seguridad y libreacion de direcciones IP elásticas. Despues del proceso de destruccion empezamos a crear la infraestructura a nuestra medida y necesidades, en nuestro caso hemos de tener una unica instancia que alojara un servidor web, los grupos de seguridad con la apertura de puertos 22 para la conexion remota a traves de SSH y los puertos 80 y 443 para permitir el trafico HTTP y HTTPS.
- `Software`: En este directorio alojamos los scripts que se van a ejecutar en nuestra instancia y el directorio `conf`, donde se guarda una plantilla del VirtualHost de Apache y un fichero de variables que utilizaremos en nuestro script.
  -  `install_LAMP.sh`: Script para la instalacion de una pila LAMP sigue siendo igual que los anteriores.
  -  `install_wp.sh`: Previamente realizamos la instalacion de WP-CLI y a continuacion utilizamos el CLI de Wordpress para la instalacion del código fuente de Wordpress, crear conexion con la BBDD y creacion de usuario y contraseña, y pot ultimo el despliegue para la configuracion de la URL, titulo del sitio y creación de credenciales de **Administrador**.

--- 

## `install_wp.sh` Instalación y despliegue de Wordpress

```
#!/bin/bash

set -x

source conf/config.sh

##############################################
# Permisos para el directorio /var/www/html
##############################################

sudo usermod -a -G www-data $USER
sudo chown -R $USER:www-data /var/www/html/
sudo chmod -R 2755 /var/www/html


##############################################
# Instalación de WP-CLI
##############################################

# Descargar
wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

# Instalación
php wp-cli.phar --info

chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp


##############################################
# Despliegue de Wordpress
##############################################

# Descargar el código fuente de Wordpress
wp core download --path=/var/www/html --locale=es_ES

# Creación del fichero de configuración (wp_config.php)
wp config create --path=/var/www/html --dbname=$DB_NAME --dbuser=$DB_USER --dbpass=$DB_PASSWORD

# Instalación de Wordpress
wp core install --path=/var/www/html --url=$DOMAIN --title="$TITLE_SITE" --admin_user=$WP_USER --admin_password=$WP_PASSWORD --admin_email=$WP_EMAIL

sudo chown -R www-data:www-data /var/www/html/
```
 Lo primero ha sido realizar la instalacaion de **WP-CLI**, la forma recomendada de instalarlo es descargando la compilación Phar.
 
 Descargamos wp-cli.phar:
 `curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
`

Se verifica que funciona:
`php wp-cli.phar --info
`

Para poder escribir solo `wp` en vez de `php wp-cli.phar`, hemos de otorgarle permisos de ejecución y moverlo a la ruta donde se almacenan los binarios.
```
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
```

Y con todo ello podremos trabajar sin problemas con el CLI de Wordpress.

El siguiente paso ha sido configurar nuestro directorio web de Apache con los permisos correspondientes para poder realizar el despliegue de Wordpress
```
sudo usermod -a -G www-data $USER
sudo chown -R $USER:www-data /var/www/html/
sudo chmod -R 2755 /var/www/html
```

Ahora para el proceso de despliegue de Wordpress la labor se vuelve mucho mas sencilla. Ejecutamos el siguiente comando para la descarga del código fuente de Wordpress.
```
wp core download --path=/var/www/html --locale=es_ES --allow-root
```

Los parametros como:
- `--path=`: Sirve para especificar la ruta donde va actuar la orden.
- `--locale=`= Seleccionamos el codigo de idioma para la instalacion que vamos a realizar en nuestro caso sera en castellano, cuyo código es **es-ES**. (En caso de desconcoer la lista de idiomas disponible en Wordpress la listaremos con el siguiente comando: `wp language core list`).

Ya tenemos Wordpress descargado en el directorio `/var/www/html`, el siguiente paso sera conectar Wordpress con la Base de Datos que previamente hemos creado con el script `install_LAMP.sh`, configuración del fichero **wp-config.php**:
```
wp config create --dbname=$DB_NAME --dbuser=$DB_USER --dbpass=$DB_PASSWORD --allow-root
```

Este comando va seguido de los siguientes parametros:
- `--dbname=`: El nombre de la Base de Datos, que previamente hemos creado.
- `--dbuser=`: El nombre de usuario para conectarse a dicha Base de Datos.
- `--dbpass=`: La constraseña establecida al usuario para acceder a la Base de Datos.


Como paso final solo hemos de finalizar la instalación de Wordpress con este último comando:
```
wp core install --url=$DOMAIN --title="$TITLE_SITE" --admin_user=$WP_USER --admin_password=$WP_PASSWORD --admin_email=$WP_EMAIL --allow-root
```
Y estos son los parametros que le pasamos:
- `--url=`: La direccion IP o nombre de dominio por el que accederemos a nuestro sitio Web.
- `--title=`: En el estableceremos el titulo de nuestra página Web. (**IMPORTANTE**: La variable ha de ir entre comillas dobles "" y el contenido de la variable entre comillas simples '').
- `--admin_user`: Creación del usuario Administrador del sitio Web de Wordpress.
- `--admin_password`: Contraseña asiganada al usuairo **Administrador** de Wordpress.

Y esta sería la practica nos conectamos a la URL de nuestro sitio Web y estaria listo.

Como paso final se ha añadido la siguiente orden para no hacer uso de ningun usuario FTP para modificacion de nuestro sitio web Wordpress.
`sudo chown -R www-data:www-data /var/www/html/`

---
## Punto final que seguir

El script de instalación de la pila LAMP se ejecutará con el siguiente comando:
```
sudo ./instal_LAMP.sh
```

El script de instalacion de WP-CLI, y de descarga e instalación de Wordpress se ejecutara con el siguiente comando:
```
sudo -u [usuario prpietario de /var/www/html] ./install_wp.sh
```


