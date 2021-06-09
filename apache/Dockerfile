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