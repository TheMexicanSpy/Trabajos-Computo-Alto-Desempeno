# Reporte de Creación de Clúster de base de datos y pruebas con Sysbench

En este documento se denotan los pasos a seguir para crear un nodo de clúster de base de datos,
así como las pruebas que se pueden realizar para analizar el tiempo que
ocupan para procesar datos, así como la cantidad de recursos usados en dicho
periodo de tiempo.

## Conceptos previos

### ¿Qué es un clúster de base de datos?
Se trata de un grupo de servidores de bases de datos interconectados que trabajan en conjunto para mejorar el desempeño, disponibilidad y tolerancia a los fallos.

Estos servidores comparten la carga de trabajo y aseguran que los datos sean consitentes en todo el clúster.

![image](https://github.com/user-attachments/assets/76cfd7c5-dbca-4922-abdf-7db6ccb50e68)

Existen distintos tipos:

* **Clúster de Alta Disponibilidad**: Asegura que la base de datos sea accesible si alguno de los nados falla.
* **Clúster de Balanceo de Carga**: Distribuye las consultas de en múltiples nodos para mejorar el desmpeño.
* **Clúster Segmentado**: Divide los datos entre los nodos en escala horizontal, donde cada nodo tiene un subconjunto de todos los datos del clúster.

En esta práctica se utilizará la herrramienta de MariaDB y Galera para realizar el clúster de BD.

![image](https://github.com/user-attachments/assets/40d85304-2a0e-44f6-b3d7-99f094a59f88)

### ¿Qué es Sysbench?

Se trata de una herramienta de *benchmarking* usada para evaluar el desempeño de sistemas de bases de datos, sus CPU's, memoria, entradas y salidas y otros componentes.

![image](https://github.com/user-attachments/assets/1d980fdc-fcd5-405f-ab5b-79d8e8fb2d10)

Lo más importante a destacar en nuestras pruebas será:
1. **El número de transacciones**
2. **Consultas por minuto**
3. **Uso de CPU**
4. **Escritura OLTP**

## Instalación de Cygwin Terminal en host
Una herramienta útil para realizar los siguientes pasos es tener una
terminal remota en nuestro host que pueda mandar comandos mediante SSH
desde nuestra máquina host.

![image](https://github.com/user-attachments/assets/672e2247-823d-4711-956c-59f0f5b4e4fc)


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

A este punto lo único que debemos hacer es realizar las pruebas al clúster de base de datos. Ya que realizar una conexión remota a la BD puede volverse un tanto tediosa, por motivos prácticos haremos las pruebas desde el servidor del clúster.

Lo ideal es realizar las pruebas desde una computadora cliente que pueda simular una gran cantidad de peticiones y/o
que se trate de un ataque DDoS. Por motivos de la práctica nos quedaremos a ejecutar los comandos desde el servidor.

Para ello debemos ejecutar el comando `apt -y install sysbench`. Esta es la herramienta que ejecuta todas las pruebas
necesarias.

También es necesario crear la base de datos donde se realizarán las pruebas, para ello escribimos:

`mysql -uroot -p -e "create database sbtest"`

Esto nos crea un base de datos llamada *sbtest*

Ahora se explican los pasos para realizar las pruebas:

1. Se debe prepara una prueba y para ello se debe crear una tabla para ejecutarla. El comando `sysbench  --threads=1 --db-driver=mysql --mysql-user=root --events=0 oltp_read_only prepare` lo hace. Notese al final la palabra `prepare` y `oltp_read_only` es el tipo de prueba a realizar. ![prepare stmtn](https://github.com/user-attachments/assets/e8a67c02-57e6-4e07-adfe-872f68bafc56)

2. Se utiliza todo el comando anterior, pero se reemplaza la palabra `prepare` con `run` para ejecutar la prueba. ![run stmnt](https://github.com/user-attachments/assets/972bb850-e8fc-4ce9-9665-a7c2e4279953)

3. Finalmente, se debe limpiar la tabla y los datos para la siguiente prueba, ya que cada una requiere de distintas tablas que se preparan cuando se usa el comando `prepare` y puede crear conflictos si no se limpia adecuadamente. La forma rápida de hacerlo es nuevamente reemplazando la palabra `run` por `cleanup`. ![clean stmnt](https://github.com/user-attachments/assets/fc8ea052-dcf8-44c1-ae07-baaca4aea33a)


## Realizando pruebas al clúster

### 1.bulk_insert

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 90%
![bulk_insert 1 thread](https://github.com/user-attachments/assets/f35f61cd-9d5d-464f-bec4-ba64fb6ae2cc)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 160%
![bulk_insert 2 thread](https://github.com/user-attachments/assets/719c7e90-cc67-4ee8-b1b1-88635aecd727)

### 2.oltp_delete 

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 70%
![oltp_delte 1 thread](https://github.com/user-attachments/assets/9e12aab2-30d7-44ef-b695-41378af808f7)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 130%
![oltp_delte 2 thread](https://github.com/user-attachments/assets/e6059d4b-c929-4682-8b60-415f5702c99d)

### 3.oltp_insert

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 60%
![oltp_insert 1 thread](https://github.com/user-attachments/assets/12b1b8d1-8fd2-4941-99dd-68c56d83d9b3)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 70%
![oltp_insert 2 thread](https://github.com/user-attachments/assets/e27bfdc3-3d16-4057-a490-cfa9b085fe4d)

### 4.oltp_point_select

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 75%
![oltp_point_select 1 thread](https://github.com/user-attachments/assets/b4aaac82-de24-48f0-a02c-2c145662bc42)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 140%
![oltp_point_select 2 thread](https://github.com/user-attachments/assets/b0b7f890-084e-466b-9da7-f2045f6d59df)

### 5.oltp_read_only

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 90%
![oltp_read_only 1 thread](https://github.com/user-attachments/assets/4d36fc94-a0a8-44ec-98bf-343ef6fe5aba)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 160%
![oltp_read_only 2 thread](https://github.com/user-attachments/assets/d3a26d67-a493-4cdd-b0e5-39b5174c946e)

## 6.oltp_read_write

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 70%
![oltp_read_write 1 thread](https://github.com/user-attachments/assets/3a542820-7603-4fa2-a387-7b2ab61cb6a5)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 110%
![oltp_read_write 2 thread](https://github.com/user-attachments/assets/d320fd3f-dfd1-4e56-af85-4bfaf1f78ccb)

### 7.oltp_update_index

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 60%
![oltp_update_index 1 thread](https://github.com/user-attachments/assets/c6f9c50f-6c1d-4ef0-94bb-b518c87ba420)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 70%
![oltp_update_index 2 thread](https://github.com/user-attachments/assets/1aa02d63-9db1-49ca-b468-fb0b122d0d44)

### 8.oltp_update_non_index

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 60%
![oltp_update_non_index 2 thread](https://github.com/user-attachments/assets/5774aea4-aefb-4621-902d-9b041a49063d)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 70%
![oltp_update_non_index](https://github.com/user-attachments/assets/48a01d10-cfc5-45ab-91f6-f4a6993b89cb)

### 9.oltp_write_only

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 60%
![oltp_write_only 1 thread](https://github.com/user-attachments/assets/6d0a8260-2aa5-4733-bf25-7d118c2dd9a1)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 90%
![oltp_write_only 2 thread](https://github.com/user-attachments/assets/e159d3ed-ff8e-4dee-bba4-70cf32d44f67)

### 10.select_random_points

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 90%
![select_random_points 1 thread](https://github.com/user-attachments/assets/e2efc396-cf52-4e4f-b858-33b28add7960)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 160%
![select_random_points 2 thread](https://github.com/user-attachments/assets/c3ef7172-bc35-4b0f-8293-be5b0a436625)

### 11.select_random_ranges

1 Core; 2000MB; 1 Thread; Uso de CPU: +/- 90%
![select_random_ranges 1 thread](https://github.com/user-attachments/assets/adde6a9d-a1c4-42b2-9087-a4410e23b684)

2 Cores; 4000 MB; 2 Threads; Uso de CPU: +/- 160%
![select_random_ranges 2 thread](https://github.com/user-attachments/assets/d0446312-5d41-406b-be15-cf1c3fcb00cb)

Cuando se habla del porcentaje de uso de CPU con 2 cores se toma en cuenta el porcentaje distribuido entre los 2 núcleos.

## Conclusiones

Al estudiar a grandes rasgos el rendimiento del clúster podemos tomar en cuenta el uso de CPU: Existen casos drásticos
donde al usar 2 núcleos y el doble de memoria RAM, el rendimiento era casi el doble y en otros caso el rendimiento superaba drásticamente la cantidad de transacciones en cada núcleo, alivianando la carga en cada uno.

Se puede notar que en las pruebas con características aleatorias el uso de CPU es el doble que con un núcleo, lo cual tiene sentido ya que crear aleatoriedad en una computadora es, incluso, determinístico y conlleva un montón de operaciones lógicas y matemáticas.

Es importante analizar estos conceptos ya que de esto se pueden determinar los requerimientos para la base de datos de una empresa cuando sea necesario mejorar el hardware, ya sea en premisas o con infraestructura en la nube. 
