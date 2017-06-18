# Práctica 6

1. En este punto vamos a configurar dos discos en RAID 1.

   * En VMWare, en la pantalla inicial para poner en funcionamiento las máquinas virtuales, seleccionamos una (en mi caso, la máquina 1) y pulsamos en "Edit virtual machine settings". Una vez en el mení, pulsamos Add, y añadimos dos discos iguales al primero (misma capacidad, tipo, etc...).

   * Una vez añadidos los discos, arrancamos la máquina e instalamos el siguiente software:

     > _sudo apt-get install mdadm_

   * Buscamos información de los discos creados:

     > _sudo fdisk -l_

   * Y, usando el dispositivo _/dev/md0_, creamos el RAID:

     > _sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc_

   * Formateamos el disco (Por defecto, _mkfs_ inicializa el disco con formato ext2):

     > _sudo mkfs /dev/md0_

   * Creamos el directorio en el que se montará el RAID:

     > _sudo mkdir /dat_  
     _sudo mount /dev/md0 /dat_

   * Para comprobar el estado del RAID ejecutamos:

     > _sudo mdadm --detail /dev/md0_

   * Ahora, procedemos a decirle al sistema que monte la configuración en el arranque del sistema. Para ello, editamos _/etc/fstab_ y ejecutamos los siguientes comandos:

     > _ls -l /dev/disk/by-uuid/_ (Para mostrar los discos por su UUID y anotamos la del RAID)  
     (Añadimos en _/etc/fstab/_ la siguiente línea) _UUID=uuid_de_nuestro_RAID /dat ext2 defaults 0 0_

___
2. En este punto simularemos un fallo en el RAID y comprobaremos que se puede retirar en caliente y volver a montar una unidad.

   * Vamos a simular u nfallo en el RAID:

     > _sudo mdadm --manage --set-faulty /dev/md0 /dev/sdb_

   * Lo retiramos en caliente:

     > _sudo mdadm --manage --remove /dev/md0 /dev/sdb_  
     (Comprobamos que efectivamente se ha retirado) _sudo mdadm --detail /dev/md0_

   * Y, por último, añadimos de nuevo en caliente la unidad:

     > _sudo mdadm --manage --add /dev/md0 /dev/sdb_  (Comprobamos el estado de nuevo) _sudo mdadm --detail /dev/md0_