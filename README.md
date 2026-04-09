Práctica: Compartir archivos entre Linux utilizando NFS 📂
Autor: Edwind Isaías
Matrícula: 20240318
Entorno: * Servidor: VM Linux (Host con los archivos)

Cliente: VM Linux (Donde se monta la carpeta)

🚀 Pasos en el SERVIDOR (NFS Server)
1. Instalación y creación del directorio
Primero instalamos el paquete del servidor y creamos la carpeta OS3 con los 100 archivos solicitados.

Bash
# Actualizar e instalar el servidor NFS
sudo apt update
sudo apt install nfs-kernel-server -y

# Crear el directorio
sudo mkdir -p /mnt/OS3

# Generar los 100 archivos Adrian[1..100].txt de forma automática
touch /mnt/OS3/Adrian{1..100}.txt
2. Permisos de carpeta y usuario
Para que el cliente pueda entrar y leer los archivos, debemos ajustar los permisos de la carpeta.

Bash
# Asignar el propietario al usuario nfsnobody (o nobody según la distro)
sudo chown nobody:nogroup /mnt/OS3

# Dar permisos de lectura y ejecución para otros usuarios
sudo chmod 755 /mnt/OS3
3. Configuración de los Exports
Debemos decirle al sistema quién tiene permiso de conectarse a esa carpeta.

Bash
# Editar el archivo de exportaciones
sudo nano /etc/exports
Añade esta línea al final (sustituye <IP_DEL_CLIENTE> por la IP de tu VM Cliente):

Plaintext
/mnt/OS3  <IP_DEL_CLIENTE>(rw,sync,no_subtree_check)
Luego, aplica los cambios:

Bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
🚀 Pasos en el CLIENTE (NFS Client)
4. Instalación y montaje manual
Instalamos las utilidades necesarias y preparamos el punto de montaje.

Bash
# Instalar el paquete cliente
sudo apt update
sudo apt install nfs-common -y

# Crear el punto de montaje local
sudo mkdir -p /home/usuario/compartido_os3

# Montar manualmente para probar (sustituye <IP_DEL_SERVIDOR>)
sudo mount <IP_DEL_SERVIDOR>:/mnt/OS3 /home/usuario/compartido_os3

# Verificar que los archivos están ahí
ls /home/usuario/compartido_os3
5. Configuración del montaje automático (/etc/fstab)
Para que no tengas que montarlo a mano cada vez que reinicies, editamos el archivo fstab.

Bash
sudo nano /etc/fstab
Añade esta línea al final del archivo:

Plaintext
<IP_DEL_SERVIDOR>:/mnt/OS3  /home/usuario/compartido_os3  nfs  defaults,user,_netdev  0  0
✅ Pruebas de validación (Para tus capturas)
Captura 1: En el cliente, ejecuta ls /home/usuario/compartido_os3 para mostrar los 100 archivos de "Adrian".

Captura 2 (El Reinicio): Ejecuta sudo reboot en la VM Cliente.

Captura 3: Una vez suba el sistema, abre la terminal y tira un df -h o un ls de la carpeta sin haber escrito ningún comando de montaje. Verás que la carpeta aparece montada automáticamente gracias al /etc/fstab.

🛠️ Plan de Contingencia (Troubleshooting)
🛡️ Si el comando mount se queda "colgado" o da Timeout:

Causa: El Firewall del servidor está bloqueando el tráfico NFS.

Solución: En el Servidor, permite el tráfico:
sudo ufw allow from <IP_DEL_CLIENTE> to any port nfs

🛡️ Si al ver los archivos salen con dueño "4294967294":

Causa: Es un comportamiento normal de NFS cuando no hay un mapeo de usuarios idénticos entre máquinas. Mientras puedas leer los archivos (ls), la práctica está correcta.
