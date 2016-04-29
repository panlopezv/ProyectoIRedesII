#Proyecto II Redes II
Documentacion de los servicios levantados para el proyecto del curso de Redes II.

Universidad Rafael Landívar
Campus Quetzaltenango
Facultad de ingeniera
Ingeniera en Informática y Sistemas
Redes II
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


##Ruto dinámico en Vyos (OSPF).
Al tener instalado el sistema es necesario iniciar con el Usuario por defecto que ya viene con el sistema.  
-	Login:”vyos”
-	Password: ”vyos”

Luego de esto debemos de ver nuestras interfaces, 
-	vyatta@R1#  show interfaces

Luego de que miramos que nuestras interfaces están bien, entramos a las configuraciones del router, y les asignamos la ip para cada interfaz.
  
-	vyatta@R1#  configure
-	vyatta@R1#  set interfaces ethernet eth2 address 192.168.2.66/26
-	vyatta@R1#  commit (este comando siempre debe de ejecutarse para que los cambios sean aplicados permanentemente).

De esta forma establecemos las direcciones ip de cada interfaz si cometemos un error y deseamos quitar la ip que acabamos de asignar simplemente cambiamos el comando set  por delete. Si lo que deseamos es observar cómo está el archivo de configuración una vez hemos establecido las ips lo hacemos de forma individual u observamos todo el bloque a través de los siguientes comandos:

###Configurar OSPF area 0.
-	vyatta@R0# configure
-	vyatta@R0# set protocols ospf area 0 network 192.168.2.0/24

##Servidor LDAP
LDAP es un protocolo a nivel de aplicación que permite el acceso a un servicio de directorio ordenado y distribuido para buscar diversa información en un entorno de red.
Para cada versión de LDAP que constituye un documento de referencia:
-	RFC 1777 para LDAP v2.0
-	RFC 2251 para LDAP v3.0

LDAP es un sistema cliente/servidor. El servidor puede usar una variedad de bases de datos para guardar un directorio, cada uno optimizado para operaciones de lectura rápidas y en gran volumen. Cuando una aplicación cliente LDAP se conecta a un servidor LDAP puede:
-	Conectarse.
-	Desconectarse.
-	Buscar información.
-	Comparar información.
-	Insertar entradas.
-	Cambiar entradas.
-	Eliminar entradas.

Asimismo, el protocolo LDAP v3.0 ofrece mecanismos de cifrado (SSL, etc.) y autenticación para permitir el acceso seguro a la información almacenada en la base.

###Implementacion de LDAP con OpenLDAP.

Paso 1. Instalar dominio de servidor, herramientas con el siguiente comando:
	
	sudo apt-get install slapd ldap-utils
	
Paso 2. Configurar contraseña de administrador LDAP.

	Contraseña: "Admin123"

Paso3. Configuración de dominio. redes2.url

	dc=redes2,dc=url

Paso 4. Configuración de unidades organizacionales. Almacenadas en el archivo /home/ubuntu/Documents/inicio.ldif
	
	dn: ou=usuarios,dc=redes2,dc=url
	objectClass: top
	objectClass: organizationalUnit
	ou: usuarios
	description: Usuarios

	dn: ou=grupos,dc=redes2,dc=url
	objectClass: top
	objectClass: organizationalUnit
	ou: grupos
	description: Grupos
	
Paso 5. Cargar al servidor.

	ldapadd -x -D cn=admin,dc=redes2,dc=url -W -f /home/ubuntu/Documents/inicio.ldif
	
Paso 6. Configurar un usuario de prueba "plopez". Almacenado en el archivo /home/ubuntu/Documents/usuario.ldif

	dn: uid=plopez,ou=usuarios,dc=redes2,dc=url
	objectClass: top
	objectClass: inetOrgPerson
	objectClass: posixAccount
	objectClass: shadowAccount
	uid: plopez
	sn: Lopez
	givenName: Pablo
	cn: Pablo Lopez
	displayName: Pablo Lopez
	uidNumber: 10001
	gidNumber: 15000
	homeDirectory: /home/plopez
	loginShell: /bin/bash
	gecos: Pablo Lopez
	userPassword: 0123456bcf
	shadowExpire: -1
	shadowFlag: 0
	shadowLastChange: 10877
	shadowMax: 999999
	shadowMin: 8
	shadowWarning: 7
	mail: panlopezv@gmail.com
	o: redes2
	initials: PL
	
