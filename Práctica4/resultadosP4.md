# Práctica 4

1. Vamos a crear nuestro certificado autofirmado SSL para una máquina con Ubuntu Server (en mi caso, la máquina 1 que actuaba de servidor en anteriores prácticas). Utilizamos los comandos listados a continuacion:

   > _sudo a2enmod ssl_

   > _sudo service apache2 restart_

   > _sudo mkdir /etc/apache/ssl_

   > _openssl -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt_

   * Nos pide una serie de datos, se los introducimos a placer y pasamos a modificar el archivo _/etc/apache2/sites-available/default-ssl_, agregando las siguientes líneas:

   > _sudo nano /etc/apache2/sites-available/default-ssl_

   > SSLCertificateFile /etc/apache2/ssl/apache.crt
SSLCertificateKeyFile /etc/apache2/ssl/apache.key

   * Activamos el sitio y reiniciamos apache:

   > _sudo a2ensite default-ssl_

   > _sudo service apache2 restart_

___
2. En este apartado configuraremos las reglas _iptables_ para una máquina (en mi caso, la máquina 1). De este modo, permitiremos el tráfico HTTP y HTTPS (puertos 80 y 443 respectivamente). Creamos un script para las reglas, el cual se iniciará cada vez que el equipo se inicie (le ponemos el nombre que queramos:

   > _sudo nano iptables_onStart.sh_

   \#!/bin/sh

   \# (1) Eliminar todas las reglas (configuración limpia)  
   iptables -F  
   iptables -X  
   iptables -t nat -F  
   iptables -t nat -X

   \# (2) Política por defecto: denegar todo el tráfico  
   iptables -P INPUT DROP  
   iptables -P OUTPUT DROP  
   iptables -P FORWARD DROP

   \# (3) Abrir los puertos HTTP (80) de servidor web  
   iptables -A INPUT -p tcp --dport 80 -j ACCEPT  
   iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

   \# (4) Abrir los puertos HTTPS (443) de servidor web  
   iptables -A INPUT -p tcp --dport 443 -j ACCEPT   
   iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT

   * Una vez guardado el archivo con esta configuración, lo añadimos al archivo crontab:

   > _sudo nano /etc/crontab_

   * Añadimos al archivo la siguiente línea:

   > _@reboot root ruta_del_script_

   * Ahora, cada cuando iniciemos el sistema se aplicarán las reglas iptables de nuestro script.

   ___

   3. En este apartado vamos a crear una nueva máquina que redirija el tráfico con reglas iptables al balanceador. Por tanto, el primer paso será seguir los pasos de creación del certificado autofirmado en la máquina 2 (no he probado a traspasar el SSL de la 1 a la 2, así que creé uno nuevo).  
   A partir de aquí, es simplemente añadir, como hemos hecho en el apartado anterior, las reglas iptables en crontab en la máquina 4. El script será el siguiente:

        \#!/bin/sh

        \# (1) Eliminar todas las reglas (configuración limpia)  
        iptables -F  
        iptables -X  
        iptables -t nat -F  
        iptables -t nat -X

        \# (2) Política por defecto: denegar todo el tráfico  
        iptables -P INPUT DROP  
        iptables -P OUTPUT DROP  
        iptables -P FORWARD DROP

        \# (3) Permitir cualquier acceso desde localhost (interface lo)  
        iptables -A INPUT -i lo -j ACCEPT  
        iptables -A OUTPUT -o lo -j ACCEPT

        \# (4) Abrir el puerto 22 para permitir acceso ssh:  
        iptables -A INPUT -p tcp --dport 22 -j ACCEPT  
        iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

        \# (5) Abrir los puertos HTTP (80) de servidor web  
        iptables -A INPUT -p tcp --dport 80 -j ACCEPT  
        iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

        \# (6) Abrir los puertos HTTPS (443) de servidor web  
        iptables -A INPUT -p tcp --dport 443 -j ACCEPT   
        iptables -A OUTPUT -p tcp --sport 443 -j ACCEPT

        \# (7) Redireccionar el tráfico al balanceador  
        echo > /proc/sys/net/ipv4/ip_forward  
        iptables -t nat -A PREROUTING -p tcp -m multiport --dports 80,443 -j DNAT --to-destination ipbalanceador:80-443  
        iptables -t nat -A POSTROUTING -p tcp -d ipbalanceador -m multiport --sports 80,443 -j SNAT --to-source ipmaquina4:80-443