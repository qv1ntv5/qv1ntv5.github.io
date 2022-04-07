---
layout: post
title: Protocolos Relevantes
subtitle: Presentación de los protocolos de red más importantes.
tags: [redes] 
---

# 1. Introducción.
Los protocolos son un conjunto de reglas o pautas de actuación que permite a las máquinas y aplicaciones intercambiar información entre ellas. Tales protocolos deben de ser llevados a cabo por cualquier máquina que forme parte de la comunicación. on estos protocolos se consigue un intercambio ordenado de información, eficiente y fiable.

<br />

## 2. Protocolos de la capa de red.
### 2.1. IP.

### 2.1.1. Direcciones IP.

 **IP**: El 'Internet Protocol' es un protocolo, un conjunto de pautas de acción, para enrutar y direccionar paquetes de datos de forma que estos puedan atravesar la red y llegar al destino correcto.
 
 Los datos que atraviesan la red lo hacen partidos en trozos más pequeños llamados **paquetes**. La información IP se asocia a cada paquete mediante una cabecera y esta información ayuda a los routers a mandar paquetes hacia el sitio correcto.
 
 Todo dispositivo o dominio tiene asociado una dirección IP, de forma que cuando se quiere enviar un paquete hacia ellos lo que se hace es mandar un paquete a la dirección IP que el dispositivo tiene asociada y de esa forma, el paquete puede ser enviado a su destino.
 
 Así pues podemos establecer que:
 
 - La **dirección ip** es un identificador único asignado un dispositivo o a un dominio que se conecta con internet. Cada IP tiene una serie de caracteres  dividos en cuatro grupos (mirar [[1 IPs]]).
 
 - Vía DNS, que traduce los dominios a direcciones IP y viceversa, los usuarios son capaces de acceder al servicio generado por dicho dispositivo sin memorizar las complejas direcciones IP.

<br />

### 2.1.2. Paquetes IP.

Los paquetes que se envían mediante el protocolo IP reciben el nombre de **paquetes IP** están creados añadiendo una **cabecera IP** de cada paquete de datos antes de mandarlo. Esta cabecera no es más que un conjunto de bits que recoge algunas partes de la información del paquete:

- Tamaño de la cabecera.
- Tamaño del paquete.
- TTL (Time To Live, la cantidad de tiempo que un paquete puede vagar dentro de la red).
- Protocolo de transporte utilizado.  
- Direccion IP de llegada y envío.

Entre otros.

 <br />

### 2.1.3. Cómo funciona el routing.

Internet está hecho de grandes redes interconectadas que son cada una responsables de ciertos bloques de direcciones IP. Estas grandes redes se conocen como **sistemas autónomos (AS)** 

Una variedad de protocolos de routing (recordamos que el routing es el proceso de seleccionar un camino para el tráfico de red en una red), incluyendo BGP, se encargan de transportar los paquetes IP de AS a AS hasta que finalmente llegan a la AS que se hace responsable de dicha IP.

Los distintos protocolos que atraviesa un paquete van añadiendo al mismo distintas cabeceras con información sobre cómo debe de ser tratada la información que hay dentro. 

<br />

### 2.1.4. Qué es TCP/IP?.

El modelo TCP/IP es un modelo de red que debe su nombre a la combinación de los protocolos TCP e IP. 

El protocolo TCP o Protocolo de Control de Transmisión es un protocolo de transporte, lo que significa que dicta la forma en que se envían y reciben los paquetes.

Una cabecera TCP se adhiere a cada porción de datos que usan TCP. TCP asegura que todos los paquetes lleguen en orden una vez que comience la transmisión, a través de TCP, el destinatario reconocerá haber recibido los paquetes en dicho orden y en el caso de no haber recibo se volverán a envíar aquellos paquetes que no se hayan envíado correctamente.

TCP está diseñado para brindar confiabilidad, no velocidad, debido a que tcp debe de asegurarse de la llegada de todos los paquetes este puede ser un poco más lento. 

TCP e IP se diseñaron originalmente para usarse juntos y a menudo se les conoce como la siute TCP / IP. Sin embargo se pueden utilizar otros protocolos de transporte con IP, como UDP, pero esto, al mismo tiempo que lo hace más rápido, lo hace menos confiable.

<br />

## 2.2. ICMP.

### 2.2.1. Introducción.

El ICMP (Internet Control Messag Protocol) es un protocolo de la capa de red utilizado por dispositivos de red para encontrar incidencias en la comunicación de red. Fundamentalmente ICMP es utilizado para determinar si un paquete ha llegado a su destino en un determinado tiempo o no, se utiliza sobre todo en routers.

