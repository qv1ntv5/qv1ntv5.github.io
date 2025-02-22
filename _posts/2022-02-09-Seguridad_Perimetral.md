---
layout: post
title: Seguridad Perimetral
subtitle: Concepto de seguridad perimetral y herramientas.
tags: [blueteam]
---

# 1. Antecedentes. Conceptos previos.

<br />

- **WAN**: Se trata de un acrónimo para Wide Area Network, se tratan de redes a gran escala que muchas veces abarcan países o continentes. Se suele utilizar para hacer referencia a una red externa en el contexto de dos redes, una interna y otra externa.

	<br />
	
- **LAN**: Se trata de un acrónimo para Local Area Network, se tratan redes pequeñas que abarcan una extensión física limitada y que generalmente son de tipo privada y estan gestionadas por puntos de acceso como Routers (ver tema 5 de Red Team).

	<br />

- **Firewall**: Un firewall o cortafuegos es un dispositivo (hardware o software) que tiene por funcionalidad el filtrar y bloquear tráfico de red para proporcionar mayor seguridad a una red privada de accesos o peticiones indeseables desde el exterior.

	<br />
	
- **Router**: Un Router o dispositivo de enrutamiento es un dispositivo que tiene  por objetivo **enrutar**, es decir, encaminar paquetes de red a su destino poniéndolos en dirección a el legítimo host dentro de su red privada o hacia otro router que se haga cargo del paquete.

	Los routers cuentan con dos IPs, una privada y otra pública.

	<br />
	
- **IDS**: Se trata de un acrónimo para Sistema de Detección de Intrusos; es un dispositivo que supervisa una red en busca de actividades maliciosas o infracciones de políticas. 

	Cualquier actividad de intrusión o violación generalmente se informa a un administrador o se recopila de forma centralizada mediante un sistema de gestión de eventos e información de seguridad.

	<br />
	
- **IPS**: Se trata de un IDS que tiene añadida la funcionalidad de automatizar la toma de medidas necesarias para bloquear cualquier actividad que se considere ilícita.

	<br />
	
- **DHCP**: Protocolo de asignación dinámica de IPs.

	<br />
	
	
# 2. Seguridad Perimetral. Concepto y herramientas.

El concepto de **seguridad perimetral informática** se refiere a la **seguridad** que afecta a la frontera de la red de nuestra empresa también llamada **perímetro**: el conjunto de máquinas y dispositivos que interactúan con el exterior, con otras redes.

Vamos a ver diversas herramientas que nos van a ayudar a proteger nuestro perímetro: Pfsense, IDSs (Snort), IPSs (Suricata o Banyard2).

<br />

# 3. PFSense.

## 3.1. Pfsense como concepto.

**Pfsense** es un software que combina la funcionalidad del router y del firewall construido a modo de sistema operativo. Es decir, es un sistema operativo con funcionalidades específicas de cortafuegos y router. 

Se trata herramienta de monitorización, bloqueo y redirección de tráfico que, además, ayuda a minimizar el número de exposiciones de elementos de nuestra red interna a la red externa (minimizar la superficie de ataque).

<br />

## 3.2. Instalación de pfsense.

Como ya hemos indicado, se trata de un sistema operativo que hay que instalar en alguna máquina. En este caso lo instalaremos en una máquina virtual.

Lo descargamos desde la página: https://www.pfsense.org/download/ y seleccionamos una arquitectura AMD64, con imágen ISO y un mirror en Alemania.

De todas formas hay que contrastar la velocidad de descarga de cada servidor.

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214101934.png' | relative_url }}" text-align="center"/>
</div>

Una vez hemos descargado la imágen correspondiente, la instalamos en Virtualbox:

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214104258.png' | relative_url }}" text-align="center"/>
</div>

Seguidamente, creamos dos RedNats y habilitamos dos interfaces de red para la pfsense:

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214162810.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214162844.png' | relative_url }}" text-align="center"/>
</div>

Cómo será esta nueva NatNetwork1 se deja para más adelante, de momento sólo habilitamos una segunda red.

Comenzamos la instalación:

- Aceptar:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214163256.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

- Instalar:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214105140.png' | relative_url }}" text-align="center"/>
	</div>
	
	</br>

- Elegimos "spanish es.acc.hdb keymap":
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214163508.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214163532.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- Comenzamos la instalación UFS Bios:
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214163731.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

- No queremos abrir una shell:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214163847.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214163936.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

- Una vez terminada, reiniciaremos la máquina y quitaremos la imágen ISO (medio de instalación) para que no nos vuelva a salir el wizard de instalación.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214164105.png' | relative_url }}" text-align="center"/>
	</div>

	Pinchamos sobre el disquete de la izquierda y abajo del todo seleccionamos la opción quitar disco. **Finalizamos dando a OK** no a la cruz arriba a la derecha.
	
	<br />

