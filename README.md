# ProyectoIRedesII
Documentacion de los servicios levantados para el proyecto del curso redes II

Universidad Rafael Landívar
Campus Quetzaltenango
Facultad de ingeniera
Ingeniera en sistemas e Informática
Redes 2
Ing. Dhaby Xiloj.


Proyecto No. 1
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


##Ruto dinámico en Vyatta(RIP).
Al tener instalado el sistema es necesario i con el Usuario por defecto que ya viene con el sistema.  
-	Login:”vyos”
-	 password: ”vyos”

Luego de esto debemos de ver nuestras interfaces, 
-	vyatta@R1#  show interfaces


Luego de que miramos que nuestras interfaces están bien, entramos a las configuraciones del router, y les asignamos la ip para cada interfaz.
  
-	vyatta@R1#  configure
-	vyatta@R1#  set interfaces ethernet eth2 address 10.10.10.66/26
-	vyatta@R1#  commit (este comando siempre debe de ejecutarse para que los cambios sean aplicados permanentemente).
De esta forma establecemos las direcciones ip de cada interfaz si cometemos un error y deseamos quitar la ip que acabamos de asignar simplemente cambiamos el comando set  por delete. Si lo que deseamos es observar cómo está el archivo de configuración una vez hemos establecido las ips lo hacemos de forma individual u observamos todo el bloque a través de los siguientes comandos

###Configurar Rip.
-	vyatta@R0# configure
-	vyatta@R0# set protocols rip network 10.10.10.0/24











#Quagga

Quagga es un servicio de linux que nos permite enrutar redes usando a nuestro linux como un router, para ello seguimos los siguientes pasos:

1. Para instalar quagga digitamos:

apt-get install quagga

2. Configuramos los demonios para que active zebra y ripd con el siguente comando:
    
    nano /etc/quagga/daemons  y cambiamos a yes:

    zebra=yes
    bgpd=no
    ospfd=no
    ospf6d=no
    ripd=yes
    ripngd=no



3. Después hay que entrar a la ruta cd /usr/share/doc/quagga/examples/ y copiar los archivos zebra y ripd a la ruta /etc/quagga, quedaría así:

    cp /usr/share/doc/quagga/examples/zebra.conf.sample /etc/quagga/zebra.conf
    cp /usr/share/doc/quagga/examples/ripd.conf.sample /etc/quagga/ripd.conf

4. Reiniciamos el servicio de quagga:

    sudo /etc/init.d/quagga
    
5. Ahora podremos acceder por separado con una interfaz interactiva a cada uno de los demonios. Para acceder a Zebra (Password por defecto zebra):

    Router>enabletelnet localhost 2601 o zebra

    Password: zebra
    Router>enable
    Router#conf t
    Router(config)#interface eth0
    Router(config-if)#ip address 200.100.100.1/24
    Router(config-if)#no shutdown
    Router(config-if)#exit
    Router(config)# exit
    Router# write
    Configuration saved to /etc/quagga/router.conf
    telnet localhost 2602 o ripd
    Password: zebra
    ripd> enable
    ripd# configure terminal
    ripd(config)# router rip
    ripd(config-router)# network 200.100.100.0/24
    ripd(config-router)# network 192.168.1.0/24
    ripd(config-router)# network 192.168.2.0/24
    ripd(config-router)# network 192.168.3.0/24
    ripd(config-router)# exit
    ripd(config)# exit
    ripd# write
    Configuration saved to /etc/quagga/ripd.conf

6. Después de configurar todo lo de quagga procedemos a configurar las tarjetas de red. Para eso editamos el archivo:


    nano /etc/network/interfaces
    auto eth0
    iface eth0 inet static
    address 200.100.100.1
    netmask 255.255.255.0
    network 200.100.100.0
    broadcast 200.100.100.255
    auto eth1
    iface eth1 inet static
    auto eth1.101
    iface eth1.101 inet static
    address 192.168.1.1
    netmask 255.255.255.0
    network 192.168.1.0
    broadcast 192.168.1.255
    auto eth1.102
    iface eth1.102 inet static
    address 192.168.2.1
    netmask 255.255.255.0
    network 192.168.2.0
    broadcast 192.168.2.255
    auto eth1.103
    iface eth1.103 inet static
    address 192.168.3.1
    netmask 255.255.255.0
    network 192.168.3.0
    broadcast 192.168.3.255
    Luego le damos "control mas o" y enter y después "control mas x" para salir.

