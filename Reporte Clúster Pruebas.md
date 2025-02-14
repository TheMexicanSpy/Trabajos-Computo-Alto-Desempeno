# Reporte de Creación de Clúster de base de datos y pruebas con Sysbench

En este documento se denotan los pasos a seguir para crear un nodo de clúster de base de datos,
así como las pruebas que se pueden realizar para analizar el tiempo que
ocupan para procesar datos, así como la cantidad de recursos usados en dicho
periodo de tiempo.

## Conceptos previos
INSERTAR DEFINICIONES DE CLUSTER DB, NODO DE CLUSTER,
Y SYSBENCH

## Instalación de Cygwin Terminal en host
Una herramienta útil para realizar los siguientes pasos es tener una
terminal remota en nuestro host que pueda mandar comandos mediante SSH
desde nuestra máquina host.

[Aquí](https://www.cygwin.com/) puede encontrar el link donde se puede
conseguir el instalador de Cygwin. Aunque probablemente al intentar ingresar
obtenga una página en blanco que ponga:
> **Forbidden**

En ese caso le recomiendo de plano cambiar de red, pues no sabría decirle con exactitud
la razón por la cuál sucede esto...

<sup>Yo lo descargué desde la red de la universidad...</sup>

## Instalación del clúster de base de datos

La base de datos a instalar se realizará desde un sistema Linux. Puede realizarse
desde una versión Ubuntu Lts o de una versión Ubuntu Server. En mi caso por falta de espacio
para instalar más máquinas virtuales ocuparé Ubuntu Lts que ocupo
en otras materias.

Una vez instalada la máquina virtual utilizaremos varios comandos de consola
que nos instalarán las herramientas necesarias, tales como:
```
apt -y install net-tools
apt -y install software-properties-common
apt update
apt -y install mariadb-server mariadb-client galera-4
apt -y install galera-arbitrator-4
apt -y install mariadb-client libmariadb3
```
Como se comentó al principio, usted puede ejecutar estos comando desde su terminal
de Cygwin, solo debe asegurarse que cononzca la dirección IP de su sistema Linux
virtual y redirigir un puerto con el que pueda ejecutar el siguiente comando
en Cygwin:

`$ ssh briceño@127.0.0.1 -p 2222`, donde:
* `briceño`: es el usuario de Linux registrado,
* `127.0.0.1`: La dirección IP de su máquina virtual, cambie por la suya,
* `-p 2222`: El puerto configurado para redirigir la conexión con la red.

Después se le preguntará la contraseña del usuario y porfin podrá acceder a la
consola de forma remota.

El proceso de instalación dura unos minutos dependiendo de las características de
su máquina virtual. Una vez terminado, necesitará cambiar la configuración en un
archivo específico, para ello digite:

`cd /etc/mysql/mariadb.conf.d/`

Ya ubicados en esta carpeta digite el comando `ls -al` para revelar todos los archivos
dentro de esta y debe existir un `60-galera.conf`. 

Hay que modificar su contenido y para ello el comando `vi 60-galera.cnf`.

Esto abrirá el editor de texto y debe pegar el siguiente texto:

```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://192.168.56.101"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="192.168.56.101"
wsrep_node_name="nodo1"
```

Básicamente, se configura el clúster y se le asigna una dirección IP, para poder
usar el clúster debemos configurar la dirección IP de la máquina también:
* En la pestaña *Dispositivos* > *Red* > *Preferencias de red...* saldrá un menú.
* En la sección **Conectado a:** seleccione *Adaptador solo anfitrión*.
* En la sección **Nombre:** seleccione la primera opción de *VirtualBox Host-Only Ethernet Adapter*.

Usted puede checar que dirección IP fue seleccionada para este nuevo adaprador
en la primera ventana que aparece al inicializar VirtualBox. copie esta dirección IP
y asegurese de cambiar los campos dentro de `60galera.conf` con esta dirección IP.

En mi caso la dirección IP es igual a la que viene en el código, así que
ya estoy bien configurado.

Si todo sale bien, al ejecutar `systemctl status mysql`, debe aparecer lo siguiente:

![1](https://github.com/user-attachments/assets/5860916f-4cd2-451f-a940-09de14f8c170)

Se puede observar un `Status: "Taking your SQL requests now..."` que indica
que la base de datos está funcional y no hay conflictos en las configuraciones.

Ejecutemos el comando `mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"`,
donde básicamente hacemos un consulta del tamaño del clúster, que en este caso es de 1:

![2](https://github.com/user-attachments/assets/b2719d67-b186-4cfc-a103-bbd4c64e8a99)

O tambíen puede ejecutar `mysql -u root --execute="SHOW GLOBAL STATUS WHERE Variable_name IN ('wsrep_ready', 'wsrep_cluster_size', 'wsrep_cluster_status', 'wsrep_connected');"`, 
que ofrece más detalles al respecto:

![3](https://github.com/user-attachments/assets/f9590cf4-0eab-4374-99ec-2de5bcc764d3)