Paso 7. Cargar al servidor.

	ldapadd -x -D cn=admin,dc=redes2,dc=url -W -f /home/ubuntu/Documents/usuario.ldif

Paso 8. Descargar herramientas visuales para mejor gestion de los directorios.
	
	PHP LDAP Admin
		Instalación
			sudo apt-get install phpldapadmin
		Ejecución
			http://localhost/phpldapadmin/
	LAM
		Instalación
			sudo apt-get install ldap-account-manager
		Ejecución
			http://localhost/lam/
	
##Cliente LDAP
Paso 1. Descargar la aplicación, con sus dependencias:

	Sudo apt-get install libnss-ldap libpam-ldap ldap-utils –y
	
Paso 2. Luego será necesario seguir los pasos siguientes.

	Se coloca ldap://”ip del servidor ldap”
	
Paso 3. Luego se configura el dominio de la siguiente manera:

	dc=redes2, dc=url
	
Paso 4. Luego se configura el usuario del administrador:

	cn=admin,dc=redes2, dc=url
	
Paso 5. Se ingresa la contraseña del administrador.

Paso 6. Luego de finalizar la configuracion del cliente debemos ir a la configuración de nsswitch.conf con:

	Sudo nano /etc/nsswitch.conf
	
Paso 7. Configuramos el passwd,group y shadow y al final le agregamos LDAP.

Paso 8. Entramos a configurar y hay que modificar una línea de código:

	Sudo nano /etc/pam.d/common-password
	
Paso 9. Modificamos la siguiente linea de codigo:

	Password [succes=1 user_unknown=ignore default=die] pam_ldap.so use_authtok try_first_pass
	Por
	Password [succes=1 user_unknown=ignore default=die] pam_ldap.so try_first_pass
	
Paso 10. Dembos de modicar el archivo de sesiones ingresando al siguiente archivo:

	Sudo nano /etc/pam.d/common-session
	
Paso 11. Agregando una nueva sesión al archivo al final del archivo:

	session optional pam_mkhomedir.so skel=/etc/skel umask=077

Paso 12. Instalamos sysv.

	Sudo apt-get install sysv-rc-conf
	
Paso 13. Iniciamos el servicio:

	Sudo sysv-rc-conf libnss-ldap on
	
Paso 14. Reiniciamos la máquina y cercioramos si tenemos conexión con el servidor.


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


#Servidor Cluster MariaDB Galera

Cuando se trata de sistemas de bases de datos relacionales en un entorno de producción, a menudo es mejor tener algún tipo de procedimientos de réplica en su lugar. La replicación permite que sus datos sean transferidos a diferentes nodos de forma automática.

Una simple replicación maestro-esclavo es más común en el mundo de SQL. Esto le permite utilizar un servidor "maestro" para manejar la escritura de datos, mientras que varios servidores "esclavos" se pueden utilizar para leer los datos. Mientras que la replicación maestro-esclavo es útil, no es tan flexible como la replicación maestro-maestro. En una configuración maestro-maestro, cada nodo es capaz de aceptar las escrituras y distribuirlos en todo el clúster. 

MariaDB no tiene una versión estable de esta manera predeterminada, sino un conjunto de parches conocidos como "Galera" que ayuda a implementar la replicación maestro-maestro síncrono.

En esta guía, vamos a crear un clúster Galera usando Ubuntu server 14.04. Vamos a utilizar tres servidores para fines de demostración (la más pequeña) de racimo configurable, pero se recomiendan cinco nodos para situaciones de producción.

####Requerimientos

-	3 máquinas virtuales con Ubuntu server 14.04
-	VirtualBox 5.0.16
-	Red utilizada 192.168.2.64/26

Hacer el siguiente procedimiento para las tres máquinas

####Descargar MariaDB Galera Clúster

	Sudo apt-get update
	Sudo apt-get upgrade

Importamos la clave de firma GnuPG
	
	sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db

Añadir MariaDB a tu sources.list
	
	sudo apt-get install software-properties-common
	sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xcbcb082a1bb943db 
	sudo add-apt-repository 'deb http://ftp.osuosl.org/pub/mariadb/repo/10.0/ubuntu trusty main'

Instalación MariaDB Galera Clúster
	
	sudo apt-get install mariadb-galera-server

Importante: tener la misma contraseña de root en las tres máquinas.

####Configurar MariaDB Galera Clúster

