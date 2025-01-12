# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:latest

# Damos información sobre la imagen que estamos creando
LABEL \
    version="1.0" \
    description="Ubuntu + Apache2 + virtual host" \
    maintainer="wabanchero <wabanchero@birt.eus>"

# Actualizamos la lista de paquetes e instalamos nano y apache2
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y nano apache2 proftpd openssl ssh git proftpd-mod-crypto && \
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero /var/www/html/sitiosegundo

# Copiamos archivos al contenedor
COPY /apache/indexPrimero.html /apache/indexSegundo.html /apache/sitioPrimero.conf /apache/sitioSegundo.conf /apache/sitioPrimero.key /apache/sitioPrimero.cer /

# Se copian los archivos de configuaración proftpd
COPY /ftp/proftpd.conf /etc/proftpd/
COPY /ftp/tls.conf /etc/proftpd/
COPY /ftp/modules.conf /etc/proftpd/

# Se copian los certificados
COPY /ftp/certs/proftpd_crt.pem /etc/ssl/certs/proftpd_crt.pem
COPY /ftp/certs/proftpd_key.pem /etc/ssl/private/proftpd_key.pem

# Se copia archivo de configuración ssh
COPY ssh/sshd_config /etc/ssh/sshd_config

# Se copia la clave privada ssh
COPY ssh/clave_privada /root/.ssh/id_rsa

# Movemos los archivos a sus ubicaciones adecuadas
RUN mv /indexPrimero.html /var/www/html/sitioprimero/index.html && \
    mv /indexSegundo.html /var/www/html/sitiosegundo/index.html && \
    mv /sitioPrimero.conf /etc/apache2/sites-available/sitioPrimero.conf && \
    mv /sitioSegundo.conf /etc/apache2/sites-available/sitioSegundo.conf && \
    mv /sitioPrimero.key /etc/ssl/private/sitioPrimero.key && \
    mv /sitioPrimero.cer /etc/ssl/certs/sitioPrimero.cer

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitioPrimero.conf && \
    a2ensite sitioSegundo.conf && \
    a2enmod ssl

# Se clona el repositorio 
RUN eval "$(ssh-agent -s)" \
&& chmod 700 /root/.ssh/id_rsa  \
&& ssh-add /root/.ssh/id_rsa  \
&& ssh-keyscan -H github.com >> /root/.ssh/known_hosts \
&& rm -rf /srv/ftp \
&& git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /srv/ftp


# Creamos usuarios y configuramos accesos
RUN useradd -d /var/www/html/sitioprimero -s /usr/sbin/nologin wabanchero1 && \
    echo "wabanchero1:wabanchero1" | chpasswd && \
    useradd -d /var/www/html/sitiosegundo -s /bin/bash wabanchero2 && \
    echo "wabanchero2:wabanchero2" | chpasswd && \
    useradd -d /srv/ftp/ -s /usr/sbin/nologin wabanchero 


# Permisos
RUN \
chown -R wabanchero1: /var/www/html/sitioprimero && \
chown -R wabanchero2: /var/www/html/sitiosegundo && \
chown -R wabanchero: /srv/ftp/ 
#chmod -R 755 /srv/ftp/

# Exponemos los puertos
EXPOSE 80
EXPOSE 443
EXPOSE 21 50000-50030
EXPOSE 33 

# Comando por defecto al iniciar el contenedor
# CMD ["apache2-foreground"]
CMD ["/bin/bash", "-c", "proftpd ; service ssh start ; apachectl -D FOREGROUND"]