<br />

### 2.2.2. Utilidad.

El propósito principal de ICMP es el reporte de errores. Cuando dos dispositivos se conetan a internet, el ICMP existe para generar avisos de errores y reportarlos al dispositivo emisor en el caso de que algunos paquetes no lleguen a su destino. Por ejemplo, si un paquete de datos es demasiado grande para enviarlo, el enrutador lo descarta y genera un error ICMP que se envía al emisor del paquete descartado.

Un uso secundario del protocolo ICMP es realizar diagnósticos de red. Por ejemplo, los comandos de terminal **traceroute** o **ping** operan mediante ICMP:

- **traceroute**: Es utilizado para mapear las posibles rutas que un paquete puede seguir a lo largo de una red hasta llegar al lugar de destino.

- **Routing Path**: Llamamos como Ruta de enrutamiento o "routing path" al camino físico que conecta dos routers. El viaje de un paquete entre dos routers se denomina 'hop'. Traceroute también informa del tiempo que tarda cada camino.

- **ping**: Ping es una versión simplificada de 'traceroute', calcula la velocidad de conexión entre dos dispositivos pero no aporta información acerca del hop o los routers seguidos.

Esto es que los comandos anteriores mandan paquetes con una cabecera que les permite identificarse como paquetes ICMP.

Sin embargo, también existen ataques que explotan los procesos anteriores: 'ICMP flood attack' y 'ping of death'.

<br />

### 2.2.4. ICMP y cyberataques.

- **ICMP flood attack**: Este ataque consiste en saturar a un objetivo mediante request de mensajes ICMP, de esta forma consume sus recursos atendiendo peticiones fraudulentas y no puede utilizar otros servicios.

- **Ping de la muerte**: El ping de la muerte consiste en envíar a la dirección del objetivo un ping con un tamaño mayor del que la máquina objetivo puede manejar generando un crasheo de la misma. Este en principio se manda en forma de pequeños paquetes, pero cuando este es reconstruido en la maquina receptora genera un colapaso del búffer.

-  **Smurf attack**:  Este ataque consiste en coger la dirección IP de una víctima y utilizarla como si fuese la tuya propia, de esta forma, desde tu dirección IP falsificada mandas paquetes ICMP a otro dispositivo y este responde a la dirección IP falsa sobresaturándolo con paquetes ICMP.



<br />

## 2.3. Protocolo ARP. 

Para poder enviar paquetes de datos en redes TCP/IP, un servidor necesita, sobre todo, tres datos de dirección sobre el host al que se dirige, entre el nivel 2 y 3 del modelo OSI:

- Máscara de subred.
- La dirección IP. Que sirve para localizar a una máquina a través del ARP.
- La dirección MAC (también conocida como dirección hardware o dirección física) que sirve para identificar a dicha máquina.

El protocolo ARP es un protocolo que se encarga **de vincular una dirección MAC (o física) a una dirección IP**. Se encarga de permitir que un dispositivo conectado a una red pueda obtener una ruta MAC de otro equipo que esté conectado a esa misma red. Esencialmente, lleva a cabo en el proceso de mapeo una traducción para que los sistemas puedan reconocerse entre sí. Vamos a establecer los pasos a seguir.

Supongámos que tenemos una red local en la que hay varios ordenadores conectados. Supongámos que un ordenador A quiere mandar un mensaje al ordenador B y que ya sabe su dirección IP, pero le sigue faltando la dirección MAC (la máscara de subred se obvia puesto que los dos están en la misma red) Así:

1. En primer lugar, el ordenador 'A' buscará en una lista interna llamada ARP caché para comprobar si la dirección IP del host con el que desea ponerse en contacto ya tiene asociada una dirección MAC.
2. Si no lo está, envía un mensaje utilizando la función *broadcast* al resto de ordenadores de la red preguntando por la dirección MAC asociada a la IP que ya tiene. 
3. De forma que el ordenador propietario de dicha IP, el B, recibe el mensaje y le manda la dirección MAC y de esta forma ya puede haber comuncación entre los dos ordenadores.
4. Por último, esta dirección MAC se guarda en el ARP caché que está para hacer la comunicación más eficiente, de forma que la proxima vez que ambos ordenadores deseen ponerse en contacto no hará falta recurrir a la expedición. 

