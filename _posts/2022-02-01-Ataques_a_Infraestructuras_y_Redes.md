---
layout: post
title: Ataques a Infraestructuras y Redes
subtitle: Introducción a webs, análsis de puertos.
tags: [redteam] 
---
<!-- Este tag hace que el post salga en la lista de RedTeam-->

# 1. Escaneo de puertos con Nmap.

## 1.1. Antecedentes.

### 1.1.1. Concepto de puertos.

Dentro del contexto de redes, un **puerto** es una interfaz a través de la cual se produce la transferencia de datos en una máquina (Recordamos que una interfaz es un entorno software que facilita la conexión entre dos dispositivos o entre un humano y un dispositivo, interfaz de usuario). Es decir, los puertos son los elementos que utilizan los ordenadores para comunicarse con el mundo exterior.

Existen **puertos físicos**, como las entradas USB, y **puertos digitales** aquellos que están regidos por un protocolo, es decir, cada uno de estos puertos digitales tiene una función, y dependiendo de la función el puerto atiende a unas peticiones u otras y el protocolo que se utiliza para la transferencia de datos puede ser distinto (TCP, UDP, HTTP, ICMP...) siendo a su vez distintas las pautas a seguir para comunicarse con ese ordenador mediante ese puerto. Esto quiere decir que; desde el punto de vista de la ciberseguridad, hay puertos más interesantes que otros, los puertos que tienen un protocolo cifrado como el 443 (https) son más seguros que otros y etc.

<br />

### 1.1.2. Nmap como herramienta.

Nmap es una herramienta que se utiliza para llevar a cabo un escaneo de dispositivos de una red ya sea bien de forma individual; llevar a cabo un **escaneo** de puertos de un dispositivo en particular, hasta hacer una **barrido** de todos los dispositivos conectados a una red a través de distintos mecanismos (ping, TCP handshaking, UDP procotol, etc). 

Actúa mandando paquetes a los distintos puertos digitales del dispositivo para obtener información sobre los mismos y luego muestra por pantalla un informe que contiene, en general, para un dispositivo en concreto:

- **Latencia**: Tiempo que tarda en transmitirse un paquete.
- Los puertos a los que ha tenido acceso.
- **Estado del puerto**: Abierto, cerrado, filtrado, etc.

	- **Abierto**: Acepta conexiones porque hay una aplicación escuchando peticiones en ese puerto.
	- **Cerrado**: Accesible para Nmap, pero no hay una aplicación escuchando en ese puerto.
	- **Filtrado**: Nmap no puede determinar que el puerto este abierto porque un filtrado de paquetes intercepta las sondas antes de que lleguen al equipo (un cortafuegos).
	- **No filtrado**: Puerto accesible pero no se puede determinar si está o no abierto. Común con el sondeo ACK, otro tipo de escaneo puede ayudar a discenir si está o no abierto.
	- **Abierto|Filtrado**: Incapaz de determinar si está abierto o filtrado. (Ausencia de respuesta).
	- **Cerrado|Filtrado**: Incapaz de discernir si un puerto está cerrado o filtrado, sólo en escáneres IPID pasivo.
	
- Servicio/Protocolo que rige el puerto.

**Sin embargo, este es sólo un esquema general y el comportamiento de Nmap puede alterarse con modificadores específicos que pueden asignar a Nmap una tarea que fuerze un cambio en el informe.**

<br />

## 1.2. Instalación.

Buscamos \^nmap en el apt-cache, (el gorrito es una expresión regular que hace que la búsqueda empiece siempre con la palabra que le sigue).

```bash
apt-cache search \^nmap
```

Y seguidamente apt-get install nmap.

```bash
sudo apt-get install nmap
```

<br />

## 1.3. Utilizando Nmap.

### 1.3.1. Barrido de red con Nmap.

Con Nmap podemos llevar a cabo un barrido de una red, esto es; identificar el máximo número de dispositvos activos que forman parte de una red. Llevamos a cabo el siguiente procedimiento:

- Cogemos la **IP** de un dispositivo y obtenemos de ella la **IP de red** (recordamos que en una IP hay dos partes, una parte que identifica al dispositivo, **parte variable**, y otra que identifica la red; la **parte fija**, en la que el dispositivo está).  Por ejemplo en la IP 192.168.1.140 la parte fija, que identifica a la red es 192.168.1.X (clase C) y la parte variable es 140. Estas partes variarán en función de la clase a la que la IP pertenezca.