## 3.3. Configuración de pfsense.

Una vez hemos instalado el sistema operativo debemos configurar las funcionalidades de router y de firewall a través de una configuración local y remota (configuración con sitio web) respectivamente.

<br />

### 3.3.1. Antecedentes en Virtualbox. 

Por un lado, configuramos la recién creada NatNetwork1 y deshabilitamos el DHCP de la misma (**debemos observar bien que la dirección de red  de esta rednat1 es 10.0.3.x, no 10.0.2.x**, si el 3 no aparece en lugar del 2 habrá que cambiarlo manualmente)

El protocolo DHCP asigna dinámicamente IPs dentro de una red, pero esa es precisamente una tarea propia del router con lo que configuraremos pfsense para que desempeñe esa misma tarea y por eso deshabilitamos esta opción:

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214164632.png' | relative_url }}" text-align="center"/>
</div>

Además, configuramos la Kali en modo RedNat y le asignamos la NatNetwork1. Esto lo hacemos porque vamos a configurar pfsense desde un servidor web que sólo es accesible desde la LAN y por eso cambiamos la interfaz de red de la Kali temporalmente.

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214165113.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 3.3.2. Configuración local pfsense.

Una vez hemos instalado el software, comienza la configuración de la máquina. Volvemos a encender la máquina. Al encenderse y cargarse aparecerá un menú:

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214165440.png' | relative_url }}" text-align="center"/>
</div>

<br />

1. En primer lugar, elegimos la opción para asignar las interfaces de red.  
 
	Como queremos que tenga una funcionalidad de router necesitamos que tenga una IP para la red externa y otra IP para la red interna.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214165543.png' | relative_url }}" text-align="center"/>
	</div>
	
	No levantamos la VLAN. Asignamos a la WAN -> em0 y a la LAN -> em1.
		
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214165701.png' | relative_url }}" text-align="center"/>
	</div>

	Le damos a 'y'.
	
	<br />
	
2. Volverá a aparecernos el menú inicial. Pulsamos la opción 2 para configurar las direcciones IP.

	- En primer lugar configuramos la IP para la interfaz de red asignada a la WAN, que será configurada por el DHCP y dejaremos en blanco lo demás. De esta forma, el protocolo DHCP se encargará de asignar a pfsense una IP para la red WAN (que será la red 10.0.2.x de Virtualbox que sí tiene habilitado el DHCP).

		<div style="text-align:center">
		<img src="{{ 'assets/img/M10/Pasted image 20220214170708.png' | relative_url }}" text-align="center"/>
		</div>
		
		Y le indicamos que no a la última opción para que pfsense se expona a través de HTTPS y la comuniación siempre sea cifrada.
		
		<br />
		
	- Hacemos lo propio con la LAN:

		Esta red coincide con la NatNetwork1 que tiene deshabilitado el DHCP y por tanto le asignamos manualmente la IP estática 10.0.3.19, la máscara de red es 24, no asignamos dirección de gateaway ya que el propio pfsense actuará como router según hemos dicho.

		<div style="text-align:center">
		<img src="{{ 'assets/img/M10/Pasted image 20220214171227.png' | relative_url }}" text-align="center"/>
		</div>
		
		Habilitamos el protocolo DHCP para dotar de IPs a las máquinas dentro de la LAN y dotar a pfsense de funcionalidad router (recordamos que hemos deshabilitado el DHCP de la NatNetwork1) e introducimos el rango de direcciones IPv4 que irá desde 1 a 200 y finalizamos.
		
		<div style="text-align:center">
		<img src="{{ 'assets/img/M10/Pasted image 20220214171601.png' | relative_url }}" text-align="center"/>
		</div>
		
		<br />
		
Nos aparecerá el siguiente mensaje por pantalla:

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214171844.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 3.3.3. Configuración de pfsense a través del servidor web.

Una vez hemos accedido a la dirección indicada en el mensaje anterior desde nuestra Kali a través de la LAN obtenemos:

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214172111.png' | relative_url }}" text-align="center"/>
</div>

Donde las credenciales por defecto serán: **admin:pfsense**.

<br />

Una vez hemos accedido al sitio web con las credenciales anteriores veremos la siguiente ventana, le damos a Next:

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174023.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174046.png' | relative_url }}" text-align="center"/>
</div>

Hostname: pfSense
Domain: Localdomain

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174151.png' | relative_url }}" text-align="center"/>
</div>

Time Server: 2.pfsense.pool.ntp.org o el que esté disponible. Podemos verlo en pool.ntp.org para ver cuáles son de confianza.