7. Después de hacer estas configuraciones hay que restaurar el servicio de red con el siguiente comando:
    sudo /etc/init.d/networking restart

8. Después hay que activar el enrutamiento en GNU/Linux con el siguiente comando:

echo "1" > /proc/sys/net/ipv4/ip_forward
y para que no se borre despues de reiniciar el sistema utilizamos la siguiente linea:

    echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf

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

##Clúster HA WEB Y FTP
Para hacer el clúster HA se usaron dos máquinas virtuales, en virtualbox, para la comunicación con el resto de computadoras la red se estableció como puente, cada una de las maquinas virtual tenían las siguientes líneas:
  Servidor1: 10.10.10.195 /26
  Servidor2: 10.10.10.196 /26
Para crear el clúster se hizo con varios servicios, uno el corosync que sirve para sincronizar las maquinas, el otro era pacemaker que era el que se encargaba de si una maquina estaba activa la instalación de estos servicios se hicieron en Ubuntu server 14.04, con la siguiente línea se instala:
  sudo apt-get update
  sudo apt-get upgrade
  sudo apt-get install pacemaker corosync
Una vez instalado los servicios se procede a generar una key para que los nodos puedan ser capaces de comunicarse unos dentro de otros, ya que esa key sirve para autentificar los paquetes enviados, para generar la key se ingresa:
-sudo corosync-keygen
Después de ingresar el comando sale una pantalla en la que hay estar tecleando varias teclas o mover el mouse, esto se hace para ser capaz de crear entropía, para tener suficientes bits aleatorios para generar la clave.
Una vez generada la clave se procede a copiar el archivo al otro servidor para poder copiarlo de manera fácil se va hacer uso del servicio ssh para poder enviar el archivo, con el siguiente comando se envía el archivo:

-sudo scp /etc/corosync/authkey root@servidor2:/etc/corosync 

Una vez que genere la clave, se tiene que modificar el archivo de corosyncque se encuentra en /etc/corosync/corosync.conf y se escribe las siguientes líneas:

totem{
 version:2 
secauth:off 
cluster_name: clusterarchivos 
transport:udpu 
} 
nodelist{ 
node{ 
ring0_addr:servidor1 
nodeid:1 } 
node{ ring0_addr:servidor2 
nodeid:2 }
 } 
quorum{ 
provider:corosync_votequorum 
two_node:1 } 

Esto se tiene que hacer tanto en el servidor1 como en el servidor2, una vez hecho esto, se modifica otra línea para que comience el servicio cuando se inicia el servidor, el archivo que se modifica se encuentra en etc/default/corosync y se agrega “yes” a la línea que tiene: 

-START=yes

Después de hacer eso, se inicia el servicio de corosyncy se reiniciar el servicio pacemaker

-sudo service corosync start 
-sudo service pacemaker start 

Sistema de archivos clusterizados
Para el uso de sistema de archivo para clúster se usó DRBD un programa que me permite compartir información entre las dos máquinas, su funcionamiento es como el RAID 1 pero en red, lo que hace es replicar la información de un disco a otro, entonces se comienza creando las particiones correspondiente que se van a usar para el clúster ,en este caso vamos a usar un disco que agregamos y vamos a crear la partición en este disco, para crear la particiones: 
sudo fdisk /dev/sdb (sbd es el disco que se agregó)
n -> nueva partición
p ->primario 1-> partición #1
enter-> si se presiona enter para poner el valor default
enter-> se presiona enter para ocupar todo el espacio
w -> para escribir los cambios
Esto se hace también en el servidor2 una vez creado las particiones sigue, instalar DRBD para instalar DRBD se instala con la siguiente línea, estese tiene que instalar en ambos servidores:
sudo apt-get install drbd8-utils
Una vez instalado se procede a crear un archivo que es /etc/drbd.conf que va a contener la siguiente información

