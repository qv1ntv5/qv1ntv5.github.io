---
layout: post
title: Hardening de sistemas
subtitle: Teoría y ejemplos del concepto de hardening.
tags: [blueteam]
---
# 1. Preparación. 

Nos descargamos la máquina del enlace: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/ y descargamos ![[Pasted image 20220208094223.png]].

Seguidamente comparamos el hash extraído con el hash proporcionado por la página web.


<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220208094320.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220208094340.png' | relative_url }}" text-align="center"/>
</div>

Y podemos observar que son idénticos aunque también podemos meter ambos strings en un fichero y pedir que se extraiga el contenido del fichero eliminando los repetidos. Si sólo se extrae una línea, significa que ambas caenas son idénticas y por tanto el archivo se ha descargado íntegro.

<br />

# 2. Montado manual de la máquina Debian.

## 2.1. Introducción.

Vamos a instalar de manera manual la máquina recién descargada.

Después de haber configurado la estructura de la máquina virtual en VirtualBox (unidad óptica, eliminar disquetes, ram, procesador, etc) iniciamos la instalación de la máquina:

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220208205548.png' | relative_url }}" text-align="center"/>
</div>

Seleccionamos idioma, distribución de teclado, región, etc.
Introducimos el nombre de la máquina: debianVM.
No introducimos nombre de dominio (Active DIrectory) porque no pertenecemos a ninguno.
Introducimos contraseña de súper usuario, nombre de cuenta de usuario, contraseña de la cuenta de usuario y comenzamos el particionado manual del disco duro.

<br />

## 2.2. Particionamiento manual del disco duro.

Las particiones no son más que divisiones de la memoria que hay en el disco duro, existen las **particiones primarias** de las cuales puede haber hasta un máximo de cuatro y el resto son  **particiones lógicas**. 

