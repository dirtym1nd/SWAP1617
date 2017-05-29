# Práctica 2

1. Para copiar un archivo por ssh, por ejemplo, desde la máquina 1 a la 2 usamos (en la máquina 2):

   >_rsync -avz -e ssh ipmáquina1:directorioOrigen/archivo directorioDestino/archivo_
___
2. El clonado es igual que la copia de archivos. En la la máquina 2 introducimos:

   >_rsync -avz -e ssh ipmáquina1:directorioOrigen directorioDestino_
___
3. En los ejemplos anteriores nos pedirá contraseña. Para evitarlo generamos una clave en la máquina 2, por ejemplo, con:

   >_ssh-keygen -b 4096 -t rsa_

   A continuación, copiamos la clave a la máquina 1 con:
   
   >_ssh-copy-id ipmáquina1_
___
4. Si queremos mantener actualizado el contenido de la máquina 2 con el de la máquina 1, abrimos el archivo crontab con un editor:

   >_sudo nano /etc/crontab_

   Introducimos en el crontab lo siguiente:
   
   >_00 *    * * *   propietario rsync -avz --delete -e ssh ipmáquina1:/var/www/ /var/www/_