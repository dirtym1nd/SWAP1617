# Práctica 5

1.  Para la creación de la base de datos, entramos en _MySQL_ con:

    > _sudo mysql -u root -p_
  
    * Una vez dentro, creamos la base de datos:
    > _CREATE DATABASE contactos;_

    * A continuación, entramos en la BD:
    > _USE contactos;_

    * Una vez abierta, creamos una tabla:

    > _CREATE TABLE datos(nombre varchar(100),tlf int);_

    * e insertamos los datos que queremos en la tabla:

    > _INSERT INTO datos(nombre,tlf) VALUES("Pepe",95834987);_
___
2. Accedemos a _MySQL_ y usamos: 
   > _FLUSH TABLES WITH READ LOCK;_

   * Una vez hecho esto, guardamos la BD en la carpeta temporal para proceder a la copia y, posteriormente, a la restauración:

   > _mysqldump contactos -u root -p > /tmp/contactos.sql_

   * Desbloqueamos las tablas:
   > _UNLOCK TABLES;_

___
3. Ahora, en la máquina 2, realizamos la copia:

   > _scp usuario@ipmaquina1:/tmp/contactos.sql /tmp/_

   * Repetimos el proceso de creación de una BD en la máquina 2, pero esta vez introducimos los datos de la copia a la nueva BD:

   > _sudo mysql -u root -p contactos < /tmp/contactos.sql_

___
4. En primer lugar entramos en la configuración de _MySQL_ del maestro:

   > _sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf_

   * Comentamos el parámetro _bind-address_:

   > #bind-address 127.0.0.1

   * Indicamos el log de errores, el identificador del servidor y el log binario:

   > log_error = /var/log/mysql/error.log

   > server-id = 1

   > log_bin = /var/log/mysql/bin.log

   * Guardamos el documento y reiniciamos el servicio:

   > _sudo /etc/init.d/mysql restart_

   * Ahora, pasamos a hacer la misma configuración en la máquina esclavo, pero esta vez el _server-id_ será 2.
   * Reiniciamos el servicio de la máquina 2.
   * Entramos de nuevo en la máquina maestro y creamos un usuario con los siguientes comandos:

   > _CREATE USER esclavo IDENTIFIED BY 'esclavo';_

   > _GRANT REPLICATION SLAVE ON \*.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';_

   > _FLUSH PRIVILEGES;_

   > _FLUSH TABLES;_

   > _FLUSH TABLES WITH READ LOCK;_

   * Usamos el siguiente comando para obtener los datos de la BD que vamos a replicar en el esclavo:

   > _SHOW MASTER STATUS;_

   * Volvemos a la máquina esclavo, entramos en _MySQL_ y pasamos los datos del maestro:

   > _CHANGE MASTER TO MASTER_HOST='ip-máquina1', MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='log-máquina1', MASTER_LOG_POS='pos-máquina1', MASTER_PORT=3306;_

   * Arrancamos el esclavo para poder empezar con la replicación:

   > _START SLAVE;_

   * En la máquina maestro desbloqueamos las tablas para poder introducir nuevos datos:

   > _UNLOCK TABLES;_

   * Y, para asegurarnos de que todo funciona perfectamente, introducimos el siguiente comando en el esclavo y, si la variable "Seconds_Behind_Master" es diferente de null, todo estará funcionando perfectamente y los datos introducidos en el maestro se replicarán al esclavo:

   > _SHOW SLAVE STATUS\G;_

___
## MAESTRO-MAESTRO
* Vamos a seguir los mismos pasos que en la sección anterior referente a crear el usuario y pasarle los datos del maestro, pero con las máquinas intercambiadas. Ponemos el usuario que queramos en la máquina 2:

  > _CREATE USER maestro IDENTIFIED BY 'maestro';_

  > _GRANT REPLICATION SLAVE ON \*.* TO 'maestro'@'%' IDENTIFIED BY 'maestro';_

  > _FLUSH PRIVILEGES;_

  > _FLUSH TABLES;_

  > _FLUSH TABLES WITH READ LOCK;_

  * De nuevo, sacamos los datos, esta vez de la máquina 2 (desde la máquina 1), que era nuestro primer esclavo y ahora es maestro:

  > _SHOW MASTER STATUS;_

  * En la máquina 1 ahora, realizamos los mismos comandos:

  > _CHANGE MASTER TO MASTER_HOST='ip-máquina2', MASTER_USER='maestro', MASTER_PASSWORD='maestro', MASTER_LOG_FILE='log-máquina2', MASTER_LOG_POS='pos-máquina2', MASTER_PORT=3306;_

  * Arrancamos el esclavo en la máquina 1:

  > _START SLAVE;_

  * Desbloqueamos las tablas de la máquina 2:

  > _UNLOCK TABLES;_

  * Y, como antes, nos aseguramos de que la variable "Seconds_Behind_Master" es diferente de null y ya tenemos nuestro mestro-maestro funcionando:

  > _SHOW SLAVE STATUS\G;_