Las particiones tienen criterios/permisos propios que priman sobre los permisos de la información que está contenido en ellas. Por eso, como una medida de seguridad, tiene sentido dividir el sistema de ficheros de Linux (las carpetas que se encuentran dentro del directorio raíz: **/**) para mayor seguridad en distintas particiones.

Para el caso que nos ocupa, que es configurar un servidor web, nos interesa serparar las siguentes carpetas:

- Directorio **/boot**, que contiene los elementos necesarios para el correcto arranque del sistema operativo.
- Directorio **/var**, que contiene los archivos que componen el sitio web.
- Directorio **/tmp**, que tiene permisos de lectura, escritura y ejecución para cualquier usuario y por tanto supone una brecha de seguridad bastante grande.


Para ello seguimos el siguiente procedimiento. Para empezar, seleccionamos el particionado manual de discos:

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220208210756.png' | relative_url }}" text-align="center"/>
</div>


Seleccionamos nuestro disco duro y creamos una nueva tabla de particiones con todo el espacio. Seguidamante seleccionamos el espacio libre y empezamos a configurar cada partición:

- **/boot**: La carpeta /boot será la carpeta a la que dedicaremos la primera partición primaria de 1 GB que alojaremos al principio del disco y que tendrá las siguientes características:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220208212947.png' | relative_url }}" text-align="center"/>
	</div>
	
	Notemos que para /boot la carpeta de arranque 
	
	<br />
	
- **/**: La carpeta raíz le seguirá a la anterior como otra partición primaria de 10 GB que situaremos al principio justo después de la carpeta /boot:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220208213111.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **/var**: La carpeta var se alojará como la tercera y última partición primaria de 4 GB que volveremos a situar al principio, después de las dos particiones anteriores:

	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220208213247.png' | relative_url }}" text-align="center"/>
	</div>

	<br />
	
- **/tmp**: Configuramos la partición dedicada a la carpeta /tmp, que será la primera partición lógica situada también al principio:

	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220208213859.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **/swap**: Por último, generamos el área de memoria dedicada al intercambio:


	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220208213959.png' | relative_url }}" text-align="center"/>
	</div>
	
<br />


De manera que al final se queda como sigue:

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220208214126.png' | relative_url }}" text-align="center"/>
</div>

<br />

Observamos que tenemos 3 particiones primarias y dos por encima de 4 y que por tanto son lógicas. Finalizamos y escribimos los cambios en el disco.

</br>

## 2.3. Configuración de programas de la máquina. 

No analizamos el medio de instalación.

Configuramos la región y el gestor de paquetes.

Dejamos en blanco el proxy pues no tenemos.

Nos negamos a participar en la encuesta de compartir nuestros paquetes.

Seguidamente configuramos los programas que queremos instalar, como lo que estamos configurando es un servidor web, seleccionamos las siguientes opciones:

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220208224326.png' | relative_url }}" text-align="center"/>
</div>

Instalamos el GRUB en la partición. Y reiniciamos.

</br>

## 2.4. Modificando los permisos de la partición /tmp.

Ahora, siguiendo con el propósito inicial, vamos a modificar los permisos de la partición /tmp. Para ello vamos a editar los permisos del fichero **/etc/fstab**, se trata de un archivo de configuración que contiene y maneja información relevante de las particiones.

En primer lugar, para poder editar adecuadamente con vi ejecutamos:

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220209101942.png' | relative_url }}" text-align="center"/>
</div>

Y seguidamente nos transformamos en super usuario para poder editar el fichero en cuestion.

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220209102125.png' | relative_url }}" text-align="center"/>
</div>

Y ejecutamos: vi /etc/fstab.

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220209102228.png' | relative_url }}" text-align="center"/>
</div>

Ahí podemos observar información acerca de cada partición y lo que contiene cada partición. Por ejemplo nos informa de que la carpeta /tmp está en la primera partición lógica referida como /dev/sda5, esto es; SataDiska5 o Disco Sata 'a' 5 (como es superior a 4 no puede ser primaria, luego es lógica).

Así, protegemos la carpeta /tmp y optimizamos la solución de erroes del sistema llevando a cabo los siguientes cambios en el fichero:

<br />

- En la línea de configuración de /dev/sda5 añadimos, separados por coma, al lado de defaults, **noexec**. Y como los ficheros que tiene /tmp no nos interesan, cambiamos el 2 por un 0 ya que no quiero que esta partición se examine si hay algún escaneo del disco duro.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220209103329.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- Seguidamente, cambiamos la preferencia de análisis de archivos de la partición que contiene a /boot y / ya que queremos que la primera partición que se analize ante un escaneo del disco duro sea la partición de arranque y las demás después. 

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220209104027.png' | relative_url }}" text-align="center"/>
	</div>
	
<br />

# 3. Hardening.

## 3.1. Concepto.

El ejemplo anterior es un caso de lo que conocemos por hardening.

En informática, el **Hardening** es el proceso de asegurar un sistema mediante la reducción de debilidades o vulnerabilidades del propio sistema aplicando la configuración adecuada (recordamos que una debilidad es una malconfiguración que predispone al sistema una vulnerabilidad y una vulnerabilidad es una malconfiguración que puede ser explotable por un atacante, no toda debilidad es de hecho una vulnerabilidad, podríamos decir que una vulnerabilidad).

**El objetivo princnipal del hardening por tanto es el de entorpecer en la media de lo posible las posibilidades del atacante**.  

<br />

## 3.2. Medidas más utilizadas.

Evidentemente, se trata de un campo muy extenso y vamos a exponer aquí algunas de las medidas más utilizadas:

</br>

- **Hardening general**:

	-   Cambiar todas las claves que tengamos por defecto.
	-   Desinstalación todo el software que sea innecesario.
	-   Dar de baja a los usuarios que son innecesarios.
	-   Deshabilitar todos los servicios que no se están utilizado.
	-   Aumentar todo lo posible, la seguridad de los servicios o procesos que si tendrán que ser utilizados.
	-   Cerrar puertos que se encuentren sin uso.
	-   Utilizar **backup** (copias de seguridad  como respaldo de datos importantes.
	-   Instalar un **firewall**.
	-   Actualizar los sistemas operativos para obtener los parches de seguridad.



	Todo esto contribuirá a que el área de ataque sea más pequeña. También hay que mencionar que el Hardening igual que consiste en “desprendernos” de lo innecesario de nuestros sistemas con el fin de endurecerlo, **también nos habla de tener sumamente cuidado, en que las acciones de protección no afecten al correcto funcionamiento del mismo sistema y por consiguiente a su propósito**.

	<br />
		
- **Hardening Usuarios**:

	-   No abrir archivos desconocidos.
	-   No descargar desde paginas no oficiales
	-   Tener contraseñas robustas.
	-   Tener cuidado con los correos electrónicos.
	-   Tener nuestros sistemas operativos actualizados.
	-   Tener un programa antivirus activado.

	El endurecimiento de del sistema, no solo es cosa de los sistemas operativos o hardware… también pasa por la **concienciación de los usuarios**, unos usuarios bien concienciados acerca de los posibles problemas reales son menos propensos  a ser vulnerables y, por ende, a que lo sea nuestro sistema.
	
	<br />
	
- **Hardening contra amenazas**:

	Una **amenaza** es el término que se utiliza para referirse a los activos de vulnerabilidad, es decir, son todos aquellos elementos que son susceptibles de ser utilizados por un atacante.

	-   **Usuarios**: Mediante cursos de concienciación en la seguridad informática.
	-   **Programas malintencionados**: No descargando ningún programa de fuentes desconocidas o no oficiales.
	-   **Exploits**: Manteniendo nuestros sistemas operativos actualizados. Los exploits es código que explotan vulnerabilidades muy conocidas que rara vez están presentas en los sistemas informáticos modernos.
	-   **Intrusos**: Estableciendo permisos y niveles de acceso.

Podemos emplear la guía de hardening de CIS para ayudarnos. PDF en la misma carpeta que contiene este archivo.

<br />

## 3.3. Ejemplos de Hardening.

A continuación vamos a ver una serie de ejemplos de hardening con la máquina Debian11.

<br />

### 3.3.1. Asegurarse de que /tmp está configurado (1.1.2).

- **Descripción**:

	El directorio **/tmp** es un directorio 'world-writable' utilizado para almacenamiento temporal por todos los usuarios y algunas aplicaciones. Esto plantea un problema de seguridad ya que un atacante podría utilizar esta carpeta para subir y ejecutar archivos maliciosos.

	<br />
	
- **Revisión**:

	En primer lugar, vamos a verificar que /tmp esté montada con el comando **mount**. Este es un comando que permite efectuar acciones relacionadas con sistemas de ficheros. El comando mount a secas muestra una lista de los directorios y dispositivos en los que estos directorios están montandos. Con grep buscamos /tmp en esa lista.

	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220216101458.png' | relative_url }}" text-align="center"/>
	</div>

	Para resolver este problema, vamos a hacer que la carpeta /tmp tenga su propio sistema de archivos. De esta manera un adminsitrador puede habilitar la opción **noexec** en el mount lo que hace que /tmp se vuelva inútil a la hora de que un atacante instale código ejecutable. Para estar más seguros buscamos información sobre la partición cuyo punto de montaje es /tmp buscando /tmp en 
	
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220217162757.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Resolución**:

	Ahora que sabemos con certeza que la carpeta /tmp está en una partición separada del resto de la carpeta raiz, vamos a modificar **los permisos de la partición** para impedir quie nadie pueda ejecutar ningún archivo en la carpeta /tmp.
	
	
	Esta modificación se hace en el fichero **/etc/fstab** que contiene información sobre las particiones y los sistemas de ficheros en ella montados. 
	

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220217164406.png' | relative_url }}" text-align="center"/>
	</div>

	Esta tiene este aspecto, en él podemos comprobar que tiene el campo *options* en el que podemos introducir, seguido del default por una coma la opción *noexec*:
	
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220217165915.png' | relative_url }}" text-align="center"/>
	</div>
	
	De esta manera nos aseguramos de que no se puedan ejecutar binarios en la partición que contiene la carpeta /tmp.
	

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220217171655.png' | relative_url }}" text-align="center"/>
	</div>

	<br />
	
### 3.3.2. Asegurarse de que existe una partición separada para /var (1.1.13).

- **Descripción**: 

	Como estamos construyendo un servidor web, la carpeta **/var** tiene un papel fundamental devido al hecho de que almacenará  todos los archivos relacionados con el sitio web. Será por tanto una medida de seguridad correcta apartarlo de la partición principal para dotar a dichos archivos de una seguridad adicional.
	
	<br />
	
- **Resolución**:

	Lo correcto es separar la partición durante la instalación de la máquina.
	
	Para asegurarnos de que tenemos una partición separada en la que está montada la carpeta /var podemos utilizar dos métodos:
	
	- **mount**: Utilizar el comando *mount* para listar todos los dispositivos montados. Si existe una partición separada que cumpla la condición pedida tendrá que aparecer en dicha lista:

		<div style="text-align:center">
		<img src="{{ 'assets/img/M10/Pasted image 20220218100242.png' | relative_url }}" text-align="center"/>
		</div>

		Ahí podemos comprobar que /var está montada en el dispositivo /dev/sda3, es decir, en el sata disk 'a' 3 y es por tanto una partición primaria.

		<br />

	- **fstab**: También podemos comprobar si en *fstab*, que es un fichero que contiene información de configuración de los sistemas de ficheros almacenados en cada partición del disco duro, hay una partición que contenga la carpeta /var montada:	

		<div style="text-align:center">
		<img src="{{ 'assets/img/M10/Pasted image 20220218100548.png' | relative_url }}" text-align="center"/>
		</div>

		Y tenemos la misma información que obteníamos con el comando mount.
		
		<br />
		
### 3.3.3. Deshabililtar Automontaje (1.1.22).
	
- **Descripción**: 

	En primer lugar, cuando queremos adherir un directorio (un dispositivo de almacenamiento externo) al sistema operativo debemos  antes montarlo. **Montar** un directorio consiste en situar dicho directorio en algún punto del esquema de ficheros para volverlo accesible por el usuario.
	
	Existe una herramienta, **autofs**, que monta automáticamente como un directorio cualquier unidad de almacenamiento externa que se conecte a la máquina. Esto puede permitir que cualquiera que conecte un dispositivo de almacenamiento a la máquina tenga automáticamente acceso al contenido del mismo aun careciendo de permisos para montarlo. Por tanto, el automontaje debe ser deshabilitado.

	<br />
	
-  **Auditoria**: Vamos a comprobar si **autofs** está instalado. Esto lo podemos hacer de dos maneras:

	- Utilizando la opción **is-enabled** de systemctl y pasándole como argumento el nombre del programa que queremos ver:
		
		<div style="text-align:center">
		<img src="{{ 'assets/img/M10/Pasted image 20220219103935.png' | relative_url }}" text-align="center"/>
		</div>
		
		Podemos ver en este caso que no se reconoce como un programa instalado en el sistema.
		
		<br />
		
	- Utilizando el gestor de paquetes de debian, **dpkg**, que lleva un control de qué programas están o no instalados en la máquina.

		<div style="text-align:center">
		<img src="{{ 'assets/img/M10/Pasted image 20220219104327.png' | relative_url }}" text-align="center"/>
		</div>
		
		En este caso también se nos informa de que no está instalado.
		
		<br />

- **Resolución**: En el caso de que hubiéramos encontrado que autofs estaba habilitado, lo más recomandable es deshabilitarlo a través de alguno de los siguientes comandos:

	```bash
	systemctl --now disable autofs
	```

	<br />
		
	```bash
	apt purge autofs
	```
		
	En la máquina Debian11 no hay necesidad de recurrir a esto debido a que, como enseñabamos antes, autofs no está instalado.
	
	<br />

### 3.3.4. Asegurar que sudo está instalado (1.3.1.).

- **Descripción**: **sudo** es un comando de Linux que permite a un usuario (con permisos) ejecutar un comando como el superusuario (root) o como otro usuario en función de cómo este configurado en la política de seguridad.
	
	De otra manera, para que un usuario sin privilegios tuvoiera que realizar un acción restringida debería de pasar al súperusuario para realizar dicha acción. Con sudo, se evita este traspaso y se minimiza la superficie de ataque.
	
	<br />
	
- **Auditoría**:

	Comprobamos si tenemos **sudo** instalado con **dpkg**, el gestor de paquetes de debian.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220219120916.png' | relative_url }}" text-align="center"/>
	</div>
	
	En este caso podemos ver que no tenemos instalado ni **sudo** ni el programa de configuración del comando sudo; **sudo-ldap**, que se sirve del protocolo **LDAP** para configurar el archivo **sudoers** que gestiona los permisos del comando sudo.
	
	<br />
	
- **Resolución**: Instalamos sudo o sudo-ldap desde la apt.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220219132002.png' | relative_url }}" text-align="center"/>
	</div>
	
	Y de esta manera:
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220219132221.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

### 3.3.5. Asegurar de que el log de sudo existe. 

- **Descripción**: Asegurarse de que el log para el sudo existe, esto monitorizará quién utiliza el comando y para qué.

	<br /> 
	
- **Auditoria**: Verificamos que estos log files estén configurados en los ficheros /etc/sudoers o cualquiera de los ficheros dentro de /etc/sudoers.d/

	Estos son ficheros de configuración del comando sudo y albergan configuración de logs.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220219235419.png' | relative_url }}" text-align="center"/>
	</div>
	
	Lo que hemos introducido es un comando que filtrará (grep) el contenido de /etc/sudoers y de cualquier archivo que esté dentro del directorio /etc/sudoers.d/ según un patrón que se interpretará como una expresión regular (-E) y se ignorará si son mayúsculas o minúsculas (-i).
	
	En este caso; **no sale nada porque no hay configurado un log para el sudo** (Defaults+logfile=).
	
	<br />
	
- **Resolución**: Como no tenemos configurado este fichero log creamos uno y los describimos en el fichero /etc/sudoers mediante el siguiente comando:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220220004647.png' | relative_url }}" text-align="center"/>
	</div>
	
	Y de esta forma: 
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220220012443.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

### 3.3.6. Asegurarse de que el mensaje del día está configurado apropiadamente (1.8.1.1)

- **Descripción**: El contenido del fichero **/etc/motd** es mostrado a los usuarios despues de logearse y en algunos casos funciona como un mensaje del día para usuarios autenticados. 
	
	<br />
	
- **Resolución**: Vamos a ver si este fichero existe volcando su contenido con *cat* o directamente podemos borrar dicho fichero si no lo utilizamos.

	<br />

### 3.3.7. Asegurar que las actualizaciones, los parches y software de seguridad están instalados (1.9).

- **Descripción**: Periodicamente aparecen parches que se realizan por cuestiones de seguridad o de actualizaciones de determinados programas, etc.
	
	<br />
	
- **Resolucion**: Vemos si hay algún parche de actualización con el siguiente comando:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220222104255.png' | relative_url }}" text-align="center"/>
	</div>

	En este caso no hay, pero si hubiera sencillamente haríamos un update.
	
	<br />
	
### 3.3.8. Asegurarse de que ntp está configurado. (2.2.1.4)

- **Descripción**: *ntp* es un *daemon* que implementa el protocolo NTP (Network Time Protocol). Este está diseñado para sincronizar el reloj del sistema a lo largo de todos los sistemas.

	<br />

- **Auditoría**: Para saber si *ntp* está configurado adecuadamente vamos a ejecutar los siguientes comandos:  

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220222110532.png' | relative_url }}" text-align="center"/>
	</div>
	
	Como no tenemos un fichero de configuración podemos deducir que ntp no está instalado y no hace falta hacer ninguna operación de modificación.
	
	<br />
	
- **Resolución**: Añadimos las siguientes líneas 

	<br />

### 3.3.9. Asegurarse de que el servidor HTTP no está habilitado (2.2.10).

- **Descripción**: Los servidores HTTP o web brindan la capacidad de alojar contenido del sitio web.
	
	</br>
	
- **Auditoría**: Vamos a comprobar si tenemos el servidor web habilitado:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220222214505.png' | relative_url }}" text-align="center"/>
	</div>
	
	
	<br />
	
- **Remediation**: Vamos a deshabilitar el servidor web.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220222214943.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />

### 3.3.10.  Asegurarse de que telnet no está instalado (2.3.4).

- **Descripción**: El paquete de *telnet* contiene el client que permite a los usuarios empezar conexiones a otros sistemas de una manera insegura y no cifrada. Por tanto debe ser deshabilitado.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220222223427.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Resolución**: Borramos del sistema el paquete de telnet de la apt.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220222223723.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

### 3.3.11. Asegurarse de que la redirección IP está deshabilitada.

- **Descripción**: Los flags *net.ipv4.ip_forward* y *net.ipv6.ip_forward* son utilizados para contar al sistema si pueden o no redireccionar paquetes.


- **Auditoría**: Vamos a verificar el valor de dichos flags.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220222225220.png' | relative_url }}" text-align="center"/>
	</div>
	
	Podemos ver que tiene valor 0 y por tanto no hace falta hacer nada más.
	
	<br />

### 3.3.12. Configure rsyslog (4.2.1)

#### 3.3.12.1 Asegurarse de que rsyslog está instalado (4.2.1.1.).

- **Descripción**: El *rsyslog* es un software que se recomienda para reemplazar al original *syslogd*, es un acrónimo de System Logging protocol, se trata de un protocolo estándar para enviar mensajes de registro o eventos del sistema a un servidor específico llamado servidor de syslog. 

	<br />
	
- **Auditoria**: Verificar si rsyslog está configurado con el comando: **dpkg -s rsyslog**.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220224165247.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Resolución**: Si no, lo instalamos desde la apt.

</br>

#### 3.3.12.2 Asegurarse de que el servicio rsyslog está instalado (4.2.1.2.).

- **Description**: Una vez se ha instalado el paquete rsyslog lo que queda es habilitarlo.

	<br />
	
- **Auditoria**: Vamos a verificar que rsyslog está habilitado con el siguiente comando:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220224172646.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Resolución**: Si no está habilitado ejecutamos el siguiente comando:

	```bash
	systemctl --now enable rsyslog
	```
	
	<br />

#### 3.3.12.3 Asegurarse de que el loggin está configurado. (4.2.1.3.).
	
- **Descripción**: Los archivos /etc/rsyslog.conf y /etc/rsyslog.d/\*.conf especifican reglas para el registro y qué archivos se utilizarán para registrar ciertas clases de mensajes.

	<br />
	
- **Auditoría**: Vamos a revisar el contenido de los archivos /etc/rsyslog.conf y /etc/rsyslog.d/\*.conf para asegurarnos de que se establece el registro apropiado. Además, ejecutamos el siguiente comando y verificamos que los archivos asociados están registrando información:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220224173941.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Resolución**: De lo contrario habría que configurar dichos ficheros con los archivos correspondientes y luego ejecutar:

	```bash
	systemctl reload rsyslog
	```
	
	<br />
	
#### 3.3.12.4 Asegurarse de que el fichero de configuración de rsyslog por defecto está configurado. (4.2.1.4.).

- **Descripción**: rsyslog va a crear logfiles que no existían en el sistema. Este fichero de configuración seteará los permisos que se aplicarán sobre los ficheros que rsyslog construya.

	<br />
	
- **Auditoría**: Vamos a verificar que cualquier fichero se cree con los permisos 0640. 

	![[Pasted image 20220224215620.png]]
	
	Podemos comprobar que se ha seteado con los permisos adecuados
	
	<br />
	
- **Resolución**: De no haber estado configurado adecuadamente sólo habría hecho falta incluir la linea $FileCreateMode 0640 en uno de los ficheros.

	<br />

#### 3.3.12.5 Asegurarse de que el fichero de configuración de rsyslog por defecto está configurado. (4.2.1.5.).

- **Descripción**: La utilidad rsyslog tiene la capacidad de mandar logs que colecciona a hosts remotos.

	<br />
	
- **Auditoría**: Vamos a revisar los ficheros de configuración para verificar que la información se envía a un host central:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220224220902.png' | relative_url }}" text-align="center"/>
	</div>
	
	Evidentemente, en nuestro caso no se envía ningún log a una máquina externa.
	
	<br />
	
- **Resolución**: Para añadir un host remoto al que enviar hosts utilizamos el siguiente comando:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220224221745.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
### 3.3.13. Asegurarse de que el daemon 'cron' está habilitado. (5.1.1).

- **Descripción**: El demonio 'cron' se utiliza para ejecutar jobs en el sistema de manera periódica.
	
	<br />
	
- **Auditoría**: Comprobamos con systemctl si cron está habilitado.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220224222659.png' | relative_url }}" text-align="center"/>
	</div>
	
	Como está habilitado no tenemos que hacer nada más pero de lo contrario habría que habilitarlo con systemctl como hemos hecho con otros servicios.
	
	<br />
	
### 3.3.14. Asegurarse de que el at/cron está restringido para usuarios con permisos. (5.1.8).

- **Descripción**: Para las aplicaciones **at** y **cron** existen dos ficheros de configuración, **allow** y **deny** que gestionan respectivamente quién tiene acceso o quién no lo tiene.

	Vamos a modificar o bien los ficheros /etc/at.allow (respec cron.allow) o /etc/at.deny (respec cron.allow).
	
	Si /etc/cron.allow o /etc/at.allow no existen, entonces utilizaremos /etc/at.deny y /etc/cron.deny. Cualquier usuario no definido específicamente en esos archivos puede usar at y cron. Al eliminar los archivos, solo los usuarios en /etc/cron.allow y /etc/at.allow pueden usar at y cron. Tenga en cuenta que, aunque un usuario determinado no aparezca en la lista cron.allow , los trabajos cron todavía se pueden ejecutar como ese usuario. El archivo cron.allow solo controla acceso administrativo al comando crontab para programar y modificar trabajos cron.
	
	<br />
	
- **Auditoría**: Vamos a comprobar quién tiene acceso y quién no en nuestro sistema a utilizar at y cron.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220225093514.png' | relative_url }}" text-align="center"/>
	</div>
	
	Evidentenmente nuestro sistema no tiene configurados dichos ficheros. En caso de necesidad se configurarían aportando usuarios específicos.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220225093935.png' | relative_url }}" text-align="center"/>
	</div>

	<br />
	
### 3.3.15. Asegurarse de que el at/cron está restringido para usuarios con permisos. (5.2.10).

- **Descripción**: En la configuración del servidor ssh existe la opción de que un individuo entre al sistema como root. Esto se encuentra en el fichero de configuración de sshd.
	
	<br />
	
- **Auditoría**: 

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220225094710.png' | relative_url }}" text-align="center"/>
	</div>
	
	Podemos comprobar que en el fichero */etc/sshd_config* no está la opción, y eso es equivalente a que esta tenga el valor por defecto que se puede ver más abajo gracias al comando sshd -T con el que podemos ver todos los parámetros de conexión.
	
	<br />
	
- **Resolución**: Introducimos la opción "PermitRootLogin no" en el fichero de configuración sshd_config:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220225095654.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
### 3.3.16. Asegurarse de que el at/cron está restringido para usuarios con permisos. (5.6).

- **Descripción**: El comando **su** permite que un usuario ejecute un comando o shell como otro usuario. El programa ha sido reemplazado por sudo , que permite un control más granular sobre privilegios acceso. Normalmente, cualquier usuario puede ejecutar el comando su. Al descomentar el pam_wheel.so en /etc/pam.d/su , el comando su solo permitirá a los usuarios en un grupos específicos para ejecutar su. Este grupo debe estar vacío para reforzar el uso de sudo para acceso privilegiado.

	<br />
	
- **Auditoría**: Vamos a revisar el contenido del fichero */etc/pam.d/su*.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220225100225.png' | relative_url }}" text-align="center"/>
	</div>

	Observamos que en nuestro caso existe un grupo que tiene denegado la utilización del comando su, se trata del grupo **nosu**.
	
	<br />
	
- **Resolución**: Bastaría por tanto con añadir un usuario al grupo nosu para que no puediera tener acceso al comando en cuestión.


