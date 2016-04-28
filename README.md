# ProyectoIRedesII
Documentacion de los servicios levantados para el proyecto del curso redes II

Universidad Rafael Landívar
Campus Quetzaltenango
Facultad de ingeniera
Ingeniera en sistemas e Informática
Redes 2
Ing. Dhaby Xiloj.


Proyecto No. 2
Servicio de hosting y redundancia.


- Miguel Orlando Diaz Muñoz 15227-12
- Kevin Antonio Velásquez Aguilar 16736-11
- Pablo Andrés López Velásquez 16293-11
- Luis Antonio Makepeace de León 15676-12
- Cristian Daniel Flores Puac 15030-11
- Josue Alexander Gomez Lopez 1656411
- Abner Méndez Gómez 22262-10


##Diseño de la red.
![Alt text](https://github.com/panlopezv/ProyectoIRedesII/blob/master/red.png?raw=true"Optional title")


##Ruto dinámico en Vyatta(OSPF).
Al tener instalado el sistema es necesario iniciar con el Usuario por defecto que ya viene con el sistema.  
-	Login:”vyos”
-	Password: ”vyos”

Luego de esto debemos de ver nuestras interfaces, 
-	vyatta@R1#  show interfaces

Luego de que miramos que nuestras interfaces están bien, entramos a las configuraciones del router, y les asignamos la ip para cada interfaz.
  
-	vyatta@R1#  configure
-	vyatta@R1#  set interfaces ethernet eth2 address 10.10.10.66/26
-	vyatta@R1#  commit (este comando siempre debe de ejecutarse para que los cambios sean aplicados permanentemente).
De esta forma establecemos las direcciones ip de cada interfaz si cometemos un error y deseamos quitar la ip que acabamos de asignar simplemente cambiamos el comando set  por delete. Si lo que deseamos es observar cómo está el archivo de configuración una vez hemos establecido las ips lo hacemos de forma individual u observamos todo el bloque a través de los siguientes comandos

###Configurar OSPF area 0.
-	vyatta@R0# configure
-	vyatta@R0# set protocols ospf area 0 network 192.168.2.0/24


##Servidor FTP

FTP se utiliza para transferir archivos desde un host a otro a través de la red TCP. Hay 3 paquetes de servidor FTP populares disponibles PureFTPd, vsftpd y Proftpd. Aquí he utilizado Vsftpd que es la vulnerabilidad de peso ligero y menos. 

Paso 1. Actualizar repositorios.

    sudo apt-get update

Paso 2. Instalar paquete vsftpd con el comando a continuación:

    sudo apt-get install vsftpd

Paso 3. Despues de la instalacion abrir archivo /etc/vsftpd.conf y descomentar las lineas 29 y 30:

    write enable = YES
    local_unmask=022
    
    También la linea 120 para impedir el acceso a otras carpetas.

    chroot_local_users=YES

    Añadir la siguiente linea al final.

    allow_writeable_chroot=YES
    
    Tambien las siguientes lineas para activar el modo pasivo.

    pasv_enable=YES
    pasv_min_port=40000
    pasv_max_port=40100

Paso 4. Reiniciamos vsftpd servicio utilizando el comando a continuación,

    sudo service vsftpd restart

Paso 5. Ahora el servidor FTP estara en el puerto 21. Se debe crear usuario con el siguiente comando  para luego impedir el acceso a la carpetas del usuario FTP.

    sudo  useradd -d /home/pedro -m -s /usr/sbin/nologin

De esta manera habremos creado su carpeta home aparte del usuario. Tan solo nos queda     establecer su contraseña con el comando

    sudo passwd nombre

    Entonces el sistema nos preguntara dos veces la contraseña que queremos  asignar a nombre 
    El comando use radd permite crear muchos usuarios automaticamente mediante archivos de     comandos (scripts)


##Servidor Cluster MySQL
Requerimientos 
-	MySQL Cluster 
-	4 máquinas (virtuales o físicas)
Las cuatro máquinas se distribuirán de la siguiente forma
-	1 nodo administrador  ip 10.10.10.96/26
-	2 nodos de datos  ip’s 10.10.10.97/26 y 10.10.10.98/26
-	1 nodo SQL 10.10.10.99/26
Para la configuración del nodo administrador
Descomprimir mysql-cluster-gpl.tar.gz y mover ndb_mgm y ndb_mgmd al directorio /usr/local/bin:
-	tar -zxvf mysql-cluster-gpl-7.3.7-linux-glibc2.5-x86_64.tar.gz
-	cd mysql-cluster-gpl-7.3.7-linux-glibc2.5-x86_64
-	cp bin/ndb_mgm* /usr/local/bin
y hacerlos ejecutables
-	cd /usr/local/bin
-	chmod +x ndb_mgm*
crear el directorio y archivo de configuración
-	mkdir /var/lib/mysql-cluster
-	cd /var/lib/mysql-cluster
-	gedit config.ini
el archivo de configuración debe contener lo siguiente:
[ndbd default]
Options affecting ndbd processes on all data nodes:
NoOfReplicas=2     Number of replicas
DataMemory=80M     How much memory to allocate for data storage
IndexMemory=18M    How much memory to allocate for index storage
                   For DataMemory and IndexMemory, we have used the
                   default values. Since the "world" database takes up
                   only about 500KB, this should be more than enough for
                   this example Cluster setup.

[tcp default]
TCP/IP options:
portnumber=2202   # This the default; however, you can use any
                  # port that is free for all the hosts in the cluster
                  # Note: It is recommended that you do not specify the port
                  # number at all and simply allow the default value to be used
                  # instead

[ndb_mgmd]
Management process options:
hostname=10.10.10.96          # Hostname or IP address of MGM node
datadir=/var/lib/mysql-cluster  # Directory for MGM node log files

[ndbd]
 Options for data node "A":
                                # (one [ndbd] section per data node)
hostname=10.10.10.97          # Hostname or IP address
datadir=/usr/local/mysql/data   # Directory for this data node's data files

[ndbd]
 Options for data node "B":
hostname=10.10.10.98          # Hostname or IP address
datadir=/usr/local/mysql/data   # Directory for this data node's data files

[mysqld]
 SQL node options:
hostname=10.10.10.99          # Hostname or IP address
                                # (additional mysqld connections can be
                                # specified for this node for various
                                # purposes such as running ndb_restore)
 
Configuración del nodo de datos (la misma configuración para los dos nodos)
Descomprimir mysql-cluster-gpl.tar.gz y mover ndbd and ndbmtd al directorio /usr/local/bin:
-	tar -zxvf mysql-cluster-gpl-7.3.7-linux-glibc2.5-x86_64.tar.gz
-	cd mysql-cluster-gpl-7.3.7-linux-glibc2.5-x86_64
-	cp bin/ndbd /usr/local/bin
y hacerlos ejecutables
-	cd /usr/local/bin
-	chmod +x ndb*
crear el directorio /usr/local/mysql/data
crear un archivo de configuración
-	gedit /etc/my.cnf
debe contener lo siguiente
[mysqld]
 Options for mysqld process:
ndbcluster                      # run NDB storage engine

[mysql_cluster]
 Options for MySQL Cluster processes:
ndb-connectstring=192.168.0.102  # location of management server 
configuración del nodo SQL
comprobar si en los archivos /etc/passwd  y  /etc/group  para verificar si ya existe un grupo y usuario de sistema llamados mysql. Si no es así, crear un nuevo  mysql  grupo de usuarios, y luego añadir un  mysql  usuario a este grupo:
-	groupadd mysql
-	useradd -g mysql mysql
Descomprimir el archivo, y crear un enlace simbólico llamado  mysql  al directorio de mysql.
-	tar -C /usr/local mysql-cluster-gpl-7.3.7-linux-glibc2.5-x86_64.tar.gz
-	ln -s /usr/local/mysql-cluster-gpl-7.3.7-linux-glibc2.5-x86_64 /usr/local/mysql
pasar al directorio de mysql y ejecutar el script proporcionado para la creación de las bases de datos del sistema:
-	cd /usr/local/mysql
-	scripts/mysql_install_db --user=mysql
Establecer los permisos necesarios para el servidor y el directorio de datos de MySQL
-	chown -R root .
-	chown -R mysql data
-	chgrp -R mysql .
Copiar el script de arranque de MySQL en el directorio apropiado, volverlo ejecutable, y configurarlo para comenzar cuando el sistema operativo inicie.
-	cp support-files/mysql.server /etc/init.d
-	chmod +x /etc/init.d/mysql.server
-	update-rc.d mysql.server defaults
el archivo de configuracion para este nodo es el mismo que la configuracion de los nodos de datos.
Iniciar el MySQL Cluster

#Clusters HA
Se hicieron tres clúster HA con seis máquinas virtuales, en virtualbox, para la comunicación con el resto de computadoras la red se estableció como puente, cada una de las maquinas virtual tenían las diferentes direcciones:

##Clúster HA WEB
- ServerHTTP1: 192.168.2.194 /26
- ServerHTTP2: 192.168.2.195 /26
- IPVirtual   192.168.2.196  /26

##Cluster HA Archivos
- ServerFile1: 192.168.2.197 /26
- ServerFile2: 192.168.2.198 /26
- IPVirtual    192.168.2.199 /26

##Cluster HA FTP
- ServerFTP1: 192.168.2.201 /26
- ServerFTP2: 192.168.2.202 /26
- IPVirtual   192.168.2.203 /26

##Pacemaker y Corosync
Estos servicios se instalan en las seis maquinas virtuales, para crear el clúster se hizo con varios servicios, uno el corosync que sirve para sincronizar las maquinas, el otro era pacemaker que era el que se encargaba de si una maquina estaba activa, la instalación de estos servicios se hicieron en Ubuntu server 14.04, con la siguiente línea se instala:

- sudo apt-get update
- sudo apt-get upgrade
- sudo apt-get install pacemaker corosync

Una vez instalado los servicios se procede a generar una key para que los nodos puedan ser capaces de comunicarse unos dentro de otros, ya que esa key sirve para autentificar los paquetes enviados, para generar la key se ingresa:
- sudo corosync-keygen

Después de ingresar el comando sale una pantalla en la que hay estar tecleando varias teclas o mover el mouse, esto se hace para ser capaz de crear entropía, para tener suficientes bits aleatorios para generar la clave.

Una vez generada la clave se procede a copiar el archivo al otro servidor para poder copiarlo de manera fácil se va hacer uso del servicio ssh para poder enviar el archivo, con el siguiente comando se envía el archivo:

- sudo scp /etc/corosync/authkey root@servidor2:/etc/corosync 

Una vez que genere la clave, se tiene que modificar el archivo de corosyncque se encuentra en /etc/corosync/corosync.conf y se escribe las siguientes líneas:

###Cluster HA WEB

- totem{
- version:2 
- secauth:off 
- cluster_name: clusterweb
- transport:udpu 
- } 
- nodelist{ 
- node{ 
- ring0_addr:ServerHTTP1
- nodeid:1 } 
- node{ ring0_addr:ServerHTTP2
- nodeid:2 }
- } 
- quorum{ 
- provider:corosync_votequorum 
- two_node:1 } 

###Cluster HA Archivos

- totem{
- version:2 
- secauth:off 
- cluster_name: clusterarchivos 
- transport:udpu 
- } 
- nodelist{ 
- node{ 
- ring0_addr:ServerFile1
- nodeid:1 } 
- node{ ring0_addr:ServerFile2
- nodeid:2 }
- } 
- quorum{ 
- provider:corosync_votequorum 
- two_node:1 } 

###Cluster HA FTP

- totem{
- version:2 
- secauth:off 
- cluster_name: clusterftp
- transport:udpu 
- } 
- nodelist{ 
- node{ 
- ring0_addr:ServerFTP1
- nodeid:1 } 
- node{ ring0_addr:FTP2
- nodeid:2 }
- } 
- quorum{ 
- provider:corosync_votequorum 
- two_node:1 } 


Esto se tiene que hacer en los servidores conrrespondientes, una vez hecho esto, se modifica otra línea para que comience el servicio cuando se inicia el servidor, el archivo que se modifica se encuentra en etc/default/corosync y se agrega “yes” a la línea que tiene: 

- START=yes

Se cambia el orden de pacemaker

- update-rc.d pacemaker 20 01

Ya que corosync inicia en el 19

Después de hacer eso, se inicia el servicio de corosyncy se reiniciar el servicio pacemaker

- sudo service corosync start 
- sudo service pacemaker start 


##Configuracion Cluster HA Archivos

###Sistema de archivos clusterizados

Para el uso de sistema de archivo para clúster se usó DRBD un programa que me permite compartir información entre las dos máquinas, su funcionamiento es como el RAID 1 pero en red, lo que hace es replicar la información de un disco a otro, entonces se comienza creando las particiones correspondiente que se van a usar para el clúster ,en este caso vamos a usar un disco que agregamos y vamos a crear la partición en este disco, para crear la particiones: 
sudo fdisk /dev/sdb (sbd es el disco que se agregó)
- n -> nueva partición
- p ->primario 1-> partición #1
- enter-> si se presiona enter para poner el valor default
- enter-> se presiona enter para ocupar todo el espacio
- w -> para escribir los cambios
Esto se hace también en el servidor2 una vez creado las particiones sigue, instalar DRBD para instalar DRBD se instala con la siguiente línea, estese tiene que instalar en ambos servidores:

- sudo apt-get install drbd8-utils

Una vez instalado se procede a crear un archivo que es /etc/drbd.conf que va a contener la siguiente información

- Resource disk1 {
- device /dev/drbd0;
- disk /dev/sdb1;
- meta-diskinternal;
- on servidor1 {
- address 192.168.2.195:7788;
- } on servidor2 {
- address 192.168.2.196:7788;
- } syncer
- { rate 10M;
- }
- }

Donde disk1 va hacer el nombre del recurso, “/dev/drbd0” es la unidad que van a compartir los dos servidores y “/dev/sdb1” es la partición que creamos. Ya que se creó el recurso sigue crear los meta datos, para crear los meta datos se escribe el siguiente comando

- drbdadm create-md disk1 (El nombre del recurso)

Después de crear los meta datos sigue iniciar los servicios, en las dos máquinas para eso se escribe el siguiente comando:

- sudo service drbd start

Para ver que los dos se sincronización exitosamente, se procede a verificar que los dos están conectados con el siguiente comando:

- sudo drbd-overview

Y el resultado tendría que ser el siguiente:

- 0:disk1 Connected Secondary/Secondary Inconsistent/Inconsistent Cr-----
 
Una vez que se verifico que se sincronizaron se pone el siguiente comando para sincronizar los datos:

- sudo drbdadm -- --clear-bitmap new-current-uuid export

Poniendo otra vez el comando para verificar si se actualizaron los datos en los dos discos, una vez puesto el comando, el resultado sería:

- sudo drbd-overview
- 0:disk1 Connected Secondary/Secondary UpToDate/UpToDate C r--

Una vez que los datos ya se actualizaron lo que se hace es establecer un nodo como primario y todos los cambios que se hagan en el nodo primario se harán automáticamente en el nodo secundario, con esto podemos formatear la unidad creada para eso se escribe la siguiente linea: 

- sudo mkfs -t ext3 /dev/drbd0 

con esto ya tenemos el sistema de archivos clusterizados.

###NFS
Para que los otros cluster pudieran montar la unidad donde iban a estar almacenados sus datos, se uso nfs, para la instalacion de nfs se escribe la siguiente linea:

- sudo apt-get install nfs-kernel-server

Despues se crea una carpeta en la unidad montada por DRBD donde se van almacenar los datos de configuracion de NFS, para eso se escribieron las siguientes lineas

    cd /mnt
    mkdir data
    En ServerFile1:
    mv /var/lib/nfs/ /mnt/data/
    ln -s /mnt/data/nfs/ /var/lib/nfs
    mv /etc/exports /mnt/data
    ln -s /mnt/data/exports /etc/exports
    mkdir /mnt/export
    mkdir /mnt/export/WEB
    En ServerFile2:
    rm -rf /var/lib/nfs
    ln -s /mnt/data/nfs/ /var/lib/nfs
    rm /etc/exports
    ln -s /mnt/data/exports /etc/exports

Se agrega en el archivo de configuracion de NFS que es lo que queremos montar para los nfs clientes, para eso se modifica el archivo "exports" que ahora se encuentra en /mnt/data/exports, y se agrega la siguiente linea:


- /mnt/export/WEB   192.168.2.192/255.255.255.192(rw,wdelay,crossmnt,root_squeash,no_subtree_check,fsid=0)

Despues de agregarlo, se inicia el servicio nfs

- sudo service nfs-kernel-server start

##Configuracion Pacemaker Cluster HA Archivos
Para la configuración de pacemaker se necesitan que este iniciado los servicios pacemaker y corosync, si están iniciados se procede a escribir los siguientes comandos, los comandos que se van a ejecutaren pacemaker son:

###IPVirtual 
Para la configuración de la IPVirtual y que esta se posicione en el nodo que tiene los servicios en ese momento, si el nodo se cae estese levanta en el otro nodo, para configurar la IPVirtual se escribe el siguiente comando:

- crm configure primitive IPVirtual ocf:heartbeat :IPaddr2 params ip="192.168.2.199" cidr_netmask="26" nic="eth0:0"

###DRBD 

Para la configuración de DRBD y pacemaker se escribe las siguientes líneas:

- crm configure
- crm(live)configure# primitive res_drbd_disk1 ocf:linbit:drbd paramsdrbd_resource="disk1"
- crm(live)configure# primitive res_fs ocf:heartbeat :Filesystem params device="/dev/drbd0"directory="/mnt" fstype="ext3"
- crm(live)configure# msms_drbd_disk1 res_drbd_disk1 meta notify="true"master-max="1" master-node-max="1" clone-max="2"clone-node-max="1" 
- crm(live)configure#colocation c_export_on_drbd inf: res_fs ms_drbd_export:Master
- crm(live)configure# ordero_drbd_before_nfs inf: ms_drbd_export:promote res_fs:start
- crm(live)configure#property stonith-enabled=false crm(live)configure#property no-quorum-policy=ignore
- crm(live)configure# commit
- crm(live)configure# exit

###NFS
- crm primitive configure NFSServer lsb:nfs-kernel-server op monitor="30s"


##Configuracion Cluster HA WEB

###Montar unidad NFS
Para montar una unidad que esta dando un nfs server, se nesecita descargar el siguiente paquete:

- apt-get install nfs-common

Una vez instalado se procede a agregar la montura en /etc/fstab, para hacer eso se agrega la siguiente linea:

- 192.168.2.199:/mnt/export/WEB /mnt nfs vers=3,user 0 0

Se guarda y para ver si no hay problemas se escribe:

- mount -a

Y con esto ya tenemos la unidad montada como si fuera nuestra pero los archivos se almacenan en el servidor de archivos.

###Instalacion Apache y PHP
Para el cluster WEB se necesito la instalacion de Apache y PHP, y la instalacion de Wordpress que mas adelante se explicara como se instalo, para la instalacion de apache y php se escribe:

- sudo apt-get install apache2 php5 php5-mysql 

Despues de la instalacion se agrega al archivo de Apache, 2 hosts virtuales que van a corresponder a los dos subdominios, el archivo que se modifica es /etc/apache/sites-enabled/000-default.conf y se agrega las diferentes lineas

Para el subdominio que va a tener el archivo HTML 

- <VirtualHost *:80>
-       ServerName www1.redes2.url
-       ServerAlias redes2.url
-       DocumentRoot /mnt/html/
- </VirtualHost>

Para el subdominio que va a tener el archivo wordpress

- <VirtualHost *:80>
-       ServerName wordpress.redes2.url
-       ServerAlias www.wordpress.redes2.url
-       DocumentRoot /mnt/wordpress
- </VirtualHost>

##Instalacion y Configuracion Wordpress
Wordpress es un CMS(Sistema de Manejo de Contenido) donde se puede configurar blogs y sitios web de manera flexible. Es util para cuando se quiere que un sitio web funcione rapidamente. Antes de inciar con la instalacion, es necesario tener ya configurado un servicio LAMP(Linux, Apache, MySQL, APACHE) en tu servidor. 

###Paso 1. Crear una base de Datos Mysql y un usuario para Wordpress. 
        Para empezar, se necesita iniciar sesion con root en MySQL con el siguiente comando:
          mysql -u root -p
        Se le solicitara la contraseña, se ingresa. Luego, crear la base de datos:
          CREATE DATABASE wordpress;
        Toda sentencia de MySQL debe finalizar con punto y coma(;), Detrás, vamos a crear una cuenta de usuario de MySQL:
          CREATE USER wordpressuser@localhost IDENTIFIED BY 'password'
        En este momento, ya se tiene la base de datos y la cuenta de usuario, cada uno hechos especificamente para 
        WordPress. Sin embargo, el usuario no tiene acceso a la base de datos. Para solucionarlo se da acceso al usuario 
        con el siguente comando:
          GRANT ALL PRIVILEGES ON wordpress.* TO wordpressuser@localhost;
        Ahora ele usuario ya tiene acceso a la base de datos. Se necesita hacer un flush a los privilegios para 
        que la instancia de MySQL sepa acerca de los cambios que hemos realizado sobre los privilegios recientemente:
          FLUSH PRIVILEGES;
        Configurado todo esto, podemos salir de la linea de comandos de MySQL escribiendo:
          exit

###Paso 2. Descargar WordPress
        A continuacion, se descargan los archivos Wordpress de su sitio web. Se debe de actualizar nuestro indice 
        local de paquetes:
          sudo apt-get update
        Y tras ellos, deberemos obtener los dos paquetes necesarios:
          sudo apt-get install php5-gd libssh2-php
        Esto permitirá que trabajes con imagenes. que isntales plugins y que actualices porciones de tu sitio usando 
        tus credenciales SSH para logarte.
        
###Paso 3. Configurar WordPress
        Ya descargado, se descomprime el archivo quedandose una carpeta llamada wordpress, esta carpeta se copia en el 
        directorio de Apache(se debe de tener ya instalada Apache) "/var/www/html". Una vez copiada la carpeta se debe 
        de crear el archivo de configuracion. WorPress ya trae un archivo de configuracion de ejemplo, este se copia a 
        la configuracion por defecto para que WordPress reconozca al archivo. Se hara así:
          cp wp-config-sample.php wp-config.php
        Ahora que se tiene un archivo de configuracion con el que trabajar, se abre con un editor de texto:
          nano wp-config.php
        Abierto se debe de encontrar la configuraciones de DB_NAME, DB_USER, DB_PASSWORD para que WordPress pueda 
        conectar y auntenticarse correctamente en la base de datos creada.
          / ** MySQL settings - You can get this info from your web host ** //
          
          /** The name of the database for WordPress */
          define('DB_NAME', 'nombre de la base de datos');
          
          /** MySQL database username */
          define('DB_USER', 'nombre de usuario');
          
          /** MySQL database password */
          define('DB_PASSWORD', 'contraseña');

###Paso 4. Completar la instalación
        Ahora que se tiene los archivos en su sitio y tu software esta configurado, se completa la instalación a 
        través de la interfaz web. En el navegador web, dirigirse a la URL de tu servidor web:
          http://nombre_DNS_o_IP
        Veras la página inicial de configuración, donde se creara una cuenta inicial de administrador de tu sitio 
        web. Rellenar la informacion que te pide. Cuando se haya finalizado, se da click en el botón de instalar 
        abajo. WordPress confirma la instalación y te pide que accedas con la cuenta que se acaba de crear. 
        Presionar el botón debajo y después rellenar los campos necesarios para acceder con la información de tu 
        cuenta.
        
        Tras esto se vera la información de WordPress y se podra comenzar a montar su web o blog.

###Configuración Pacemaker Cluster HA WEB
Para la configuración de pacemaker se necesitan que este iniciado los servicios pacemaker y corosync, si están iniciados se procede a escribir los siguientes comandos, los comandos que se van a ejecutaren pacemaker son:

###IPVirtual

Va hacer la ip virtual con el que se van a comunicar las demás máquinas para poder acceder al Apache.
Para la configuración de la IPVirtual y que esta se posicione en el nodo que tiene los servicios en ese momento, si el nodo se cae estese levanta en el otro nodo, para configurar la IPVirtual se escribe el siguiente comando:

- crm configure primitive IPVirtual ocf:heartbeat :IPaddr2 params ip="192.168.2." cidr_netmask="26" nic="eth0:0"

###Apache 

Para la configuración de Apache y que se inicie en el otro servidor si se cae uno de los nodos, es el siguiente:

- crm configure primitive Apache lsb:apache2 op monitor interval=”15s”meta target_role=”Started”

##Configuracion HA FTP

###Montar unidad NFS
Para montar una unidad que esta dando un nfs server, se nesecita descargar el siguiente paquete:

- apt-get install nfs-common

Una vez instalado se procede a agregar la montura en /etc/fstab, para hacer eso se agrega la siguiente linea:

- 192.168.2.199:/mnt/export/WEB /mnt nfs vers=3,user 0 0

Se guarda y para ver si no hay problemas se escribe:

- mount -a

Y con esto ya tenemos la unidad montada como si fuera nuestra pero los archivos se almacenan en el servidor de archivos.

###Proftpd

Para instalar Proftpd se escribe la siguiente linea

- apt-get install proftpd

Despues de instalar el paquete se procede a crear los usuarios, los usuarios que se van a usar son:

- adduser userhtml 
- adduser userword

userhtml es el que se va a encargar de subir,borrar,modificar el subdominio html
userword es el que se va a encargar de subir,borrar,modificar el subdominio wordpress

Despues de agregar los usuarios, hay que indicarles en que carpetan van acceder por defecto para eso, se modifica el
achivo de proftpd que se encuentra en /etc/proftpd/proftpd.conf y se busca la linea que esta comentada que dice "DefaultRoot"
una vez encontrado se desmarca y se agregan las siguientes lineas:

- DefaultRoot       /mnt/html userhtml
- DefaultRoot       /mnt/wordpress userword

Y con esto cuando acceda un cliente FTP, se va a dirigir a su carpeta correspondiente.

###Configuración Pacemaker Cluster HA FTP

###IPVirtual 

Va hacer la ip virtual con el que se van a comunicar las demás máquinas para poder acceder al FTP.
Para la configuración de la IPVirtual y que esta se posicione en el nodo que tiene los servicios en ese momento, si el nodo se cae estese levanta en el otro nodo, para configurar la IPVirtual se escribe el siguiente comando:

- crm configure primitive IPVirtual ocf:heartbeat :IPaddr2 params ip="192.168.2.203" cidr_netmask="26" nic="eth0:0"

###FTP

- crm configure primitive FTP lsb:proftpd op monitorinterval="30s"

##El área de desarrollo
En este segmento de la red se tienen servidores HTTP (apache) y MYSQL (Mysql-server) para poder hacer las pruebas
correspondientes con la página de wordpress y luego poder actualizar cualquier cambio ya definitivo que quiera hacerse a los
archivos de la página. Para el funcionamiento correcto del área de desarrollo se utilizaron los siguientes complementos:

###Cliente FTP
También se necesita de un cliente FTP para poder compartir los archivos de wordpress con el cluster de archivos, que es donde
los toma el cluster HTTP, para lo que se ha usado la implementación de FileZilla, instalado con el comando apt-get installa filezilla.

###Servidor SSH
Este servicio se implementa con la instalación del paquete openssh-server con el comando apt-get install openssh-server, ésto
para poder brindarle acceso a los empleados, en la capa de acceso, al área de desarrollo, en caso de que necesitaran hacer
alguna actualización ya sea en la base de datos o en los archivos de configuración del servidor HTTP.

###Servidor-Cliente VNC
El servicio de escritorio remoto se implementó también para poder brindar acceso a los empleados del área de acceso en caso de
que se necesitara de una interacción más visual con el área de desarrollo a la hora de implementar cambios en la página de
wordpress. Con esto también se les brinda la opción de usar el cliente FTP para poder actualizar los archivos de los servidores
de producción. Para esto se instaló Vino en el área de desarrollo, y KRDC en un empleado de la capa de acceso como cliente VNC,
éste ultimo se instala con el comando apt-get install KRDC.

1. La instalación del servidor VNC se hace desde consola tecleando:
  - sudo apt-get install vino.
2. Para configurar el servidor hay que acceder a las preferencias de compartición de escritorio con el siguiente comando:
  - vino-preferences.
3. La configuración empleada en el servidor vnc es:
  - Permitir a otros usuarios ver mi escritorio.
  - Permitir a otros usuarios controlar mi escritorio.
  - Requerir que el usuario introduzca una contraseña: “vyatta”.
4. Para permitir acceso remoto se ingresa el siguiente comando:
  - gsettings reset org.gnome.Vino network-interface.
5. Por último se inicia el servicio con los comandos:
  - export DISPLAY=:0.0
  - /usr/lib/vino/vino-server &

##Sincronización con el área de producción
Desde el área de desarrollo era necesario poder actualizar tanto los archivos web para el servicio web clusterizado del área de producción con el servicio clusterizado de base de datos.

###Sincronización de los servicios Mysql
Para ésto se ha creado un script que primero obtiene una copia de la base de datos del área de desarrollo, luego se borra la
base de datos del cluster en el área de producción (esto para evitar cualquier pérdida de integridad en la base de datos), y
por último se restauraba en éste la copia anteriormente creada, para ésto se implementaron las siguientes instrucciones:

    FECHA=$(date +"%d%m%Y")
    mysqldump -uwordpressuser -ppassword wordpress > /home/ubuntu/Backups/$FECHA.sql
    sed -i '1i use wordpress;' >> /home/ubuntu/Backups/$FECHA.sql
    sed -i '1i create database wordpress;' >> /home/ubuntu/Backups/$FECHA.sql
    sed -i '1i drop database wordpress;' >> /home/ubuntu/Backups/$FECHA.sql
    mysql -uusuariowp -h 192.168.2.98 -paleXela wordpress < /home/ubuntu/Backups/$FECHA.sql

###Sincronización de los archivos web
Para ésto se utiliza el cliente FTP, tal y como se describió en la sección anterior. Por medio de filezilla se accede al
usuario correspondiente, bien sea al usuario de administración de wordpress o al usuario de administración html y luego
reemplazar allí los archivos necesarios.

Para esto se tuvo que hacer la previa instalación y configuracón de WordPress que se detallo anteriormente.

##Dominio de Internet con No-Ip
Para el acceso a nuestra red desde internet se ha registrado un dominio temporal de internet en la página de No-Ip para lo cual
es necesario hacerlo desde un ordenador que salga a internet con la misma ip pública que la del servidor web, de tal forma que
despues sea posible desde el router redireccionar las peticiones web a los servicios correspondientes.

Para tales efectos es necesario 

- Crear una cuenta en https://www.noip.com.
- Agregar un nuevo host, colocar el nombre y dominio deseado.
- Si la dirección ip del servidor a redireccionar es asignada de forma manual, es necesario instalar la aplicación de no-ip2  
  para que se encargara de actualizar la ip con el servicio por si recibía alguna ip distinta a la que tenía al inicio. Esto
  también es posible hacerlo desde el router, en la parte de ruteo dinámico, en donde se configura la cuenta no-ip y el host
  para que él lo actualice.
- Por último se redireccionan los servicios necesarios desde el router, por ejemplo HTTP, FTP y SSH hacia los equipos (la ip o
  el nombre de equipo) que prestan el servicio. De esta forma será posible poder acceder a los servicios configurados desde
  internet.