Cabe destacar que la dirección MAC así guardada en el ARP caché (comando arp -a) se guarda de forma dinámica y eventualmente se terminará borrando. Esto se hace para prevenir que el ARP caché se llene de direcciones que no se utilizan frecuentemente.

Por otro lado, si alguien intencionalmente escribe dicha dirección en el ARP caché se guardará de forma estática.

<br />

## 2.3. Protocolo PPP.

### 2.3.1.  Antecedentes.

Aproximación a la orientación de bytes. Se trata simplemente de entender el frame como una colección de bytes o caracteres. Entre los protocolos que están orientados a bytes nos encontramos a: BISYNC, PPP, DDCMP.

<br />

### 2.3.2. PPP.

- PPP es un protocolo de la capa de enlace de datos.
- PPP es un protocolo WAN (Wide Area Network), que es comúnmente utilizados entre links de internet, esto es, por ejemplo, entre dos routers.

Existen dos principales usos para el protocolo PPP:

- Es muy utilizado en comunicaciones de banda ancha.
- También es muy utilizado para transmitir datos multiprotocolos entre dos ordenadores contectados directamente (point-to-point devices, like routers).


<br />

### 2.3.3. Frame format.

En PPP nos encontramos la siguiente estructura:

| Flag | Addres | Control | Carga útil | Checksum | Flag |
| - | - | - | - | - | - | 

- Flag: Es un byte que marca el inicio y el final del frame. El patrón del flag suele ser: 01111110.
- Addres: Un byte que es 11111111 en el caso de ser un dato de broadcast.
- Control: Un byte constante de 11000000.
-  Protocol: 1 o 2 bytes que contienen información acerca del tipo de dato contenido en el apartado de carga útil.
-  Carga útil: Esta parte contiene los datos de la capa de red, tiene una capacidad máxima de 1500 bytes.
-  Cheksum: Son dos bytes que están oritentdados a la detección y reporte de errores.


Sin embargo existe un problema, y es que los bits que integran los flags de inicio y final también pueden aparecer dentro del apartado de la carga útil y recodermos que estos bits marcan el inicio y el final del frame. Por tanto, ¿como nos las apañamos para diferenciar el flag de un byte normal?
Para ello vamos a explicar lo que es el 'relleno de caracteres o bytes' ya que PPP es un protocolo orientado a bytes.

<br />

### 2.3.4.  Relleno de caracteres. 

Sabemos que la secuencia de los flags pueden aparecer en la parte de la carga útil y vamos a manejar esto metiendo o añadiendo un byte de más.

Es decir, el relleno de byte o relleno de caracteres es el proceso de añadir un byte o caracter de más siempre que la secuencia del 'flag' aparezca dentro de la carga útil para indicar de esta forma que no se trata de la secuencia que indica que es el final del 'frame'. 

<br />

# 3. Protocolos de la capa de transporte.

<br />

## 3.1. TCP.

TCP o (Protocolo de Control de transmisión) es un protocolo que permite a las aplicaciones comunicarse con garantias independientmente de ls capas inferiores del modelo de red.


Esto significa que los dispositivos tienen únicamente que enviar los "segmentos" (unidad de medida de TCP) sin preocuparse de si van a llegar los datos correctamente. TCP da sorporte a múltiples protocolos de la capa de aplicación: HTTP, POP3, SMTP, TLS, FTPES, SFTP, SSH etc.

<br />

### 3.1.2. Principales características.

TCP es un protocolo de la capa de transporte y tiene mecanismos que aseguran la integridad de los segmentos durante su envío sin la necesidad de recurrir a la capa de aplicación para solucionar errores. TCP inicia la transmisión de datos sin intervención de la capa aplicación.

- El MSS es el tamaño máximo en bytes que TCP puede recibir en un sólo segmento (similar al MTU, pero sólo en la capa de transporte), 1500 bytes ,menos la cabecera TCP y la cabecera IP:


	MSS = MTU - Cabecera TCP -Cabecera IP


- TCP tiene un mecanismo complejo de control de errores, entre los distintos métodos para detectar errores se encuentran:

	- Checksum (Bloque de datos encargado de la detección de errores).
	- Numeración de segmentos.
	- Confirmación ACK selectivas.
	- Temporizadores.
	- Descarte de segmentos duplicados.


	Si TCP detecta un error también tiene mecanismos de solvención de la incidencia para que la capa de aplicación no tenga que intervenir.


