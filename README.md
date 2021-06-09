# Práctica 16: Dockerizar una aplicación LAMP

Para esta práctica tendremos que crear un archivo Dockerfile en la cual crearemos nuestra imagen docker que tendrá una aplicación web LAMP. 

## Dockerfile

- Utilizaremos de imagen la versión de **ubuntu** que está etiquetada como **focal** con el comando **FROM**
- Añadimos una etiqueta utilizando **LABEL**
- Añadimos la variable de entorno con **ENV**
- Ejecutaremos los comandos necesarios para la LAMP con **RUN** , la primera vez es para instalación y la segunda de configuración
- Añadimos el puerto utilizando **EXPOSE**
- Y ejecutamos el apache en primer plano con **CMD**

Mi dockerfile quedaría asi: 

```
# Ubuntu Focal Ultima version LTS
FROM ubuntu:focal 
LABEL author="Resti Sanchez" 
# Configuracion de las variables de entorno
ENV DEBIAN_FRONTEND=noninteractive
# Ejecuta bash
RUN apt update \
    && apt install apache2 -y \
    && apt install php libapache2-mod-php php-mysql -y 

RUN apt install git -y \
    && cd /tmp \
    && git clone https://github.com/josejuansanchez/iaw-practica-lamp \
    && mv /tmp/iaw-practica-lamp/src/* /var/www/html \
    && sed -i 's/localhost/mysql/' /var/www/html/config.php \
    && rm /var/www/html/index.html
# Indica los puertos que va a escuchar el contendor
EXPOSE 80
# Ejecuta apache en primer plano
CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]
```
Este dockerfile deberá de ir en la carpeta que escribamos en el dockercompose para que se ejecute

## Docker Compose

Los resquisitos para este son: 

- MySQL
- phpmyadmin
- Apache

### Apache

Para la imagen apache a diferencia de las anteriores al no ser una imagen sacada de DockerHub sino que la imagen la creamos nosotros tendremos que hacer uso del **build** 

```
 apache: 
        build: ./apache
        ports: 
            - 80:80
        depends_on: 
            - mysql
        networks: 
            - frontend-network
            - backend-network
        restart: always
```


### MySQL

Dentro de **volumes** habrá que añadir una nueva línea que será para guardar los datos dentro de la base de datos. 

```
 mysql:
        image: mysql:5.7
        ports:
            - 3306:3306
        environment:
            - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
            - MYSQL_DATABASE=${MYSQL_DATABASE}
            - MYSQL_USER=${MYSQL_USER}
            - MYSQL_PASSWORD=${MYSQL_PASSWORD}
        volumes:
            - mysql_data:/var/lib/mysql
            - ./sql:/docker-entrypoint-initdb.d
        networks:
            - backend-network
        restart: always
```
Docker iniciaría cualquier .sql dentro de la carpeta sql creada con la linea  ``- ./sql:/docker-entrypoint-initdb.d  ``  

### phpmyadmin 

```
 phpmyadmin:
        image: phpmyadmin
        environment:
            - PMA_ARBITRARY=1
        ports:
            - 8080:80
        networks:
            - frontend-network
            - backend-network
        depends_on: 
            - mysql
        restart: always
```

Solo quedaría crear el .env con las variables que vayamos a utilizar en el Docker-Compose