Timezone: ETC/GMT+1 (En función de dónde estemos ubicados.)

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174346.png' | relative_url }}" text-align="center"/>
</div>

WAN Interface: DHCP

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174522.png' | relative_url }}" text-align="center"/>
</div>

LAN Interface: 10.0.3.19

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174545.png' | relative_url }}" text-align="center"/>
</div>

Password: admin (Credenciales: admin:admin) O LA QUE NOSOTROS CONSIDEREMOS CONVENIENTE.

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174621.png' | relative_url }}" text-align="center"/>
</div>

Le damos a Reload.

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174707.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174726.png' | relative_url }}" text-align="center"/>
</div>

Le damos a 'Check Update': 

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214174910.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214182802.png' | relative_url }}" text-align="center"/>
</div>


Una vez hemos terminado la configuración buscamos la posibilidad de hacer un backup de la máquina (snapshot, clonar, copiar el disco duro, etc.)

<br />

## 3.4. Reglas de Pfsense.

Hemos dicho que pfsense hace las veces de router y firewall. La funcionalidad de firewall de pfsense se configura desde el servidor web a través de la definición de reglas que permitirán o bloquearán tráfico de red desde la WAN a la LAN y viceversa. Vamos ahora a definir una serie de reglas para llevar a cabo una serie de redirecciones de tráfico y permitir el paso de un tipo de paquetes.

<br />

### 3.4.1. Política de bloqueo por defecto.

Vamos a deshabilitar la política de bloqueo de redes por defecto. Pfsense por defecto bloquea todo el tráfico desde WAN que provenga desde IPs privadas (192.168..., 10.0..., etc). Por lo que para funcionar en un entorno de red privada como el de virtualbox debemos deshabilitar esta opción (SI no, esto no aplica).

Para ello, debemos acceder al sitio web de configuración de pfsense con la Kali desde la red que este configurada como LAN en pfsense. Según la guía anterior, esta red es 10.0.3.x

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214190217.png' | relative_url }}" text-align="center"/>
</div>

<br />

Por tanto, configuramos nuestra Kali con la NatNetwork1 que esta configurada previamente con 10.0.3.0 y accedemos al servidor de web de pfsense a través de la LAN (IP 10.0.3.19). Una vez dentro, acudimos a la sección de reglas.

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214190433.png' | relative_url }}" text-align="center"/>
</div>

<br />

Acudimos a la primera opción, la que dice **block private networks** y le damos a Actions (Ruedecita).

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214190640.png' | relative_url }}" text-align="center"/>
</div>

<br />

Bajamos hasta abajo del todo hasta la sección **Reserved Networks** y quitamos el tick de las dos opciones que hay y guardamos: 

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214190759.png' | relative_url }}" text-align="center"/>
</div>


<br />

**aplicamos cambios**:

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214190930.png' | relative_url }}" text-align="center"/>
</div>

<br />

###  3.4.2. Permitir ICMP (ping).

Ahora vamos a crear una regla para permitir paquetes ICMP que por defecto se bloquean desde una WAN.

Para ello, vamos a crear una regla al principio de la lista

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214210939.png' | relative_url }}" text-align="center"/>
</div>

Y en ella establecemos las siguientes opciones:

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214211510.png' | relative_url }}" text-align="center"/>
</div>

Y a modo de descripición indicamos que se trata de una regla que permite el paso de paquetes ICMP por WAN. Habilitamos cualquier pero podríamos dejarlo solo en echo reply para habilitar sólo el *ping*.

FInalmente obtenemos: 

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214211616.png' | relative_url }}" text-align="center"/>
</div>

Aplicamos los cambios y guardamos. Esto en la práctica no nos conviene, es sólo a modo de ejemplo.

<br />

### 3.4.3. Redirección de puertos.

Vamos a generar reglas que redireccionan conexiones provenientes desde la WAN a una máquina que se encuentra dentro de la LAN, en este caso será la Metasploitable2. Por ello es **importante tener configurada la Metasploitable2 en modo NatNetwork1 (10.0.3.x)**.

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214215150.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/M10/Pasted image 20220214213631.png' | relative_url }}" text-align="center"/>
</div>

<br />



- **Todo lo que entre por WAN al puerto 22 se redirigirá a la Metasploitable puerto 22**.

	Conviene recordar que la Metasploitable será una máquina que estará dentro de la LAN, es decir, que pfsense se utiliza aquí a modo de **enrutador** que conecta dos redes: 10.0.2.x con 10.0.3.x según la guía que estamos siguiendo. 
	
	Por eso hemos permitido el desbloqueo de conexiones WAN y de trazas ICMP, de lo contrario esta regla no tendría sentido.
	
	Para ello esta vez nos dirigimos a FIrewall -> NAT.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214212330.png' | relative_url }}" text-align="center"/>
	</div>
	
	Y ajustamos las opciones correctamente.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214213229.png' | relative_url }}" text-align="center"/>
	</div>
	
	En este caso, optamos por tener sólo una máquina pero podemos añadir tantas como queramos, sólo hacen falta definir las reglas adecuadas para establecer una redirección de puertos satisfactoria.
	
	<br />
	