- Otra característica importante de TCP es que se encarga de que los datos lleguen en orden, en el mismo orden en el que fueron emitidos. El protocolo está diseñado para *el receptor esperé después de un segmento el siguiente y si este no llega, o llega otro en su lugar, genera un error y pedirá un retransmisión del segmento que falta antes de la llegada del siguiente.*

- Por otra parte, *con cada segmento recibido de forma satisfactorio se emite un ACK indicando al emisor de la correcta llegada de cada segmento, aunque en la práctica se envían confirmación del recibimiento de múltiples segmentos por una cuestión de saturación.** 


- El TCP también dispone de mecanismos dinámicos de control de flujo para evitar que ninguna de las dos partes se sature en el envío/recibimiento de datos. Este mecánismo, entendido de manera simplista, consiste en que *existe un buffer que el emisor puede llenar a conveniencia, cuando este buffer se llena, el receptor envía un ACK para indicar que de momento el emisor no puede enviar más hasta que se vuelva a enviar un ACK indicando que puede seguir envíando datos.*


- TCP también dispone mecanismos de control de congestion. Esto nos permite que no se pierdan paquetes por la red debido a la congestión de los routers por tráfico excesivo. *Si un router recibe paquetes a un ritmo mayor del que puede manejar comenzará a descartar paquetes y estos se perderán. Existen los conceptos de ventanas de congestión y de recepción que actúan de forma complementaria; cuando una crece la otra disminuye y viceversa controlando de forma dinámica la congestión de tráfico.*


- Por último, una característica interesante de TCP es la de multiplexación de datos, que nos permitirá recibir la información de varios jost simultáneamente.

<br />

### 3.1.3. Establecimiento de la conexión entre cliente y servidor y desconexión en TCP.

#### 3.1.3.1. Iniciando conexión.

La principal característica del protocolo TCP es que se trata de un protocolo orientado a conexión, para poder establecer una conexión entre cliente y servidor, es necesario establecer una conexión previa con dicho servidor.


Esta conexión previa se denomina como **3-way handshake** y consiste básicamente en que el cliente (que inica la conexión) envía un mesaje SYN al servidor (que recibe la conexión) y el servidor a su vez le enviará un SYS-ACK (un mensaje de confirmación de recibimiento del SYS) y el cliente responderá con ACK junto con el primer intercambio de datos.

<div style="text-align:center">
<img src="{{ 'assets/img/Redes/Pasted image 20211127190548.png' | relative_url }}" text-align="center"/>
</div>


*Un detalle importante de TCP es que genera números secuencia a cada lado ayudando a que no se puedan establecer conversaciones falsas entre el emisor y un falso receptor y viceversa, aunque este mecanismo no evita que la información pueda ser interceptada por ejemplo po un ataque MitM (Man in the Middle).*

<br />

#### 3.1.3.2. Ataques vía TCP.

Además, con TCP existe un ataque que consiste envíar desde múltiples host varias peticiones de conexión (varios mensajes SYS) con el objetivo de saturar de conexiones al receptor, algunas posibles soluciones a este ataque son:

- Limitar el número de conexiones.
- Aceptar conexiones solamente de IPs fiables.
- Retrasar la asignación de recursos con cookies.

<br />

#### 3.1.3.3. Finalizando la conexión.

Para finalizar la conexión el cliente envía un mensaje FIN y el servidor,  tras emitirlo responderá con un ACK y otro FIN a lo que el cliente responderá con un ACK al FIN del servidor y la conexión habrá terminado.

<div style="text-align:center">
<img src="{{ 'assets/img/Redes/Pasted image 20211127191440.png' | relative_url }}" text-align="center"/>
</div>


Si el proceso de cierre "falla", entonces nos podemos quedar con una conexión medio abierta en la que una de las partes, aquella que no ha finalizado la conexión, puede seguir mandando mensajes pero la otra no.


<br />

### 3.1.4. Cabecera de TCP.

TCP añade 20 bytes de cabecera como mínimo en cada segmento (puesto que hay un campo de "opcionales"). En esta cabecera TCP entontraremos información relativa  a la comunicación como:

- Puerto de origen.
- Puerto de destino.
- Número secuencia.
- Número ACK.
- Distintos flags de TCP como SYN, ACK, RST, ...
- Campo de la ventana de recepción.

De esta forma, el receptor de la información, recibe el segmento, recoge su cabecera, procesa la información y con ella emite un paquete que contiene la información necesaria para ser envíado a su destino satisfactoriamente.

Conviene mencionar que los puertos reservados para los 'socket' van desde el 0 hasta el 65535. Pero existen distintos tipos de puertos:

- Puertos conocidos (0 al 1023) reservados por la IANA.
- Puertos registrados (1024 al 49151) reservados por aplicaciones.
- Puertos privados (49152 al 65535) Puertos no reservados que pueden utilizarse libremente.

<br />

## 3.2. UDP.

### 3.2.1. Principales características.

La principal característica de UDP es que, aunque también es un protocolo situado en la capa de transporte, no es un protocolo orientado a conexiones y por tanto no dispone de ningún mecanismo que garantize la integridad de los datos durante su envío.

- No inicia una conexión previa al intercambio de datagramas <8indicamos datagrama puesto que segmento es propio de TCP).
- No contiene un mecanismo de control de flujo.
- No contiene un mecanismo de control de congestión.
- No garantiza la llegada en orden de los segmentos.
- No existe un procedimiento de control acerca de si un datagrama ha o no ha llegado satisfactoriamente a su destino.

Por tanto, es mucho más rápido pero mucho menos confiable.

<br />

### 3.2.2. Cabecera UDP.

UDP añade a todos los datagramas una cabecera de 8 bytes que contiene:

- Puerto origen.
- Puerto de destino.
- Longitud del datagrama.
- Checksum.

<br />

# 4. Protocolos de la capa de aplicación.

## 4.1. Protocolo HTTP/S.

### 4.1.1. Intro.

Ahora vamos a explorar el protocolo HTTP, HTTPS es HTTP cifrado, asi que nos centraremos en HTTP.

<br />

### 4.1.2. Generalidades.

HTTP es el acrónimo en inglés de "HyperText Transfer Protocol", se trata de un protocolo que nos permite realizar peticiones de datos y recursos, como documentos HTML.

Se trata de un protocolo básico en cualquier intercambio de datos en la web que ofrece estructura de cliente-servidor. Esto significa que en este protocolo existe una máquina o un dispositivo que inicia una petición de datos, normalmente un navegador web, que intercambia datos con un servidor.

Así, por ejemplo, una página web resulta de la unión de muchos subdocumentos obtenidos a través de distintas peticiones a través del protocolo http a distintos servidores.

<div style="text-align:center">
<img src="{{ 'assets/img/Redes/Pasted image 20211216171122.png' | relative_url }}" text-align="center"/>
</div>


Estos clientes y servidores se comunican mediante mensajes individuales ( en lugar de utilizar un flujo continuo de datos):

- Los mensajes del "cliente" se denominan **peticiones**.
- Los mensajes del "servidor" se denominan **respuestas**.

Es conveniente mencionar el hecho de que se trata de un protocolo flexible, ampliable que ha ido evolucionando con el tiempo. Se transmite sobretodo sobre el protocolo TCP o el protocolo cifrado TLS (una variante mejorada del ahora obsoleto SSL), aunque puede emplearse sobre cualquier protocolo fiable.

<br />

## 4.2.  SMTP.

SMTP es un acrónimo que atiende a Simple Mail Transfer Protocol. Se trata de un protocolo de comunicaciones estándard para transmisión de mails electrónicos.

<br />

### 4.2.1. Generalidades.

Mientras que SMTP es un "push protocol" (que envía datos), POP e IMAP son "pull protocols". Es decir SMTP se utiliza para enviar mails mientras que POP e IMAP se utilizan para recogerlos.

SMTP es un protocolo de la capa aplicación basado en el protocolo de transporte TCP que escucha en el puerto 25. 

SMTP corre sobre el servicio postfix, hay que tener el servicio de postfix instalado y corriendo para poder mandar mails.

<br />

## 4.3. POP3.

Mientras que SMTP envía mails, POP3 es un "pull protocol" que se encarga de recogerlos. Corre bajo el servicio dovecot-pop3d, es decir, hay que tener instalado y corriendo este servicio. Esto habilitará que una máquina externa se conecte al puerto 110 para enviar mails que nosotros recogeremos con pop3. Recordamos que pop3 está sin cifrar.

<br />

## 4.4. FTP.

FTP es un acrónimo que responde a File Transfer Protocol que actúa con TCP/IP y se utiliza para transmitir ficheros desde un host al otro. Generalmente utiliza el puerto 21. 

FTP corre desde el servicio "vsftpd". Hay que instalar y correr este servicio en linux para poder hacer una transferencia de archivos a otra máquina.


## 4.5. DHCP.
- **DHCP**: Dynamic Host Configuration Protocol. Se trata de un protocolo que asigna dinámicamente direcciones IP dentro de una red.