Esta configuración se debe hacer en los tres nodos de datos. Por defecto MariaDB tiene un archivo de configuración en 	/etc/mysql/conf.d, nosotros crearemos un archivo, nosotros crearemos un archivo en el siguiente directorio con las configuraciones necesarias para el clúster.

	sudo nano /etc/mysql/conf.d/cluster.cnf

Copiaremos y pegaremos el siguiente texto en el archivo.

	[mysqld]
	query_cache_size=0
	binlog_format=ROW
	default-storage-engine=innodb
	innodb_autoinc_lock_mode=2
	query_cache_type=0
	bind-address=0.0.0.0

	# Galera Provider Configuration
	wsrep_provider=/usr/lib/galera/libgalera_smm.so
	#wsrep_provider_options="gcache.size=32G"
	
	# Galera Cluster Configuration
	wsrep_cluster_name="test_cluster"
	wsrep_cluster_address="gcomm://primera_ip,segunda_ip,tercera_ip"
	
	# Galera Synchronization Congifuration
	wsrep_sst_method=rsync
	#wsrep_sst_auth=user:pass
	
	# Galera Node Configuration
	wsrep_node_address="ip de este nodo"
	wsrep_node_name="nombre de este nodo"
	
La primera sección se modifica o se reafirma algunos ajustes MariaDB / MySQL que permitirán MySQL para funcionar correctamente.

La sección titulada "Galera Provider Configuration" se utiliza para configurar los componentes MariaDB que proporcionan una API de replicación writeset. Esto significa que Galera en nuestro caso, ya Galera es un wsrep (writeset replicación) proveedor.
Podemos especificar los parámetros generales para configurar el entorno de replicación inicial. En general, usted no necesita hacer demasiado para conseguir un trabajo creado sin embargo.

La sección "Configuración Galera Cluster" define el clúster que vamos a crear. En él se definen los miembros de la agrupación por dirección o dominio resoluble nombres de propiedad intelectual y se crea un nombre para el clúster para asegurarse de que los miembros se unen al grupo correcto.

La sección "configuración de sincronización de Galera" define cómo el clúster se comunican y sincronizar los datos entre los miembros. Esto se utiliza sólo para la transferencia de estado que se produce cuando un nodo se pone en línea. Para nuestra configuración inicial, simplemente estamos usando rsync, ya que hace más o menos lo que queremos sin tener que utilizar componentes exóticos.
La sección " Galera Cluster Configuration " se utiliza simplemente para aclarar la dirección IP y el nombre del servidor actual. Esto es útil cuando se trata de diagnosticar problemas en los registros y para poder hacer referencia a todos los servidores de múltiples maneras. El nombre puede ser cualquier cosa que le gustaría.

Cuando esté satisfecho con su archivo de configuración de clúster, debe copiar el contenido de cada uno de los nodos individuales.
Recuerde que para cambiar la sección " Galera Node Configuration " en cada servidor individual.
Copia de la configuración de mantenimiento de Debian

Actualmente, Ubuntu y servidores de Debian MariaDB utilizan un usuario de mantenimiento especial para hacer el mantenimiento de rutina. Algunas de las tareas que quedan fuera de la categoría de mantenimiento también se ejecutan como este usuario, incluidas las funciones importantes como parar MySQL.

Con nuestro entorno de clúster que se comparte entre los nodos individuales, el usuario de mantenimiento, que ha generado de forma aleatoria credenciales de acceso en cada nodo, no será capaz de ejecutar comandos correctamente. Sólo el servidor inicial tendrá las credenciales correctas de mantenimiento, ya que los demás intentarán utilizar sus ajustes locales para acceder al entorno de clúster compartido.

Podemos solucionar este problema simplemente copiando el contenido del archivo de mantenimiento para cada nodo individual:
En uno de sus servidores, abra el archivo de configuración de mantenimiento de Debian:

	sudo nano /etc/mysql/debian.cnf

Usted verá un archivo que tiene el siguiente aspecto:

	[client]
	host     = localhost
	user     = debian-sys-maint
	password = 03P8rdlknkXr1upf
	socket   = /var/run/mysqld/mysqld.sock

	[mysql_upgrade]
	host     = localhost
	user     = debian-sys-maint
	password = 03P8rdlknkXr1upf
	socket   = /var/run/mysqld/mysqld.sock
	basedir  = /usr

Simplemente tenemos que copiar esta información y pegarla en el mismo archivo en cada nodo.

