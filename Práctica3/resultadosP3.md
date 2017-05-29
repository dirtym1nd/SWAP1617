# Práctica 3

## NGINX

* Instalamos NGINX en la máquina que usaremos como balanceador
siguiendo las instrucciones del pdf. A continuación, procedemos
a configurar el servicio:
  > _sudo nano /etc/nginx/conf.d/default.conf_

  Si tuviese contenido se elimina y añadimos la información:

  upstream apaches{  
  &emsp;server ip-servidor1;  
  &emsp;server ip-servidor2;  
}

  server{  
  &emsp;listen 80;  
  &emsp;server_name balancer;
  
  &emsp;access_log /var/log/nginx/balancer.access.log;  
  &emsp;error_log /var/log/nginx/balancer.error.log;  
  &emsp;root /var/www/;

  &emsp;location /  
  &emsp;{  
    &emsp;proxy_pass http://apaches;  
    &emsp;proxy_set_header Host $host;  
    &emsp;proxy_set_header X-Real-IP $remote_addr;  
    &emsp;proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
    &emsp;proxy_http_version 1.1;  
    &emsp;proxy_set_header Connection "";  
  &emsp;}  
  }

* En caso de querer el balanceo de carga por peso, añadimos:

  upstream apaches{  
  &emsp;server ip-servidor1 weight=1;  
  &emsp;server ip-servidor2 weight=2;  
  }


* Esto hará que la máquina 2 reciba mayor carga de trabajo que la 1.
Siempre, después de una modificación:
  > _sudo systemctl restart nginx_

___
## HAPROXY

* Instalamos HAProxy en la máquina que usaremos como balanceador
siguiendo las instrucciones del pdf. A continuación, procedemos
a configurar el servicio:
  > sudo nano /etc/haproxy/haproxy.cfg

  Si tuviese contenido se elimina y añadimos la información:

  global  
	&emsp;daemon  
	&emsp;maxconn 256

  defaults  
	&emsp;mode http  
	&emsp;contimeout 4000  
	&emsp;clitimeout 42000  
	&emsp;srvtimeout 43000

  frontend http-in  
	&emsp;bind *:80  
	&emsp;default_backend servers

  backend servers  
	&emsp;server	m1 ip-servidor1:80 maxconn 32  
	&emsp;server	m2 ip-servidor2:80 maxconn 32


* En caso de querer el balanceo de carga por peso, añadimos:

  backend servers  
	&emsp;server	m1 ip-servidor1:80 maxconn 32 weight 1  
	&emsp;server	m2 ip-servidor2:80 maxconn 32 weight 2  


* Esto hará que la máquina 2 reciba mayor carga de trabajo que la 1.
Siempre, después de una modificación:
  > _sudo systemctl restart haproxy_

___
## POUND

* En el caso de Pound, instalamos con:
  > _sudo apt-get install pound_

  A continuación, modificamos el archivo /etc/pound/pound.cfg
para que tenga el siguiente aspecto:

  \## Minimal sample pound.cfg  
  \##  
  \## see pound(8) for details


  ######################################################################  
  \## global options:

  User		"www-data"  
  Group		"www-data"  
  \#RootJail	"/chroot/pound"

  \## Logging: (goes to syslog by default)  
  \##&emsp;	0&emsp;	no logging  
  \##&emsp;	1&emsp;	normal  
  \##&emsp;	2	&emsp;extended  
  \##&emsp;	3	&emsp;Apache-style (common log format)  
  LogLevel	1

  \## check backend every X secs:  
Alive		30


  ######################################################################  
  \## listen, redirect and ... to:

  \## redirect all requests on port 8080 ("ListenHTTP") to the local webserver (see "Service" below):  
ListenHTTP  
	&emsp;Address ip-balanceador  
	&emsp;Port 80  
End

  Service  
	&emsp;BackEnd  
		&emsp;&emsp;Address ip-servidor1  
		&emsp;&emsp;Port 80  
	&emsp;End  
	&emsp;BackEnd  
		&emsp;&emsp;Address ip-servidor2  
		&emsp;&emsp;Port 80  
	&emsp;End  
End


 * En caso de querer el balanceo de carga por peso, añadimos:

   Service  
	&emsp;BackEnd  
		&emsp;&emsp;Address ip-servidor1  
		&emsp;&emsp;Port 80  
        &emsp;&emsp;Priority 3  
        &emsp;End  
	End  
	&emsp;BackEnd  
		&emsp;&emsp;Address ip-servidor2  
		&emsp;&emsp;Port 80  
        &emsp;&emsp;Priority 9  
	&emsp;End  
End

* Esto hará que la máquina 2 reciba mayor carga de trabajo que la 1.
Siempre, después de una modificación:
  > _sudo systemctl restart pound_