- Utilizamos la parte fija de la IP y además le añadimos la **máscara de red** tras un slash (**/**) que al ser de clase C esta tiene la forma: 255.255.255.0. 

	Este es el resultado de pasar las cuatro partes del número binario a decimal. Pasándolo a cifras significativas obtenemos que cada 255 son 8 bits y como son 3 partes es: 8\*3=24 y por tanto el valor que define la máscara de red es 24. Si fuera de clase A sería 255.0.0.0 -> 8 \* 1 = 8 y si fuese de clase B sería de 16. Para más dudas repasar parte de redes.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211208104950.png' | relative_url }}" text-align="center"/>
	</div>

	En este caso, no hemos añadido ninguna especificacion del protocolo que Nmap utiliza para envíar paquetes a los puertos, por ejemplo: **con -sP Nmap envía paquetes con ICMP**, pero por defecto utilizará TCP.
	
<br />

### 1.3.2. Barrido de IPs en una red con Nmap.

En realidad, el comando introducido anteriormente es un caso particular del funcionamiento general del barrido de Nmap.  El barrido de red se hace especificando el dispositivo que pertenece a un red de la cual queremos hacer un barrido y la máscara de red.

Además, la máscara puede ser un número distinto de 8, 16 o 24. Estos sólo expresan el número de bits de la IP que identifican a la red, el resto identifican a cada dispositivo. 

De forma que el número de bits restantes que identifican a cada dispositivo sigue la fórmula:

```default
32 - nmáscara = nbits dispositivo
```

Y como son bits, que pueden ser o 1 o 0, entonces el máximo número de dispositivos conectados a esa red es: **2^(32 - nmascara)**. De nuevo las matemáticas son claras: mayor clase (A, B, C), menor máscara (8, 16, 24), más dispositivos.

<br />

### 1.3.3. Escaneo de una IP con Nmap.

Cuando llevamos acabo un barrido escaneamos un conjunto de IPs mediante el mecanismo que hemos resaltado en la sección anterior. Sin embargo en muchos casos estamos interesados sólo en un dispositivo de IP conocida, por tanto conviene más hacer un escaneo particular de ese dispostivo antes que hacer un barrido de la red. Eso consumirá menos tiempo y recurso y nos permitirá utilizar modificadores que de otra forma serían inaccesibles en un barrido.

Llevamos a cabo un escaneo pasándo como parámetro la IP asociada al dispositivo a Nmap.

Hay factores, como los modificadores, los privilegios o la localización que pueden afectar a la calidad del escaneo, ya que en función de estos habrá más o menos elementos intermedios o más puertos accesibles.

<br />

### 1.3.4. Modificadores importantes.

Existen modificadores que alteran el comportamiento de Nmap, entre los cuales encontramos:

- **A**: detección de versión de la aplicación y sistema operativo del dispositivo escaneado.


- **T\[0-5]**: acelera el proceso, en función del nivel seleccionado se hace el escaneo más rápido o más despacio. Un escaneo agresivo puede afectar a los recursos de la máquina víctima y puede ser utilizado como ciberataque.


- **s\[P,S,T,U,N,F,X...]**: sondeo ICMP, TCP SYN, TCP Connect, UDP, NULL, FIN, Xmas...


- **sV**: Informa de la versión del servicio del puerto escaneado.


- **sL**: Da una lista de todos los posibles IPs de una red, y su alguna IP está asociada a una página web, te lo dice. El número de ips proporcionados dependenderá de la clase de red.


- **p**: Con -p puedes asignar el rango de puertos que quieres analizar. Por defecto serán 1000, pero puedes expandir o contraer esta cantidad con -p, si sólo indicamos un valor, nmap escaneará el puerto asociado a ese valor.


Por ejemplo:

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211207094328.png' | relative_url }}" text-align="center"/>
</div>

En la imágen podemos observar el informe del escaneo y de los puertos.
Te dice qué puertos están abiertos y la versión del sistema operativo.

<br />

## 1.4. Scripts NSE. 

Son scripts que añaden funcionalidades a nmap. Estos se pueden ver en su totalidad: https://nmap.org/nsedoc/ 

Algunos de ellos están disponibles en tu propia máquina, otros hay que descargarlos. Los disponibles se pueden ver en la carpeta **/user/share/nmap/scripts**.

Hay 12 que son los más utiles que pueden obtenerse en la página: https://research.securitum.com/nmap-and-12-useful-nse-scripts/

<br />

### 1.4.2. Scripts más populares.

- **http-enum.nse**: Enumera todos los directorios / carpetas de un sitio web. Es importante especificar que un sitio web tiene un sistema de ficheros parecido al de linux, en el que existe un directorio raiz (la página principal) del que luego derivan carpetas y subcarpetas.

- **http-grep.nse**: Busca información útil de la página web proporcionada. De manera predeterminada devuelve las direcciones de correo electrónico y las direcciones IP que se encuentran en todas las subpáginas descubiertas por el script.

- **ssh-brute.nse**: Utilizado para romper las contraseñas de servicios SSH con la entrada de texto predictivo. De forma predeterminada, utiliza su propia base de datos de usuarios y contraseñas aunque también puede pasar lo que nosotros le pidamos con los comandos userdb y passdb.

- **dns-brute.nse**: El script dns-brute intenta encontrar tantos subdominios como pruebe el host utilizando los nombres de subdominio utilizados con más frecuencia.

- **http-config-backup.nse**: El script http-config-backup envía muchas consultas al servidor web, tratando de obtener una copia de la configuración del CMS que dejó el usuario o el editor de texto.

- **vulscan.nse**: Escanea vulnerabilidades en la IP proporcionada.

- **smb-enum-users.nse**: La tarea del script smb-enum-users es enumerar todos los usuarios disponibles en el sistema de Windows escaneado. Este script utiliza el protocolo MSRPC (Microsoft RPC) y dos técnicas de enumeración:

	- SAMR: esta técnica devuelve más información que el nombre del usuario, es mucho menos "ruidosa" que LSA y, debido a que utiliza una función dedicada, devolverá todas las cuentas existentes. Requiere privilegios mínimos de usuario (no requiere ningún permiso en Windows 2000);
	- LSA: además de los nombres de usuario, devuelve alias, grupos y cuentas del sistema. Requiere menos autorización de SAMR: se puede usar con privilegios de invitado, pero genera una gran cantidad de registros en el sistema escaneado.
	
	Cuando ejecutamos el script sin argumentos, utilizará las dos técnicas anteriores. Si desea limitar el escaneo a una técnica, simplemente use el argumento:

	- samronly - para escaneo solo SAMR;
	- lsaonly: para escanear solo por LSA.
	
- **http-wordpress-enum.nse**: El script http-wordpress-enum, usando la base de datos wp-themes.lst verifica qué temas y complementos se han instalado en el sitio escaneado.

- **firewalk.nse**: El script Firewalk, utilizando la técnica de seguimiento de paquetes (traceroute) y el paso de TTL, conocido como firewalking, intenta detectar reglas de firewall / puerta de enlace. Los parámetros de escaneo se pueden controlar mediante los siguientes argumentos:

	- firewalk.max-probed-ports: el número de puertos que se probarán para cada protocolo. Establecido en -1, examine todos los puertos filtrados;
	- firewalk.max-retries: la cantidad máxima de retransmisiones;
	- firewalk.recv-timeout: la duración del ciclo que recopila paquetes (en milisegundos);
	- firewalk.max-active-probes - número máximo de exploraciones paralelas;
	- firewalk.probe-timeout: validez de un solo escaneo (en milisegundos).
	

- **mysql-empty-password.nse**: El script mysql-empty-password comprueba si es posible iniciar sesión en el servidor MySQL en la cuenta raíz o anónima utilizando una contraseña vacía.

- **mysql-users.nse**: El script mysql-users se usa para listar los usuarios disponibles en el servidor MySQL. Para listar, usaremos la cuenta root con una contraseña vacía, que descubrimos en el script anterior.

- **mysql-brute.nse**: El script mysql-brute se usa para romper los datos de acceso al servidor MySQL.

<br />

# 2. Análisis de Vulnerabilidades.

## 2.1. Antecedentes.

- **Vulnerabilidad**: Una vulnerabilidad es un fallo de configuración de un sistema que puede ser utilizado por un atacante para penetrar en dicho sistema.

- **Debilidad**: Una debilidad es un fallo de configuración que puede conducir a una vulnerabilidad. A menudo una vulnerabilidad es una concatenación de múltiples debilidades.

- **Escaneo de vulnerabilidades**: Un escaneo de vulnerabilidades es una prueba en la que se emplea un programa o aplicación que busca y reporta vulnerabilidades conocidas en un determinado sistema.

<br />

## 2.2. Escaneo de vulnerabilidades con scripts de Nmap.

Podemos emplear distintos scripts de nmap para recabar información de la existencia de vulnerabilidades en la versión de los servicios que utilizan las aplicaciones que escuchan en los puertos.

Existen fundamentalmente dos scripts de detección de vulnerabilidades con nmap: vulscan y vulners.

<br />

### 2.2.1 Escaneo de vulnerabilidades con Vulscan.

#### 2.2.1.1. Instalación de Vulscan.

Vulscan es un script que detecta vulnerabilidades y puede descargarse desde github: https://github.com/scipag/vulscan

Para instalarlo debemos acudir a la carpeta /usr/share/nmap/scripts y clonar de Github el paquete con **git clone**.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220211122509.png' | relative_url }}" text-align="center"/>
</div>

Debe de quedar de tal forma que dentro de la carpeta "scripts" haya una carpeta llamada "vulscan" que contenga nuestros archivos de scripts.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220211122527.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 2.2.1.2. Utilización de Vulscan.

Basta con añadir los modificadores -sV y --script=vulscan/vulscan.nse a Nmap:

```bash
nmap -sV --script=vulscan/vulscan.nse <IP>
```

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220211122936.png' | relative_url }}" text-align="center"/>
</div>


<br />

### 2.2.2. Escaneo de vulnerabilidades con Vulners.

Se puede descargar de Github desde: https://github.com/vulnersCom/nmap-vulners aunque muchas veces es un script incluido por defecto en nuestra libreria de kali.

Se utiliza añadiendo el script a Nmap:

```bash
nmap --script=vulners.nse <IP>
```

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220211123730.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 2.3. Escaneo de vulnerabilidades con Openvas.
OpenVASS ofrece un escaneo de vulerabilidades ofreciendo una interfaz gráfica.

Falla en versiones recientes de kali por incompatibilidades con la versión de la base de datos; no tiene disponible la versión 14 de Postgres. 

Asi que hay que utilizar Ubuntu.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211209124316.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211209124418.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211209124625.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 2.4. Nikto.
### 2.4.1. Qué es Nikto.

**Nikto** es una herramienta enfocada en escanear servidores web de código abierto y uso gratuito. 

<br />

### 2.4.2. Instalación de Nikto.

Nikto puede descargarse desde github como un script de perl y también como paquete en la apt de Kali linux.

<br />

# 3. SSH y Nessus.

## 3.1. SSH.

### 3.1.1. Introducción.

SSH es un protocolo que se utiliza para proveer un servicio de administración remota entre dos máquinas de manera cifrada. Dentro del servicio de SSH existen clientes que llevan a cabo peticiones de administración remota sobre otra máquina y servidores que son los que aceptan una petición de administración remota.

Por defecto, el servicio SSH escucha en el puerto 22 siempre y cuando la máquina tenga activado el servicio **ssh**.

Generalmente establecerás una conexión ssh desde la pseudoterminal. Para ello se necesitará tener ssh instalado en ambas máquinas, concretamente hay que estar seguros de que la máquina que ejerce de servidor tenga el servidor de ssh instalado y que la máquina que lleva el papel de cliente tenga el cliente de ssh instalado.

Para hacer una conexión ssh necesitarás al menos dos elementos:

- El host del servidor, esto es la dirección IP de la máquina a la que deseamos conectarnos aunque en algunos casos también podremos referirnos a esta IP mediante el dominio.

- El nombre del usuario al que deseamos conectarnos, que puede coincidir o no con el usuario que tenemos actualmente. Si coincide puede no ser necesario especificarlo y si no coincide hay que especificar dicho cambio.

```bash
ssh username@serverhost
```

Es probable que también necesites de un elemento de autorización como una contraseña , un código QR o una clave pública para tener acceso a la máquina.

<br />

### 3.1.2. Conexión con contraseña.

Antes de iniciar una contraseña hay que asegurarse de dos cuestiones. En primer lugar, hay que asegurarse que ambas máquinas tienen ssh instalado. En concreto, hay que asegurarse de que la máquina a la que nos vamos a conectar tiene un servidro de ssh instalado y que la máquina desde la que hacemos la conexión tiene un cliente instalado.

Por otra parte, también hay que comprobar si en ambas máquinas el proceso 'ssh' está activo y corriendo, de lo contrario la conexión será rechazada. Esto puede comprobarse mediante el comando 'sudo service ssh status' y debería de salir algo parecido a:



<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211211184638.png' | relative_url }}" text-align="center"/>
</div>

De lo contrario, se puede probar a iniciar el servicio con 'sudo systemctl start ssh' aunque el problema bien podría estribar en otra causa y ahí la solución es más compleja.

Una vez que nos aseguramos de que ssh está presente en ambas máquinas y de que está activo y corriendo veámos cómo debemos proceder para poder conectarnos.

Para llevar a cabo la conexión con credenciales de contraseña. Sencillamente escribimos:

```bash
┌──(qvintvs㉿kali)-[~]
└─$ ssh root@192.168.1.139                                         
The authenticity of host '192.168.1.139 (192.168.1.139)' can't be established.
RSA key fingerprint is SHA256:j4PlB7z6NSG5/paFpGEt+18lnXUPGWlEJrfCjuN2xwc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.139' (RSA) to the list of known hosts.
root@192.168.1.139's password: 
Linux 2.6.20-BT-PwnSauce-NOSMP.
bt ~ # 
```

Como es la primera vez que la máquina kali, que hace el papel de cliente, se conecta con la máquina servidor nos aparece un aviso que nos informa de que la autenticidad del host '192.168.1.139' no ha podido ser establecida y nos da una huella de autenticación (fingerprint) en forma de clave pública RSA que se utilizará para cifrar la información esta y las próximas veces que ambas máquinas se conecten y nos pide confirmación de continuidad si confiamos en la máquina servidor.

Seguidamente introducimos la contraseña del usuario a cuya máquina nos queremos conectar y tendríamos acceso.

<br />

### 3.1.3. Acceso con clave.

Mediante el acceso con clave, el servidor no necesita de pedir una contraseña, lo cual es más interesante puesto que añade un peldaño de seguridad más.

La clave se puede comprobar en el directorio oculto .ssh en tu directorio de trabajo.

Si no hay ningún par de claves puede crearse mediante el comando:

```bash
┌──(qvintvs㉿kali)-[~]
└─$ ssh-keygen                                                       
Generating public/private rsa key pair.
Enter file in which to save the key (/home/qvintvs/.ssh/id_rsa): (en blanco lo enviará por defecto a .ssh)
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/qvintvs/.ssh/id_rsa
Your public key has been saved in /home/qvintvs/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:8rUschLe9lzw6A7ZM22GfXrCSOEAgSbHgUelFeAsn2o qvintvs@kali
The key's randomart image is:
+---[RSA 3072]----+
|   ++==o         |
|  oo*o.          |
|  .=+  .         |
|   o .  . .      |
|    o o Soo.     |
|   . . = =oO     |
|  E   + O.OoB .  |
| .     = *.*oo.  |
|         .= .o   |
+----[SHA256]-----+
                           
```

Nos pedirá que introduzcamos una contraseña para mayor seguridad pero que se puede dejar en blanco si se prefiere.

Por otra parte, ahora hace falta mandar nuestra clave pública al servidor remoto sobre el que queremos hacer la conexión mediante el comando:

```bash
┌──(qvintvs㉿kali)-[~]
└─$ ssh-copy-id <IPsshserver>
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/qvintvs/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
<IPsshserver>'s password: 
stderr is not a tty - where are you?

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.1.139'"
and check to make sure that only the key(s) you wanted were added.

                           
```

Nos pedirá que introduzcamos la contraseña del usuario al que nos queremos conectar y el proceso terminará satisfactoriamente.

Por último, si ahora nos queremos volver a conectar a dicho usuario:

```bash
┌──(qvintvs㉿kali)-[~]
└─$ ssh root@192.168.1.139                                           
Enter passphrase for key '/home/qvintvs/.ssh/id_rsa': 
Linux 2.6.20-BT-PwnSauce-NOSMP.
bt ~ # 

```

Nos pedirá la contraseña para hacer uso de la clave que hemos introducido cuando construimos la clave y nos conecta con el host.Nexus es una herramienta producida por Tenable, una empresa. Puede escanear vulnerabilidades. Tiene una versión de prueba con la que puedes hacer 16 escaneos de IP.

Lo importante para diferenciarlo de Openvas es que este último no te pide nada de la máquina que vas a analizar más allá de la IP, necesita una conectividad constante y puede provocar falsos positivos. Sin embargo con nexus puedes hacer el escaneo desde dentro de la máquina lo cual hará que tengas menos fallos. Aunque también puedes hacerlo al estilo Openvas, con credenciales o sin credenciales.

<br />

## 3.2. Nessus.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211210092148.png' | relative_url }}" text-align="center"/>
</div>

Para descargarse Nessus Essentials primero hay que logearse con tu correo electrónico, 

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220106232521.png' | relative_url }}" text-align="center"/>
</div>


<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220106232545.png' | relative_url }}" text-align="center"/>
</div>

entonces te será facilitado un código de activación del producto que decidas descargate que por defecto será Nessus mediante el correo electrónico que hayas decido poner.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220106232644.png' | relative_url }}" text-align="center"/>
</div>

De entre todas las descargas a nosotros evidentemente nos inte
Una vez tienes el código de activación debes descargar el producto teniendo en cuenta el formato de tu sistema operativo (distro de linux, 32 bits, 64 bits, etc). Después nos vamos a la carpeta donde se halla efectuado la descarga e introducimos el comando:

```bash
$ dpkg -i <nombrearchivo>.deb
```

Y tras instalarlo nos aparecerá el siguiente mensaje.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211210173121.png' | relative_url }}" text-align="center"/>
</div>

En el cual se nos informa de que el servicio de Nessus monta para nostros una aplicación web que carga en cierto puerto al que debemos conectarnos. Y tras acudir a la dirección IP indicada y meter el código de activación proporicionado al logearse debemos iniciar sesión con nombre de usuario y genrar un contraseña y esperar que se instalen unos plugins (un software especifico que ayudará al sistema operativo a manejar el programa, los plugins son a un programa o aplicación lo que los drivers son al hardware, son piezas de código que guían al sistema operativo en la gestión de un dispositivo).

<br />

Tras la instalación de los plugins, iniciaremos sesión y se nos presentará la interfaz de Nessus.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220106235123.png' | relative_url }}" text-align="center"/>
</div>


Desde New Scan podemos elegir el tipo de escaneo que podemos hacer, por ejemplo uno básico

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220106235447.png' | relative_url }}" text-align="center"/>
</div>


<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220106235544.png' | relative_url }}" text-align="center"/>
</div>


Desde la pestaña "Credentials" podemos definir si el ataque lo hacemos con permisos (white hat) sin permisos (black hat) qué tipo de permisos concedemos (public key, user:password, ...)

![[Pasted image 20220106235751.png]]

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220106235751.png' | relative_url }}" text-align="center"/>
</div>


Una vez completada la configuración, lo guardamos con save y una vez lo tengamos en la lista lo lanzamos para que el escaneo comience.

Cuando este termina podemos pulsar sobre dicho escaneo para obtener una descripción de este y además podemos elegir tener un report sobre las vulnerabilidades obtenidas:

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220107000329.png' | relative_url }}" text-align="center"/>
</div>


<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20220107000339.png' | relative_url }}" text-align="center"/>
</div>


Entre los distintos formatos de report podemos destacar entre uno completo (tipo ejecutivo, no muy técnico) y uno detallado.

<br />

# 4. Ataque MiTM con Ettercap.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211213092937.png' | relative_url }}" text-align="center"/>
</div>


<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211213100718.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211213105024.png' | relative_url }}" text-align="center"/>
</div>

## 4.1. Antecedentes.

## 4.1.1. Ataque de Man in The Middle.

Denominamos como ataque de "Man in The Middle" a la interceptión de la información enviada entre dos A y B por un tercero situado en el medio C.

```bash
					A --------------- C -------------- B
```


<br />

### 4.1.2. Ettercap.

Ettercap es una herramienta de seguridad de código abierto y gratuita para ataques man-in-the-middle en LAN. Se puede utilizar para analizar protocolos de redes informáticas y auditorías de seguridad.

Ettercap funciona poniendo la interfaz de red en modo promiscuo (https://en.wikipedia.org/wiki/Promiscuous_mode "Modo promiscuo") y envenenando ARP (https://en.wikipedia.org/wiki/Arp_poisoning "Envenenamiento por arp") las máquinas de destino. De ese modo, altera el direccionamiento de información de red y puede actuar como un 'hombre intermedio' y desencadenar varios ataques contra las víctimas. Ettercap tiene compatibilidad con complementos para que las funciones se puedan ampliar agregando nuevos complementos.


<br />

## 4.2. Preparando el ataque.

### 4.2.1. Preparando las máquinas.

Encendemos las máquinas y las ponemos en modo Red Nat y comprobamos la conectividad entre ellas con el comando 'ping' dos a dos con las tres.

Seguidamente comprobamos las tablas arp de las tres tablas con el comando 'arp -a'. Estas tablas contienen las direcciones IP con la que la máquina ha tenido alguna relación en la sesión en cuestión y las MAC de cada IP. Gracias a que hemos hecho el ping anteriormente, necesariamente estas tablas deben de contener al menos dos elementos.

(Este comando, **arp -a**, a veces no funciona correctamente de forma que hay que insistir. De todas formas se trata de un comando antiguo y algunos distros ya no tienen arp -a. En dichas máquinas se puede utilizar el comando **ip n**. SOLO LO TIENEN LAS MÁQUINAS MODERNAS.)

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211213095218.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 4.2.2. Instalando y usando Ettercap.

Instalamos ettercap-graphical. Lo buscamos en apt-cache como ettercap. Y se ejecuta con 

```bash
sudo ettercap -G
```

Hay que conocer los nombres de la tarjetas de red, eth0, lo, docker0,... Estas pueden conecerse mediante el comando ifconfig por ejemplo.

Vamos a utilizar este software para suplantar la identidad de una tarjeta de MAC de una IP.

En prime lugar ejecutamos el comando de arriba en la máquina y nos saldrá la siguiente interfaz:

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211213182455.png' | relative_url }}" text-align="center"/>
</div>

Una vez abierto esto, elejimos la tarjeta de red eth0 y le damos al primer botón arriba a la derecha y elejimos el modo promiscuo. 

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211213182914.png' | relative_url }}" text-align="center"/>
</div>

Una vez elegido nos saldrá una nueva interfaz en la que llevamos los siguientes pasos:

- En primer lugar, de izquierda a derecha, le damos al segundo botón de la izquierda para registrar todos los nuevos host visibles.
- En segundo lugar le damos al segundo botón para elegir tarjets de la lista de los host.

	- Target 1: DVL
	- Target 2: Metasplotaible2.
	
- En tercer lugar, le damos al tercer botón de arriba a la derecha para elegir el tipo de manipulación que queremos hacer, elegimos ARN poisoning.

De esta forma, Ettercap cambia la MAC de la DVL y  Metasploitable por la de la kali, esto puede comprobarse viendo las MACs de cada IP con el comando arp -a en Metasplotaible:


<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211213184149.png' | relative_url }}" text-align="center"/>
</div>

La segunda y tercera MAC son idénticas. Y si desde la DVL hiciéramos la misma operación la MAC de la Metasploitable y la Kali también serían idénticas. De esta forma, se tiene que toda la información que se envía desde la Metasplotaible hacia la DVL pasa por Kali y la Kali lo redirecciona a DVL y viceversa. De forma que el tráfico pasa por tu máquina y puede ser analizado con WireShark y se puede leer la conversación que hay entre ambas máquinas (siempre y cuando no esté cifrada).

<br />

# 5. Wireshark

## 5.1. Qué es Wireshark?

Wireshark es un analizador de paquetes de red que presenta la información que contienen dichos paquetes de la manera más detallada posible. Se puede entender a un analizador de paquetes de red como una herramienta de medida para examinar que es lo que está pasando dentro del "cable" de red.

<br />

# 5.2. ¿Cómo funciona Wireshark?

Wireshark es una herramienta de análisis y rastreo de paqetes. Captura el tráfico de la red en la red local (*todo aquel que pase por la máquina, ya sea porque esta es la destinataria, la emisora o porque mediante alguna treta interceptamos la comunicación entre dos máquinas externas, de forma que los paquetes pasan por nuestra máquina*) y almacena esos datos para su análisis fuera de línea.

Además le permite filtrar el registro antes de que comience la captura o durante el propio análisis. Así por ejemplo puede configurar un filtro para ver el tráfico TCP entre dos direcciones IP o puede configurarlo para que le muestre los paquetes enviados desde un dispositivo. 

<br />

## 5.3. Capturar paquetes de datos en Wireshark.

En primer lugar, cuando abres Wireshark tienes una pantalla que te muestra una lista de todas las conexiones de red que puedes monitorear listadas mediantes las interfaces de red activas: eth (ethernet: internet por cable), lo (loopack; un adapatador de wireshark para leer tráfico de la máquina propia), wlan (wifi), ... en función de por dónde provenga la información de interés examinaremos una interfaz u otra.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211216114841.png' | relative_url }}" text-align="center"/>
</div>

Seleccionamos una y empezamos a capturar tráfico de red que atraviese esa conexión que WireShark nos mostrará por pantalla.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211216120648.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 5.4. Analizando paquetes de datos en Wireshark.

Wireshark muestra tres paneles diferentes para inspeccionar la información de los paquetes de datos capturados. El panel superior muestra una lista de todos los paquetes de la captura. Cuando se "clicka" sobre un paquete los otros dos paneles inferiores cambian para mostrarle los detalles sobre el paquete seleccionado. 

<br />

### 5.4.1. Panel Superior: Listado de paquetes.

<div style="text-align:center">
<img src="{{ 'assets/img/M2/Pasted image 20211216130356.png' | relative_url }}" text-align="center"/>
</div>

Tal y como se puede ver en la imágen, la información que ofrece el panel superior está divido en columnas:

- **No.**: Este es el orden numérico del paquete que se capturó. El corchete indica que este paquete es parte de una conversación. (Una conversación de red es el tráfico entre dos puntos finales específicos. En este caso se refiere a una conversación IP, que sería todo el tráfico entre dos direcciones IP).

	<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216130703.png' | relative_url }}" text-align="center"/>
	</div>
	
	En este ejemplo concreto estamos capturando el tráfico entre dos máquinas y por tanto todos los paquetes del tráficos forman parte de una única conversación, pero si nuestra máquina fuese un servidor que pusiese en contacto varias máquinas, el corchete sería un marcador útil para seguir la conversación entre máquinas cuyos paquetes no son adyacentes en el panel).
	
	Además de corchetes nos podemos encontrar diversas marcas:
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216132818.png' | relative_url }}" text-align="center"/>
	</div>
	
- **Time**: Muestra el tiempo que ha pasado desde el inicio de la captura de tráfico hasta la captura del paquete en cuestión.

- **Source**: Esta es la dirección emisora del paquete.

- **Destination**: Esta la dirección para la que va dirigida el paquete.

- **Protocolo**: Este es el tipo de paquete, podría ser un paquete TCP, DNS, DHCPv6 (Protocolo de red para configurar host con IPv6) o ARP. Resulta conveniente observar que DNS es un protocolo de identificación de IPs con dominios y viceversa, pero los paquetes que están destinados a seguir este protocolo entre dos máquinas también viajan en TCP o UDP.

	Es decir, recordamos que los paquetes viajan por la red con un conjunto de cabeceras (encapsulamiento) con información relativa al paquete. Wireshark analiza los paquetes que captura desencapsulando y procesando la información del paquete en cuestión. Este encapsulamiento se hace siguiendo una jerárquía siguiendo el modelo de red TCP/IP de forma que el encabezado que permite identificar a un paquete como TCP o UDP está después de el encabezado DNS por la capa del modelo de red a la que pertenecen cada uno de los dos protocolos. Por tanto, Wireshark lee antes que se trata de un paquete DNS que TCP y por tanto lo declara como un paquete DNS aunque también sea un paquete TCP. Esto mismo ocurre con paquetes pertenecientes a otros protocolos.
	
- **Length**: Esta columna muestra el tamaño de los paquetes en **bytes**.

- **Info**: Esta columna muestra información adicional sobre el contenido del paquete, variará del tipo de paquete que sea.

<br />

### 5.4.2. Panel central: Detalles del paquete.

El panel central muestra información sobre el paquete seleccionado con el botón izquierdo en el panel del listado de paquetes de forma detallada. En concreto muestra protocolos y campos de protocolos del paquete seleccionado y a su vez esta información se muestra en forma de opciones desplegables.


<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216135756.png' | relative_url }}" text-align="center"/>
	</div>

Nos encontramos con cuatro protocolos analizados por Wireshark:

- **Frame Protocol**: 
	
	El "protocolo" frame no es un protocolo en sí mismo pero es utilizado por Wireshark como una campo en el que recopila información acerca de la captura del paquete como: peso del paquete en cable, peso del paquete capturado, tipo de encapsulación (wifi, ethernet,...), hora exacta de la captura del paquete, etc.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216140319.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Ethernet II**: 

	Muestra la información que contiene el paquete en relación a su viaje por la parte de ethernet. Todos los paquetes tienen información relativa al cable por el que está pretendido que viajen. En principio estaba pensado que todos los paquetes viajasen mediante una conexión cableada, sin embargo con el surgimiento de la wifi existe la necesidad de adaptar los paquetes a esta uneva forma de conexión, sin emabargo se edifica sobre cimientos y por tanto la nueva adapatación simula un cable de red. Por ello, todos los paquetes, viajen o no viajen por cable, contienen información de conexión ethernet.
	
	En este caso nos presenta las MACs de fuente y destino y el envoltorio.
	
	![[]]

	<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216153547.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Internet Protocol**: 

	Muestra la información del paquete en relación al protocolo IP: IPs, protocolo de transporte utilizado, etc.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216153811.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Protocolo de la capa de transporte**: 

	Este protocolo variará en función de si ha sido transportado por TCP, UDP, ICMP...
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216154559.png' | relative_url }}" text-align="center"/>
	</div>

	Generalmente ofrece el protocolo, el puerto, el tipo de paquete, su número de secuencia, de ack, etc.
	
	<br />

### 5.4.3. Panel inferior: Paquete de Bytes.

Oferce información sobre como fue capturado el paquete en hexadecimal y una leve transcripción al ASCII, aquí es dónde podemos ver, si la información no está cifrada, parte de los mensajes envíados.

Cuando está mirando un paquete que es parte de una conversación, puede hacer clic con el botón derecho en el paquete y seleccionar Seguir para ver solo los paquetes que son parte de esa conversación.

<div style="text-align:center">
	<img src="{{ 'assets/img/M2/Pasted image 20211216155411.png' | relative_url }}" text-align="center"/>
	</div>

<br />

## 5.5. Filtros de WireShark.

Wireshark posee filtros de captura que permite filtrar la información de acuerdo a un conjunto de condiciones. Existen diversos tipos de filtros, todos ellos se pueden encontrar en: https://www.wireshark.org/docs/wsug_html_chunked/ChWorkBuildDisplayFilterSection.html