####Iniciar el clúster

Para empezar, tenemos que detener el servicio MariaDB ejecución, de modo que nuestro grupo puede poner en línea.
	
	sudo service mysql stop
	
Cuando todos los procesos han dejado de correr, debe poner en marcha su primer nodo de nuevo con un parámetro especial:

	sudo service mysql start --wsrep-new-cluster

Con nuestra configuración de clúster, cada nodo que quiera conectarse, al menos otro nodo especificado en el archivo de configuración para obtener su estado inicial. Sin la --wsrep-new-cluster de parámetros, este comando podría fallar porque el primer nodo es incapaz de conectar con cualquier otro nodo.

En cada uno de los otros nodos, ahora puede empezar a MariaDB como lo haría normalmente. Se buscará por cualquier miembro de la lista de clúster que no está en línea. Cuando se encuentran con el primer nodo, se unirán a la agrupación.

	sudo service mysql start

#HAProxy

Las soluciones de balance de carga pueden ser tanto hardware como software. En este caso, nosotros vamos a optar por una solución software llamada HAProxy que instalaremos en otra máquina virtual independiente y que funcionará de proxy con el resto de nodos SQL de la arquitectura, a fin de repartir las peticiones.

Lo primero que tenemos que hacer es instalar una nueva máquina virtual. Esta máquina solo va a actuar de proxy balanceador por lo que lo único que tenemos que instalar es el paquete HAProxy.

	sudo apt-get install haproxy

Una vez instalado vamos a configurarlo para que balance la carga de MySQL. Para ello abrimos con un editor el fichero 	/etc/haproxy/haproxy.cfg que se ha creado con contenido por defecto en la instalación del paquete.

Esta es una posible configuración para nuestro caso:

	global
		maxconn 4096
		user haproxy
		group haproxy
		daemon
	
	defaults
		mode	http
		option	tcplog
		option	dontlognull
		retries	3
		option redispatch
		contimeout	5000
		clitimeout	50000
		srvtimeout	50000
	
	listen	mysql-cluster 0.0.0.0:3306
		mode tcp
		balance roundrobin
	
		server nodo1 192.168.2.95:3306 check
		server nodo2 192.168.2.96:3306 check
		server nodo3 192.168.2.97:3306 check

Reiniciamos el proxy indicándole donde está el fichero de configuración:

	sudo haproxy -f /etc/haproxy/haproxy.cfg
	
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

##Sesiones Activas
Se crean archivos .php en el servidor, se crean varios entre ellos:

###Paso 1: 
Crear el Archivo de coneccion al servidor de la base de datos este archivo esta dentro de una carpeta llamada class y se llama conexion.php

		<?php
		session_start();
		abstract class Conexion {
   		//*****************************************************************************
   		public function conectar(){
   	    $mysqli = new mysqli("192.168.2.98","pruebawp","aleXela","redes");
      
      	if ($mysqli->connect_errno)
         header('Location: offline.html');

      	$mysqli->set_charset('utf8');
      
     	return $mysqli;
   		}
		}
		?>