- **Todo lo que entre por el puerto 80 de WAN se redirigirá al puerto 80 de la Metasploitable2**:

	Hacemos de nuevo lo que hemos hecho con SSH pero ahora con el puerto 80.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214214302.png' | relative_url }}" text-align="center"/>
	</div>
	
	Guardamos y aplicamos los cambios.
	
	<br />
	
## 3.5. Comprobando las reglas.

Ahora que hemos ajustado las reglas a través del servidor web, apagamos la Kali y la cambiamos desde la NatNetwork1 (LAN) a NatNetwork (WAN) para que pueda realizar peticiones externas desde la WAN y podamos comprobar que nuestras regas están bien hechas y que pfsense funciona correctamente.

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214214550.png' | relative_url }}" text-align="center"/>
</div>

<br />

Además antes de proseguir comprobasmo cuál es la IP de pfsense para la red WAN: 

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214214741.png' | relative_url }}" text-align="center"/>
</div>

(Por una confusión tuve que reinstalar la máquina y se me asignó otra IP a la WAN en pfsense durante la instalación, ya que **antes era la 10.0.2.15 y ahora es la 10.0.2.7** pero esto no afecta en nada a la resolución del ejercicio, sólo es la IP de conexión a pfsense desde WAN y si no hubiera reinstalado la máquina seguiría teniendo la 10.0.2.15).

<br />

- **Ping a pfsense**: Vamos a ver qué puertos ofrece la máquina pfsense para comprobar que nuestra regla de habilitación de paquetes ICMP funciona. Si es así, los paquetes ICMP que el comando ping manda deberían llegar a los puertos 22 y 80 de pfsense y de ahí deberían de ser redirigidos a los respectivos puertos 22 y 80 de Metasploitable.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214221328.png' | relative_url }}" text-align="center"/>
	</div>
	
	Comprobamos que así es y además debemos de hacer una observación. Al igual que ocurría con la redirección de puertos SSH, al pedir la versión de los servicios que ofertan dichos puertos que llevan a cabo una redirección lo que se obtiene es información del puerto remoto. No debemos de hacer caso del puerto que escaneamos (cuyo número se elige arbitrariamente) sino de la información que recibimos en el campo de la **versión** que realmente es información proporcionada por el puerto remoto y no por el local; no por estar escaneando el puerto 80 estamos accediendo al servicio HTTP ya que podríamos haber configurado para redireccionar el puerto 9999 en pfsense. Lo que realmente debe orientarnos sobre el servicio que estamos visitando en ese puerto es la versión del mismo.
	
	<br />

- **Redirección SSH**: Vamos a conectarnos al puerto 22 de pfsense con el comando SSH y utilizamos el usuario *msfadmin* para ver si la redirección es satisfactoria. De serlo, se nos debe redirigir al puerto 22 de Metasploitable y como estamos accediendo con un usuario existente en la misma la operación debería de salir correctamente:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214214902.png' | relative_url }}" text-align="center"/>
	</div>
	
	Y efectivamente, es satisfactoria.
	
	<br />
	
- **Redirección HTTP**: Vamos a intentar acceder al puerto 80 de pfsense para ver si la redirección HTTP de pfsense a la Metasploitable2 es satisfactoria.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214215435.png' | relative_url }}" text-align="center"/>
	</div>
	
	Y de nuevo es satisfactoria ya que nos conectamos a la IP de pfsense pero nos aparece la web de la Metasploitable2.
	
	<br />
	
## 3.6. ¿Qué está ocurriendo?

Lo que hacemos es que el servidor donde hemos instalado el software de pfsense actúe a modo de enrutador entre dos redes. Entonces, las conexiones que llegan a ese enrutador se redirigen según las reglas establecidas con la ventaja de que toda la información pasa a través de pfsense permitiendo su monitorización.

<div style="text-align:center">
	<img src="{{ 'assets/img/M10/Pasted image 20220214215724.png' | relative_url }}" text-align="center"/>
</div>

De esta manera, configurando reglas ponemos en contacto todos los servicios que ofrecen nuestras máquinas de la LAN (http, ssh, ...) sin exponer a estas al exterior. 

Lo único que estaría expuesto sería pfsense que es una herramienta de monitorización y los puertos objeto de la redirección de pfsense de cada una de las máquinas de la LAN. 