Resource disk1 {
device /dev/drbd0;
disk /dev/sdb1;
meta-diskinternal;
on servidor1 {
address10.10.10.195:7788;
} on servidor2 {
address10.10.10.196:7788;
} syncer
{ rate 10M;
}
}

Donde disk1 va hacer el nombre del recurso, “/dev/drbd0” es la unidad que van a compartir los dos servidores y “/dev/sdb1” es la partición que creamos. Ya que se creó el recurso sigue crear los meta datos, para crear los meta datos se escribe el siguiente comando
-drbdadm create-md disk1 (El nombre del recurso)
Después de crear los meta datos sigue iniciar los servicios, en las dos máquinas para eso se escribe el siguiente comando:
-sudo service drbd start
Para ver que los dos se sincronización exitosamente, se procede a verificar que los dos están conectados con el siguiente comando:
-sudo drbd-overview
Y el resultado tendría que ser el siguiente:
-0:disk1 Connected Secondary/Secondary Inconsistent/Inconsistent Cr-----
Una vez que se verifico que se sincronizaron se pone el siguiente comando para sincronizar los datos:
-sudo drbdadm -- --clear-bitmap new-current-uuid export
Poniendo otra vez el comando para verificar si se actualizaron los datos en los dos discos, una vez puesto el comando, el resultado sería:
-sudo drbd-overview
-0:disk1 Connected Secondary/Secondary UpToDate/UpToDate C r--
Una vez que los datos ya se actualizaron lo que se hace es establecer un nodo como primario y todos los cambios que se hagan en el nodo primario se harán automáticamente en el nodo secundario, con esto podemos formatear la unidad creada para eso se escribe la siguiente linea: sudomkfs -t ext3 /dev/drbd0 con esto ya tenemos el sistema de archivos clusterizados.

Configuración Pacemaker
Para la configuración de pacemaker se necesitan que este iniciado los servicios pacemaker y corosync, si están iniciados se procede a escribir los siguientes comandos, los comandos que se van a ejecutaren pacemaker son:

IPVirtual: Va hacer la ip virtual con el que se van a comunicar las demás máquinas para poder acceder al FTP y/o Apache.
Para la configuración de la IPVirtual y que esta se posicione en el nodo que tiene los servicios en ese momento, si el nodo se cae estese levanta en el otro nodo, para configurar la IPVirtual se escribe el siguiente comando:

-crm configure primitive IPVirtual ocf:heartbeat:IPaddr2 params ip="10.10.10.197" cidr_netmask="26" nic="eth0:0"

Apache: Para la configuración de Apache y que se inicie en el otro servidor si se cae uno de los nodos, es el siguiente:

-crm configure primitive Apache lsb:apache2 op monitor interval=”15s”meta target_role=”Started”

DRBD Para la configuración de DRBD y pacemaker se escribe las siguientes líneas:
crm configure
-crm(live)configure# primitive res_drbd_disk1 ocf:linbit:drbd paramsdrbd_resource="disk1"
-crm(live)configure# primitive res_fs ocf:heartbeat:Filesystem params device="/dev/drbd0"directory="/mnt" fstype="ext3"
-crm(live)configure# msms_drbd_disk1 res_drbd_disk1 meta notify="true"master-max="1" master-node-max="1" clone-max="2"clone-node-max="1" 
-crm(live)configure#colocation c_export_on_drbd inf: res_fs ms_drbd_export:Master
-crm(live)configure# ordero_drbd_before_nfs inf: ms_drbd_export:promote res_fs:start
-crm(live)configure#property stonith-enabled=false crm(live)configure#property no-quorum-policy=ignore
-crm(live)configure# commit
-crm(live)configure# exit