###Paso 2:
Se crea una clase para la creacion de usuarios, con los metodos para agregar usuarios.
		<?php

		require_once 'conexion.php';
		class Usuarios extends Conexion {

    	public $mysqli;
    	public $data;

    	public function __construct() {
        $this->mysqli = parent::conectar();
        $this->data = array();
    	}
    	public function usuarios() {

        $resultado = $this->mysqli->query("SELECT * FROM usuarios");

        while ( $fila = $resultado->fetch_assoc() ) {
            $this->data[] = $fila;
        }
        
        return $this->data;
    	}

    	public function nuevousuario() {
        parent::Conectar();
        $nombre = $_POST['nombre'];
        $nick = $_POST['usuario'];
        $pass = $_POST["clave"];
        // VALIDAMOS SI EXISTE EL NICK
        $resultado = $this->mysqli->query("select nick from usuarios where nick = '$nick'"); 

        $registros = $resultado->num_rows; 

        if ($registros == 0) {
            $resultado = $this->mysqli->query("INSERT INTO usuario(Nombre, nick, contrasenia) 
                VALUES('$nombre', '$nick','$pass')"); 
            	header("location: login.php");
            }

    	}
		}
###Paso 3:
Se crea una funcion para agregar un id a la sesion del usuario que se ha va a logear con el comando "setcookie" se le da el nombre y el valor para guardar en las cookies la sesion

		public function set($nombre, $valor, $idusuario) {
     	$iddesesion = $valor."ohdfur4ih";
     	var_dump ($iddesesion);
     	setcookie("nombre", $nombre);
     	setcookie("idsesion", $iddesesion);
     	$conexion = new mysqli("192.168.2.98","pruebawp","aleXela","redes");
     	$consulta = "UPDATE usuario SET idsesion = '$iddesesion' where idusuario = '$idusuario';";
     	$result = $conexion->query($consulta);
    
     	if($result->num_rows > 0)
     	{
       	$fila = $result->fetch_assoc();
       	if( strcmp($password,$fila["contrasenia"]) == 0 )
        return true;            
       	else          
        return false;
     	}
     	else
        return false;
  		}
De esta manera para mantener la sesiones abiertas, la base de datos contara con un campo dedicado al registro de las seciones, y cuando este con la sesion iniciada, sera en base a las cookis y al registro en la base de datos, que como anteriormente se menciona, este servicio esta en Alta disponibilidad, lo que garantiza las sesiones iniciadas.

###Paso 4:
Para verificar la sesion del usuario en la pagina principal se muestra el nombre del usuario con la sesion iniciada.
		/<HTML><head>
		/<title></title>
		/</head>
		/<body>
		/<h1>Hola:  <?php echo $_COOKIE["nombre"]; ?> </h1> <a href="cerrarsesion.php"> Cerrar Sesion </a>
		/<p> Aqui va el contenido de la pagina </p>
		/</body>
		/</HTML>
###Paso 5:
Como se trata de solamente una prueba, en el servidor de base de datos, se agrego la siguiente base de datos, con una sola tabla, de la siguiente manera:

		CREATE TABLE `usertbl` (
 		`id` int(11) NOT NULL auto_increment,
 		`full_name` varchar(32) collate utf8_unicode_ci NOT NULL default '',
 		`username` varchar(20) collate utf8_unicode_ci NOT NULL default '',
 		`password` varchar(32) collate utf8_unicode_ci NOT NULL default '',
 		`idsesion` varchar(20) collate utf8_unicode_ci NOT NULL default '',
 		PRIMARY KEY (`id`),
		UNIQUE KEY `username` (`username`)
		)


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

Para la parte de el área de desarrollo se tuvo que hacer la previa instalación y configuracón de WordPress que se detalló anteriormente.

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

##Servidor DHCP y Punto de acceso Wireless

En la capa de acceso se implemento un punto de acceso Wireless para que los desarrolladores/clientes tengan acceso a través de la red 172.16.1.0/24
DHCP es un servicio que usa protocolo de red de tipo cliente/servidor en el que generalmente un servidor posee una lista de direcciones IP dinámicas y las va asignando a los clientes conforme éstas van quedando libres, sabiendo en todo momento quién ha estado en posesión de esa IP, cuánto tiempo la ha tenido y a quién se la ha asignado después. Así los clientes de una red IP pueden conseguir sus parámetros de configuración automáticamente.

###Configuración de interfaz wireless en Vyos.

Paso 1. Ingresar a las configuraciones.

	configure
	
Paso 2. Configurar la interfaz wireless wlan0 como Access-Point, el canal de comunicación y el modo g (25-54 Mbps a 2.4 Ghz).

	set interface wireless wlan0 type access-point channel 11 mode g

Paso 3. Configurar SSID para identificar los paquetes pertenecientes a esta red.

	set interface wireless wlan0 ssid REDES2
	
Paso 4. Configurar autenticación con contraseña WPA-2PSK

	set interface wireless wlan0 security wpa mode both passphrase redes2url
	
Paso 5. Configurar ip estatica para la interfaz wireless.
	
	set interface wireless wlan0 address 172.16.1.1/24
	
###Configuración de servidor DHCP.
Paso 1. Configurar nombre y subred.

	set service dhcp-server shared-network-name REDES2 subnet 172.16.1.0/24

Paso 2. Configurar rango para asignar ip dinamicamente.
	
	set service dhcp-server shared-network-name REDES2 start 172.16.1.2 stop 172.16.1.254

Paso 3. Configurar router por defecto.
	
	set service dhcp-server shared-network-name REDES2 default-router 172.16.1.1

Paso 4. Configurar servidor dns.

	set service dhcp-server shared-network-name REDES2 dns-server 192.168.2.217
