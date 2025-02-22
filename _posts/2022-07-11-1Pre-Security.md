---
layout: post
title: 1.Pre-Security path.
subtitle: Notes of the TryHackMe's path Pre-Security.
tags: [thm]
---

## 0. Índice.

- 1 Network Fundamentals.
	- 1.1. Network. Internet. Dirección IP. MAC. Ping.
	- 1.2. Intro to LAN.
	- 1.3. Understanding OSI model.
	- 1.4. Packets & Frames.
	- 1.5. Extending your Network.
- 2 Web Fundamentals.
	- 2.1. DNS in detail.
	- 2.2. HTTP in detail.
	- 2.3. How Websites works.
	- 2.4. Other componentes. Web servers.
- 3 Linux Fundamentals.
	- 3.1. Linux Fundamentals I.
	- 3.2. Linux Fundamentals II.
	- 3.3. Linux Fundamentals III.
- 4 Windows Fundamentals.
	- 4.1. Windows Fundamentals I.
	- 4.2. Windows Fundamentals II.
	- 4.3. Windows Fundamentals III.

<br />

### 1. Network Fundamentals.

#### 1.1. Network. Internet. Dirección IP. MAC. Ping.

<br />

**Network. Internet.**

Vamos a comenzar definiendo que una Red es un conjunto de dispositivos conrectados que comparten información. En concreto, Internet es una red gigante de la que participan la amplia mayoría de los dispositivos existentes con capacidad para conectarse a otro dispositivo.

Internet a su vez está constituida por redes más pequeñas conectadas entre sí.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704171831.png' | relative_url }}" text-align="center"/>
</div>

Estas redes más pequeñas se denominan como "redes privadas".

<br />

**Direcciones IP. Direcciones MAC.**

Para que dos dispositivos puedan comunicarse deben de tener la capacidad de identificar y ser identificados por otro dispositivo. Para ello se emplean dos entidades fundamentales:

- Dirección IP.
- Dirección MAC.

La dirección IP (no confundir con el protocolo IP) es un número que sirve para identificar un host (ordenador) dentro de una red. Más concretamente es un conjunto de números separados en cuatro octetos (debido a que en esencia los ordenadores procesan cada número en binario y cada uno de los cuatro números que componen la IP consta de 8 bits):

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704173123.png' | relative_url }}" text-align="center"/>
</div>

En la imagen anterior lo que tenemos es una dirección IP expresada de forma decimal. Es importante saber que: **no pueden haber dos IPs iguales en la misma red**. A su vez, diferenciamos entre IPs públicas que no pueden estar repetidas en ningún caso, e IPs privadas que pueden estar repetidas en redes privadas distintas. Generalmente, los dispositivos que poseen una IP privada emplean un dispositivo con IP pública para comunicarse con otro dispositivo a través de una red pública como Internet.

Por otra parte, la dirección MAC es una dirección asociada a un chip interno que actúa como una interfaz de red física (se trata de una dirección física del dispositivo en sí) y que supuestamente actúa como un identificador único de la máquina. Mientras que las IPs pueden variar en el tiempo, las direcciones MAC no lo harán nunca. Sin embargo, existen procesos de "MAC spoofing" con los que se puede suplantar la identidad MAC de un dispositivo.

En esencia la dirección MAC es una dirección de 6 números hexadecimales. Los tres primeros identifican al vendedor del dispositivo y el segundo identifica a la interfaz de red del dispositivo:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704175629.png' | relative_url }}" text-align="center"/>
</div>

Cabe decir que no todos los protocolos de comunicación emplean la MAC.

<br />

**Ping**

Ping es una herramienta fundamental en el campo del Networking. Este emplea el ICMP (Internet Control Message Protocol) para determinar el funcionamiento de una red. **El tiempo que tardan los paquetes ICMP en llegar al host de destino es medido por la herramienta y mostrado al usuario por pantalla.**

Esta herramienta viene con frecuencia integrada en dispositivos en forma de comando del propio OS del dispositivo.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704180818.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 1.2. LAN. Topologías. Swtichs. Routers. Subnetting. ARP. DHCP.
Recordamos que una red es un conjunto de dispositivos conectados y la topología de red consiste en el diseño de esas conexiones. A lo largo de la historia se han implementado diversos diseños de topología:

<br />

**Topologías**:

- *Topología de estrella*:

Los dispositivos de la red están individualmente conectados a un **switch**, es una de las más comúnes hoy en día:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704210442.png' | relative_url }}" text-align="center"/>
</div>

Este esquema funciona conectando cada host a un dispositivo llamado switch que pone a todos ellos en contacto. Es una buena elección para manejar redes pequeñas pero la cosa cambia cuando la network crece.

<br />

- *Topología de Bus*:

En esta topología todos los ordenadores están conectados a un cable principal denominado "espina dorsal" (backbone) por el que circula la información:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704212209.png' | relative_url }}" text-align="center"/>
</div>

El problema de este tipo topología es que se satura con facilidad. No puede manejar un exceso de información.

<br />

- *Topología de anillo*:

En este caso, los ordenadores también están conectados entre sí formando un anillo:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704212537.png' | relative_url }}" text-align="center"/>
</div>

Si observamos que si algunos de los dispositivos quedase incapacitado o el cable sufriese algún tipo de daño la comunicación se vería cortada sin remedio.

<br />

**Switch. Router.**

Como ya hemos indicado, la topología más importante por ser la más frecuente consisten en aquella en la que todos los dispositivos están conectados entre sí mediante los **swtich**. Este es un dispositivo encargado de dos funciones fundamentales:

- Agregar más dispositivos a la red.
- Llevar la información que le transmiten los dispositivos conectados al equipo de destino que generalmente se trata de un router.

Por otra parte, un **router** es un dispositivo que como su propio nombre indica, enruta la información hacia otra network.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220704214854.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, el esquema general es que en una red de dispositivos, uno emite información contra el switch que es reenviada al router que la dirige sobre la network donde se encuentra el host de destino.

<br />

**Subnetting. Mascaras de red. Gateaway.**

Subnetting es el término que hace referencia a la división de una network en redes de menor tamaño:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220705141115.png' | relative_url }}" text-align="center"/>
</div>

En este caso el ejemplo expone redes, grupos de ordenadores que comparten una categoría concreta. Para diferenciar entre cada pequeña subred se emplean las máscaras de red.

Este no es más que una pequeña parte de la IP que se queda fija para cada dispositivo de la subred. Esto permite identificar tres datos fundamentales:

- Identificar la dirección de red.
- Identificar la dirección del host.
- Identificación del Gateaway.

El Gateaway es aquel dispositivo dentro de una red que se pone en contacto con otra red. Frecuentemente suele ser el router.

Así por ejemplo, una mascara de red que fija los 3 primeros octetos de una red cuya dirección es: 192.168.1.0 tendrá 254 direccione disponibles para asociar dispositivos. Así, por ejemplo, para el dispositivo 192.168.1.100 se sabe que, teniendo en cuenta la máscara de red:

- 192.168.1.0 --> Network Address.
- 192.168.1.100 --> Host Address.
- 192.168.1.254 --> Default Gateway.

<br />

**Protocolo ARP, DHCP**

El término **ARP** responde sobre: Address Resolution Protocol y asocia una dirección MAC con una dirección IP dentro de una red. Se emplea para poner en contacto dispositivos dentro de una misma red.

Funciona enviando un mensaje de difusión (broadcast) a todos los ordenadores de una red buscándo aquel dispositivo que tenga una IP concreta (ARP request), el dispositivo que posee dicha IP se identifica aportando al emisor del mensaje original su MAC (ARP replay) y este último guarda la misma en el **ARP cache** para no tener que repetir el proceso cada vez que quiera ponérse en contacto con dicho dispositivo.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220705164629.png' | relative_url }}" text-align="center"/>
</div>

<br />

Por otro lado, el protocolo **DHCP** responde a Dynamic Host Configuration Protocol y se trata de un protocolo que asigna a un dispositivo nuevo en una red una dirección IP. El proceso consiste en que el dispositivo que no tiene todavía una dirección IP se pone en contacto con el servidor que gestiona dicho proceso a menudo referido como DHCP Server (DHCP Discover), este le proporciona una IP (DHCP Offer), el dispositivo la confirma (DHCP Request) y por último el servidor le manda una confirmación de su confirmación (DHCP ACK).

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220705170235.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 1.3. OSI model.

**Introducción al modelo OSI**

El modelo OSI (Open System Interconnection Model) es modelo fundamental empleado en la conexión entre dispositivos. Este ofrece un framework consistente en un conjunto de reglas que dictan cómo se emiten, reciben e interpretan los datos que dos dispositivos comparten. Este modelo está construido para permitir que los dispositivos que participen de él desempeñen cualquier tipo de funcionalidad.

Se compone de 7 campos de abstracción, llamados *capas*, cada uno con un conjunto de responsabilidades distinto en las que operan una serie de protocolos (reglas de acción que manipulan los datos).

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220705220858.png' | relative_url }}" text-align="center"/>
</div>

**Los datos viajan a través de cada capa y en el proceso se le van añadiendo fragmentos de información relativos a cada campo**(encapsulamiento de datos).

<br />

**Layer 7: Application**

La capa de aplicación es la capa que en la operan aquellos protocolos que deciden cómo el usuario interactúa con los datos que recibe.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220705222016.png' | relative_url }}" text-align="center"/>
</div>

En muchas ocasiones, estos datos son transferidos a la interfaz **GUI** (Graphical User Interface) para que el usuario pueda interactuar con ellos de una manera más comoda.

<br />

**Layer 6: Presentation**

En la capa de presentación entran en juego aquellos protocolos que estandarizan la información en función del que sea esta. Por ejemplo, muchos desarrolladores pueden elaborar software que envíe datos de formas múltiples y arbitrarias, sin embargo, este último se debe de manejar de la misma forma en todas las máquinas y por ello debe ser estandarizada. Actúa como una especie de capa traductora de datos.

<br />

**Layer 5: Session**

Llamamos **sesión** a una conexión satisfactoriamente establecida entre dos dispositivos. Una vez que los datos han sido correctamente traducidos desde la capa de presentación, pasan a la capa de sesión. En ella se encuentran protocolos que llevan a cabo un manejo de la sesión que conecta los dos dispositivos que se están pasando los datos.

Concretamente, la capa de sesión sincroniza las dos máquinas para asegurarse de que ambas máquinas esperan enviar y recibir los mismos datos e impone medidas de seguridad que pretenden garantizar la integridad de la sesión. Seguidamente, se procede a dividir estos en fragmentos de datos (paquetes) que serán posteriormente enviados de forma individual uno cada vez. Esto se hace principalmente para prevenir pérdida de información en caso de que falle la comunicación entre ambas máquinas.

Sin embargo, lo más destacable es que la sessión permite una cobertura de seguridad ya que los datos viajan a través de una misma sesión y se prevenie el cruce de datos entre sesiones.

<br />

**Layer 4: Transport**

La capa de transporte del modelo OSI juega un papel importante en la transmisión de datos a lo largo de la red. Empezaremos aclarando que cuando dos ordenadores se ponen en contacto y comienzan a intercambiar paquetes emplean uno de los siguientes protocolos de transporte:

- TCP
- UDP

El protocolo **TCP** que responde a *Transmision Control Protocol* esta diseñado para transmitir datos entre dispositivos de manera fiable y garantizada. Existen dos características distintivas del protocolo TCP:

- En primer lugar se formaliza una conexión entre ambos dispositivos (Three-way handshaking) que se abre antes de comenzar el intercambio de datos y se cierra al terminalo. Esta sesión TCP implementa varios mecanismos de seguridad a través de los cuales se garantiza de que la información llegue completa y ordenada.

- Además incorpora un mecanismo de checkeo de errores que garantiza que los paquetes fragmentados en la capa de sesión sean reensamblados de forma correcta.

De esta forma y en resumidas cuentas se puede afirmar que es seguro y fiable pero el conjunto de mecanismos que implementa para garantizar las características anteriores lo vuelven más lento de lo que inicialmente sería se enviará la información sin más además de que se puede volver aún más lento si la conexión entre ambos dispositivos no es la adecuada.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220705232658.png' | relative_url }}" text-align="center"/>
</div>

<br />

Por otra parte, **UDP** que atiende a *User Datagram Protocol* se emplea para la misma función que el protocolo anterior pero no posee sus características ni sus mecanismos de seguridad. Por lo tanto, es más rápido pero la información que se reparte no tiene las mismas garantias.

<br />

Ninguno de los dos es mejor que el otro. Sirven para distintos propósitos en función de la importancia que se le de al estado de la información recibida.

Por lo general, TCP es preferible cuando se traspasan datos que resultan en una gran cantidad de paquetes, tales como emails, fotos, etc. UDP tiene el propósito contrario, cuando los datos que se mandan ocupan muy pocos paquetes, por ejemplo, protocolos de sincronización temporal o de descubrimiento de dispositivos.

<br />

**Layer 3: Network**

En la capa de red es la capa donde sucede el enrutamiento y el reensamblado de paquetes a la forma original de los datos sucede.

Lo que tiene lugar en primera instancia es el routing, este determina la ruta por la que enviar a estos paquetes de datos. Entre los protocolos que se encargan de determinar la ruta correcta encontramos el OSPF (Open Shortest Path First) y el RIP (Routing Information Protocol), estos llevan a cabo su tarea mediante el sopesamiento de las siguientes cuestiones:

- La ruta más corta.
- La ruta más fiable.
- La ruta con la conexión física más rápida. (Conexión por cable, fibra óptica, etc.)

En esta capa, el componente más importante sobre el que se realiza todos los cálculos es la dirección IP que se adhiere en el paquete.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706101724.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Layer 2: Data link**

La capa de enlace de datos se concentra en la parte de la transmisión que tiene que ver con la dirección física. Recibe los paquetes de la capa de red con la dirección IP incluida y seguidamente y añade la dirección MAC del dispositivo de destino.

Dentro de cada dispositivo habilitado para conectarse a una red contiene una Network Interface Card que contiene una dirección MAC única.

<br />

**Layer 1: Physical**

En la capa física se referencian los componentes físicos por los que los datos pasan a la hora de cruzar la red. Cable ethernet, señales electricas, etc.

<br />

Así, a modo de resumen, tenemos la siguiente tabla:


|nº|nombre|función|
|---|---|---|
|7| Application | Cómo el usuario interactúa con los datos, a menudo GUI. |
|6| Presentation | Estandarización de los datos provenientes de la capa anterior. |
|5| Session | Mecanismos de seguridad que garantizan una buena conexión entre ambos dispositivos. |
|4| Transport | Cómo se envían los datos (TCP, UDP).|
|3| Network | Capa de transporte define las direcciones IP y la ruta óptima de circulación de los datos. |
|2| Data Link | Transferencia fiable de datos (ARP). |
|1| Physical | Referencia los medios físicos por los que se van a transportar los datos. |

De esta forma, si nosotros envíasemos un email, los datos que nosotros mandamos recorren las capas de abstracción del modelo OSI de mayor a menor (7 --> 1) sufriendo el proceso de encapsulación en cada capa hasta que los paquetes se envían y vuelven a recorrer dichas capas de abstracción de menor a mayor (1 --> 7) donde a medida que suben se "desencapsulan" y procesan cuando son recibidos por el receptor de los datos.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706105841.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 1.4. Packets & Frames.

**Definición y diferencia**

Tanto los paquetes como los marcos (frames) son pequeñas porciones de datos que al ensamblarse de manera ordenada forman una pieza mayor que puede ser interpretada por una máquina.

Sin embargo, es importante tener en cuenta la siguiente diferencia:

- El término **paquete** refiere a la estructura que se forma en la capa 3, o de *Network*, y que contiene las direcciones IP.

- Por otro lado, un **frame** es **aquello que se aporta en la capa inmediatamente posterior, de la capa 2, o de *Data Link*** del modelo OSI. Más concretamente, un frame es una unidad de transmisión de datos digitales, actúa como un contenedor de un único paquete de red por encapsulamiento.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706124006.png' | relative_url }}" text-align="center"/>
</div>

Llegados a este punto, es seguro asumir que cuando hablamos de una dirección IP, estamos hablando de paquetes. Cuando se elimina la información de encapsulación, estamos hablando del marco en sí.

<br />

**TCP/IP (Three-Way Handshake)**

TCP/IP es otro modelo que representa la superación del modelo OSI en cuanto a la relación de dispositivos en la red se refiere. Como cualquier otro modelo, se trata de un conjunto de protocolos agrupados en capas de abstración que permiten la comunicación entre los ordenadores pertenecientes a una red.

Las capas del modelo TCP/IP es:

- Application
- Transport
- Internet
- Network Interface

De la misma forma que en el modelo OSI, los datos atraviesan las distintas capas de abstracción recopilando cabeceras y encapsulando datos hasta conformar el paquete.

Una de las características del modelo TCP/IP es que emplea el protocolo TCP que es un *connection-based protocol* que significa que este protocolo se asegura de abrir una sesión TCP entre ambos dispositivos antes de comenzar a enviar datos a modo de mecanismo de seguridad y garantía. Este proceso es conocido como Three-Way Handshaking.

Vamos a ver algunas de las cabeceras más importantes que el paquete recopila en este modelo:

|Nombre|Descripción|
|-|-|
|Source Port|Contiene el número del pueto desde el que se ha enviado los datos. Se elige de forma aleatoria entre el 0 y el 65535.|
|Destination Port|Contiene el número del puerto al que se dirigen los datos en el otro dispositivo. El número dependerá del servicio al que se esté intentando acceder, por ejemplo, un servicio web correría en el puerto 80, el ssh en el 22, etc.|
|Source IP|Contiene la dirección IP del dispositivo emisor.|
|Destination IP| Contiene la dirección IP del dispositivo de destino.|
|Sequence Number| El protocolo TCP enumera los paquetes de datos transmitidos para asegurar un orden.|
|Acknowledgement Number| Para la siguiente pieza de datos, el número que lleva es el número "sequence number + 1".|
|Checksum|Se trata de una cabecera que lleva a cabo una correción de errores. Esto es la herramienta mediante la cual TCP garantiza una integridad.|
|Data|Los datos en sí, contenido HTTP, HTML, etc.|
|Flag|Esta cabecera determina cómo se debería de manejar el paquete.|

Ahora que conocemos las cabeceras más importantes, veámos cómo se produce una comunicación en el modelo TCP/IP. En este modelo que emplea el protocolo TCP (que recordemos es un *Connection-based protocol*) se emplea un Three-way handshaking consistente en una apertura formal en el que un dispositivo le manda un SYN (synchronise) a otro con la finalidad de abrir un canal de comunicación. Si el otro dispositivo está condiciones responderá con un SYN/ACK (SYNchronise ACKnowledge) y por último el emisor inicial devolverá un último ACK:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706171753.png' | relative_url }}" text-align="center"/>
</div>

Para cerrar una conexión el emisor mandar un FIN, el receptor devuelve un ACK en señal de que ha recibido el mensaje y seguidamente manda otro FIN y por último el emisor emite un último ACK que devuelve la conexión:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706171920.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, se tiene que una comunicación TCP hay generalmetne la siguiente estructura:

|Paso|Mensaje|Descripción|
|-|-|-|
|1|SYN|El primer mensaje que el cliente envía con la finalidad de abrir comunicación entre ambos dispositivos.|
|2|SYN/ACK|Este paquete lo manda el servidor para aceptar la comunicaicón con el cliente.|
|3|ACK|El cliente devuelve un último ACK para dar a entender al servidor que ha recibido su mensaje de confirmación.|
|4|Data|Una vez la conexión a tenido lugar se procede al envío de datos. Cada paquete mandado está serializado en una secuencia y tiene su propio ACK también respectivamente serializado.|
|5|FIN|Este paquete se emplea por el cliente para indicar que desea finalizar la conexión. El servidor responde con su respectivo ACK y otro FIN.|
|#|RST|Si existe algún problema durante la comunicación este paquete indica la finalización de toda comunicación.|

<br />

**UDP/IP**

De nuevo, UDP/IP es otro modelo que contiene como protocolos principales UDP e IP. Recordamos que UDP es **User Datagram Protocol** se otro protocolo que emplea para comunicar datos entre dispositivos que se centra más en la rapidez y menos en la calidad de la información. Se emplea en servicios en los que se puede tolerar una pérdida parcial de la información.

Las cabeceras más importantes que acompañan al protocolo UDP son, entre otras:

|Nombre|Descripción|
|-|-|
|Time To Live (TTL)|Este campo define un tiempo límite para el paquete de forma que se evita que la red cuelgue si no alcanza su destino.|
|Source Address| Dirección IP del emisor.|
|Destination Address|La IP del dispositivo de destino.|
|Source Port|Puerto desde el que se ha emitido el paquete. Recordamos que este es un número aleatorio desde el 0 al 65535.|
|Destination Port|Este es el puerto al que va dirigido el paquete. Dependerá del servicio al que esté dirigido.|
|Data|El contenido a transmitir en sí.|

A modo de diferencia, el diagrama que representa una comunicación UDP entre dos dispositivos sería el siguiente:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706180836.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Puertos comunes**

Existen fundamentalmente dos tipos de puertos, comunes o permanentes y temporales. Los primeros abarcan desde el 0 --> 1024 y corresponden a servicios conocidos que operan constantemente hasta que se le indican lo contrario y cada servicio tiene asociado un puerto específico:

|Protocol|Nº Puerto|Descripción|
|-|-|-|
|File Transfer Protocol|21|Se trata de un protocolo empleado por aplicaciones que comparten ficheros en un modelo cliente-servidor.|
|Secure Shell|22|Es un protocolo cryptográfico que entre otras aplicaciones se encuentra la *administración remota* entre otras. La comunicación está cifrada.|
|HyperText Transfer Protocol|80|Este es un protocolo fundamental en el uso de la exploración web.|
|HTTPS|443|Este protocolo es la variante HTTP cifrado con SSH.|
|Server Message Block|445|Este protocolo es similar a FTP pero exclusivo en Windows.|
|Remote Desktop Protocol|3389|Este protocolo se emplea por aplicaciones que comparten el escritorio de forma visual. Sería un homólogo de SSH pero sin las limitaciones de la terminal.|

<br />

#### 1.5. Extending your Network.

**Introduction to Port Forwarding**

La **redirección de puertos** o Port Forwarding es un componente esencial en la conexión de aplicaciones y servicios en Internet que permite extender las funcionalidades de una aplicación más allá de los confines de una red.

Concretamente, la redirección de puertos permite la redirección de comunicación de una combinación direcciónIP/puerto a otra combinación direcciónIP/puerto.

Por ejemplo, supongámos una red internat (Intranet) como la siguiente:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706194154.png' | relative_url }}" text-align="center"/>
</div>

Sin la redirección de puertos, todos los dispositivos quedarían restringidos a las funcionalidades presentes en los dispositivos de dicha red. Sin embargo, con el port forwarding se puede instaurar en el router un servicio que este coge de un dispositivo de una red externa. Más concretamente, cualquier conexión dirigida sobre el puerto 80 del router es redirigida sobre el mismo puerto en otro dispositivo accediendo a su contenido:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706194601.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Firewall**

Un **firewall** o cortafuegos es un dispositivo dentro de una red responsable de determinar qué tráfico está permitido y cuál no. Está configurado por un administrador y los criterios empleados para determinar si el tráfico es o no bloqueado son múltiples:

- Origen.
- Destino.
- Puerto de destino.
- Protocolo empleado.

El firewall lleva a cabo una inspección para resolver estas incógnitas y en base a la aplicación de una serie de reglas impuestas por un administrador decide a qué deja el paso.

Un firewall puede tener dos actitudes:

- *Statefull*: En la que emplea la totalidad de la información que proporciona una conexión para decidir si permite el tráfico.
- *Stateless*: En la que analiza cada paquete individualmente para decidir si lo deja pasar o no basado en una serie de reglas.

Observemos que los firewall regulan el tráfico y lo redireccionan, con lo que actúan sobre la capa 3 (Network) y la capa 2 (Data Link) del modelo OSI.

<br />

**VPN basics**

VPN es el acrónimo de Virtual Private Network y se trata de una tecnología que permite a dispositivos en redes separadas comunicarse de forma segura creando rutas entre ellas a lo largo de Internet más conocido como *túneles*. **Es precisamente a través de estos túneles mediante los que se forma una red privada de forma virtual**.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706211150.png' | relative_url }}" text-align="center"/>
</div>

Así, los dispositivos conectados en la 3ª Network siguen formando parte de la 1ª y la 2ª respectivamente, sin embargo, al mismo tiempo ambos conforman un 3ª de forma virtual.

Veámos algunos de los beneficios ofertadas por una VPN.

- Permite una conexión de redes de distintas localizaciones geográficas en una red privada.

- Ofrece privacidad. (Cifrado de datos).

- Ofrece discreción. La VPN oculta la IP natural cambiandola por una artifical dentro de la red privada.

Entre las tecnologías VPN más utilizadas se encuentran PPP y PPTP que actúan conjuntamente, concretamente, PPTP (Point-To-Point Protocol). También tenemos IPSec.

<br />

**Routers y Switches en profundidad. VLANs.**

Como ya hemos comentado antes, el trabajo de un router es el enrutar (routing) el tráfico en la dirección correcta mediante una serie de criteriors basados en información.

A su vez, el routing o enrutar involucra el crear una ruta entre redes de forma que los datos puedan llegar de forma rápida y segura a su destino. Operan sobre la capa 3 del modelo OSI. Sobre todo son útiles cuando ambos dispositivos están conectados a lo largo de muchas rutas:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706214919.png' | relative_url }}" text-align="center"/>
</div>

En la imagen anterior podemos ver dos dispositivos conectados mediante diversas rutas. Qué ruta seguirán los datos es algo que el primer router decidirá en base a las siguientes cuestiones:

- Qué camino es más corto.
- Qué camino es más seguro.
- Qué camino tiene el medio físico (cable, fibra óptica, etc) más rápido.

<br />

Por otro lado, los switches son dispositivos dedicados responsables de la conexión de múltiples dispositivos de distintas redes a un router.

Un switch puede operar con una funcionalidad que le sitúa o bien en la capa 2 (Data Link) o bien en la capa 3 (Network) de forma excluyente (no pueden operar en varias capas al mismo tiempo):

- Un switch de capa 2 redireccionará *frames* (recordemos que los frames son un contenedor de paquetes, la estructura que contiene la cabecera IP y el IP payload, no es el paquete completo) al dispositivo correcto.

- Un switch de capa 3 es algo más sofisticado que uno de capa 2 y puede desarrollar algunas compentencias de un router convencional en el sentido de que además de frames pueden enrutar paquetes.

Por otra parte, tenemos el concepto de **VLAN** o Virtual Local Area Network, que permite a dispositivos específicos dentro de la misma red.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220706230453.png' | relative_url }}" text-align="center"/>
</div>

Esto está construido con ayuda de los switches, y en este contexto ambos departamentos no pueden comunicarse entre sí pese a pertencer al mismo router.

<br />

**Protocolos en acción**

Tenemos como último paso un evento que simula una comunicación real entre dos dispositivos en el que se nos pide que envíemos un paquete TCP de un dispositivo a otro. Se producen los siguientes eventos. Las flechitas indican el sentido de la comunicación:

|nº|Sentido|evento|descripción|
|-|-|-|-|
|1 | ---> |ARP Request|Con un *ARP request* se lleva a cabo un mensaje de difusión (broadcast) a todos los ordenadores con orden de que el propietario de una IP se identifique con su MAC.|
|2 | <--- |ARP Response|El propietario de la IP se identifica con su MAC mediante el envío de un ARP Response. En el caso de que el dispositivo buscado no se encuentre dentro de la red será el dispositivo el que envíe el ARP Response.|
|3 | ---> |TTWH SYN|Como se trata de un paquete TCP se formaliza una conexión entre ambos dispositivos envíando un paquete SYN de TCP Three-Way Handshaking. Notemos que cuando el paquete pasa a la otra red, antes que nada, se repite el proceso de búsqueda del dispositivo en cuestión mediante el protocolo ARP.|
|4 | <--- |TTWH SYN/ACK| El dispositivo con el que se quiere empezar una conexión responde con un mensaje de confirmación ACK al SYN, denominado SYN/ACK.|
|5 | ---> |TTWH ACK|El impulsor de la conexión termina con un último ACK al SYN/ACK antes de mandar el DATA.|
|6 | ---> |DATA|El contenido que se quiere transmitir de un dispositivo al otro.|
|7 | <--- |ACK|El receptor del DATA responde con un mensaje de confirmación al recibimiento del paquete.|

Posteriormente habría una terminación de la comunicacíon mediante un FIN, ACK, FIN, ACK. Pero el paquete ha sido mandado.

<br />

### 2. Web Fundamentals.

#### 2.1. DNS en detalle.

**Presentación del protocolo DNS**

DNS son las siglas de Domain Name System, se trata de un protocolo que asocia un dominio, esto es; una entidad informática (o conjunto de ellas) identificadas por un nombre con su respectiva IP.

<br />

**Jerarquía de dominios**

Entre las partes que conforman un dominio distinguimos:

- **TLD**: Son las siglas de Top-Level Domain. Un TLD es la parte más a la derecha de un nombre de dominio. Dentro de los TLD se encuentran el gTLD que históricamente tenía el propósito de definir cuál era la función del dominio y el ccTLD que tenía propósitos geográficos.

- **Second-Level Domain**: Es el termino inmediantamente a la izquierda del TLD. Está limitado a 63 caracteres.

- **Subdominio**: El subdominio es el término a la izquierda del Second-Level Domain, generalmente separado por un punto.

Así por ejemplo, en el término admin.tryhackme.com, admin es el subdominio, tryhackme es el second level domain y .com es el TLD.

<br />

**DNS Record Types**

Sin embargo, DNS también actúa más allá de los sitios web. Existen muchos tipos de registros de DNS:

- A Record: Este se encargar de asociar direcciones IPv4, las más convencionales.
- AAAA Record: Este registro asocia IPv6.
- CNAME Record: Este tipo de registro asocia un nombre con otro en lugar de con una IP.
- MX Record: Estos asocian un nombre con el servidor que gestiona los mails.
- TXT Record: Se trata de listas de servidores que pueden mandar un mail en nombre del dominio.

<br />

**Making a request**

Cuando se realiza una request se llevan a cabo los siguientes pasos:

1. En primer lugar, el ordenador busca en el caché para comprobar si has buscado antes dicha dirección.

2. En segundo lugar, se firige sobre un Recursive DNS Server, que tiene un conjunto de dominios buscados recientemente por otros ordenadores en la misma red.

3. Si no se consigue localizar la IP adecuada mediante el Recurisve DNS se dirige al Root DNS Server, el cual es una base de datos mayor.

<br />

#### 2.3. HTTP in detail.

**Presentación de HTTP/s**

HTTP son las siglas de HiperText Transfer Protocol, se trata de un protocolo que implementa unas reglas dirigidas al transporte de datos de páginas web, así como HTML, vídeos, imágenes, etc.

En HTTPs, la s viene de secure porque el contenido se cifra.

<br />

**Requests & Responses. URL.**

Cuando intentamos acceder a un sitio web desde un navegador este necesita hacer una request para adquirir elementos web del servidor como HTML, imagenes, etc y descarga el contenido.

Pero antes, se necesita comunicar al navegador dónde y cómo acceder a estos recursos.

<br />

Es en este punto en el que entra en juego la URL. Esta en la práctica no es más ue una instrucción que indica cómo acceder a un recurso desde internet.

Una URL se compone de los siguientes componentes:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707142103.png' | relative_url }}" text-align="center"/>
</div>

<br />

|Nombre|Descripción|
|-|-|
|Scheme|Protocolo que se utiliza para acceder al recurso. También podría utilizarse por ejemplo, HTTPs, FTP, etc.|
|User|Credenciales de autenticación requeridas por algunas páginas web.|
|Host/Domain|El dominio o dirección IP del servidor que custodia el recurso.|
|Port|El puerto al que conectarse que dependerá del servicio en el que esté basado el recurso. Por ejemplo, un documento HTML será un recurso web accesible desde el puerto 80 o 443, un servicio de administración remota por SSH es un recurso SSH accesible desde el puerto 22, etc.|
|Path|La ruta del fichero la que se intenta acceder.|
|Query String|Información extra que puede estar requerida en forma de valores, preferencias, parámetros, etc.|
|Fragment|Referncia de una localización de una página actual.|

Una vez tenemos la URL, el buscador genera un documento HTTP con la siguiente forma:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707152857.png' | relative_url }}" text-align="center"/>
</div>

Esta es la forma más básica que puede tener una request, está compuesta de una línea que específica el método de la request (GET) y la versión de HTTP en la que está diseñado el documento (HTTP/1.1).

Seguidamente, se añaden una serie de cabeceras que el servidor puede necesitar para gestionar adecuadamente la request.

Automáticamente el servidor nos devolverá una HTTP Response como la siguiente:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707153240.png' | relative_url }}" text-align="center"/>
</div>

En el que además de lo anterior se añade el código HTML. El contenido y significado de las cabeceras se tratará más adelante.

El resumen es sin embargo que nuestro buscador emplea la URL a modo de instrucción para construir de forma eficiente una HTTP request a un servidor para acceder a un recurso web. Este nos devolverá una HTTP Response que nuestro navegador procesará y nos mostrará permitiéndonos acceder al recurso si es el caso.

<br />

**HTTP Methods**

Un método HTTP es una herramienta que define lo que queremos del servidor a grandes rasgos. A cada método acompañan una serie de características o parámetros:

- *GET*: Empleamos el método GET para leer u obtener un recurso. Un GET satisfactorio devuelve una respuesta conteniendo la información pedida.

- *POST*: Empleamos el método POST para crear un nuevo recurso en el servidor. El POST necesita de un cuerpo en el que se define los datos con los que el nuevo recurso va a crearse.

- *PUT*: Empleamos el método PUT para modificar un recurso. De nuevo necesita un body con el contenido que va ha utilizarse para modificar recurso.

- *DELETE*: Se emplea para borrar información. Se necesita de nuevo un body como en los anteriores casos.

<br />

**HTTP Status Codes**

En las respuestas, tal y como hemos podido ver en el ejemplo anterior, se incluye un código que indica el status de la operación producida por la request. Estos códigos consisten en números de tres cifras pertenecientes a un conjunto de rangos con distintos significados:

<br />

- **100-199**: Information Response. No son códigos muy comunes. Están para indicar que la primera parte de la request ha sido aceptada y que deben de seguir mandando requests
- **200-299**: Success. La request se ha procesado satisfactoriamente.
- **300-399**: Redirection. Este rango están para indicar que el navegador debe realizar otra request a otra dirección.
- **400-499**: Client Errors. Empleados para informar al cliente de que ha habido un error al procesar su request.
- **500-599**: Server Errors. Empleados para informar al cliente de que ha ocurrido un error de procesamiento con su request en el lado del servidor.

<br />

Algunos de los códigos más comunes son:

- **200** - OK
- **201** - Created. El recurso ha sido creado satisfactoriamente.
- **301** - Permanent Redirect. El recurso se ha movido de sitio a otra ubicación de forma permanente.
- **302** - Temporary Redirect. El recurso se ha movido de sitio temporalmente.
- **400** - Bad Request. Esto informa al cliente de que la request no ha sido construida correctamente.
- **401** - Not Authorised. El cliente tiene restringido el acceso a dicho recurso debido a una falta de autenticación o derivados.
- **403** - Forbidden. El cliente tiene el acceso restringido se autentique o no.
- **404** - Page not found. Página no encontrada.
- **500** - Internal Service Error. El servidor a encontrado un error en la request y no sabe cómo manejarla adecuadamente.
- **503** - Service Unavialable. El servidor no puede manejar la request debido a un fallo de mantenimiento del mismo.

<br />

**Headers**

Son bits extra de información que pueden ser envíados en la request. Algunas de ellas son necesarias en la mayoría de los casos para el correcto procesamiento de la request.

Veámos algunas de las cabeceras más comunes:

- **Host**: El valor de esta cabecera contiene la dirección del dispositivo al que la request se dirige. Muchos servidores web hostean varias aplicaciones web al mismo tiempo. La cabecera host puede servir para distinguir la aplicación web a la que se dirige aunque el último uso que se hace del contenido de dicha cabecera depende de la aplicación web y su configuración. En muchos casos, un uso inapropiado de la cabecera host puede liderar a múltiples vulnerabilidades muy peligrosas.

- **User-Agent**: El contenido de esta cabecera contiene el software y el tipo de versión del que está hecho el buscador.

- **Content-Length**: El contenido de esta cabecera contiene el tamaño del cuerpo de la request. Es aquello que contiene ni la primera línea ni la cabecera.

- **Accept-Encoding**: Le dice al servidor que tipo de comprensión se ha empleado a la hora de transferir los datos.

- **Cookie**: Datos enviados en forma de parámetros extra.

<br />

Algunas cabeceras propias de las respuestas HTTP:

- **Set-Cookie**: Información que se almacena y se devuelve en cada respuesta.

- **Cache-Control**: Cuando tiempo (en segundos) lleva almacenada la respuesta en cache. Asocida a esta generalmente aparecen las cabeceras Cache-hit y Max-Age que definen respectivamente si la respuesta se cachea y cuál es el tiempo máximo que una respuesta aparece en cache.

- **Content-Type**: Esto informa al cliente qué tipo de datos se han devuelto.

- **Content-Encoding**: El método empleado para comprimir los datos.

<br />

**Cookies**

Las cookies, son solo una pequeña parte de los datos que se almacenan en la máquina y se envían al servidor siempre que se lleva a cabo una request.

Las cookies se guardan cuando recibe un encabezado "Set-Cookie" de un servidor web. Luego, cada solicitud adicional que realice, enviará la cookie. Debido a que HTTP no tiene estado (no realiza un seguimiento de sus solicitudes anteriores).

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183745.png' | relative_url }}" text-align="center"/>
</div>

Las cookies pueden verse fácilmente desde el navegador desde Inspeccionar Elemento > Application (o Storage en firefox) > Cookie.

<br />

#### 2.4. How websites works.

**Front-end y Back-end**

Ahora que tenemos una noción básica acerca de cómo funcionan los dos protocolos más importantes que hay en internet, DNS y HTTP, veamos cómo un sitio web hace uso de ellos para proporcionar una serie de servicios a los que un cliente desde su navegador puede acceder.

Cuando visitas un sitio web desde el buscador el navegador construye una request a partir de la URL pidiendo información para cargar la página al sitio web que estás pidiendo. La respuesta contiene la información necesaria para que el navegador pueda renderizar la página:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707211019.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, distinguimos en lo que a una aplicación web se refiere, dos grandes componentes:

- **front-end**: Es aquella parte de la aplicación web que queda a disposición del cliente como las páginas web.

- **back-end**: Es aquella parte que queda en el lado del servidor, que no se muestra al cliente y que se utiliza para formar el front end.

<br />

**HTML**

El Client-side de un sitio web se construye con tres componentes básicos:

- *HTML*, para construir la estructura de la página.
- *CSS*, para añadir opciones de decoración y estilo.
- *JavaScript*, para añadir funcionalidades complejas de interacción con el usuario a través del DOM.

<br />

Nos centraremos en el documento HTML, este suele tener una estructura parecida la que sigue:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707212301.png' | relative_url }}" text-align="center"/>
</div>

De forma que en ella podemos diferenciar los siguientes elementos:

- La entidad *\<!DOCTYPE html>* define que una página es un documento HTML5.

- La entidad \<html> define el comienzo del documento HTML, que consta de dos partes bien diferenciadas, la cabecera (head) y el cuerpo (body).

- La entidad \<head> contiene información sobre la página como su título y derivados. Sin embargo, nada de lo que hay en esta cabecera aparece renderizado en el navegador.

- La entidad \<body> define el cuerpo HTML que será renderizado por el navegador. Dentro de este podemos encontrar otras dos entidades HTML muy comunes:

- \<h1>; que define un encabezado.
- \<p>; que define un párrafo.

Observamos que toda entidad \<tag> se cierra con su respectivo \</tag>.

Además, existen muchos tipos de entidades HTML que desempeñan funcionalidades distintas cada una de ellas y eso provoca que puedan contener atributos, u otro tipo de parámetros dentro del mismo tag.

<br />

**JavaScript**

JavaScript es uno de los más lenguajes de programación más populares y permite a las páginas volverse interactivas para el usuario, sin él serían completamente estáticas.

El código JavaScript se inscrusta en el código HTML mediante la etiqueta \<script>.

```HTML
<script>javascript/code</script>
```

También pueden cargarse desde una localizacion mediante el atributo "src":

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707221249.png' | relative_url }}" text-align="center"/>
</div>

Además, existen eventos especificos de ciertas entidades HTML que cargan código JavaScropt como "onmouseover", "onclick", etc.

<br />

#### 2.4. Other componentes. How work server websites.

**Other components**

- *Loaf Balancers*: Los balanceadores de carga son dispositivos que proveen dos características fundamentales:

- Asegurarse de que el sitio web puede gestionar el tráfico de red.
- Comprobar el estado del servidor mediante **health checks**. Enviar un mensaje de error si el servidor cae.

Tienen una importancia fundamental en servidores web expuestos a una gran cantidad de tráfico y que necesitan aportar una gran disponibilidad.

Se dedican a recibir el tráfico y desviarlo en función de un conjunto de algoritmos que le ayudan a decidir qué servidor puede lidiar, mejor con la request.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220708094625.png' | relative_url }}" text-align="center"/>
</div>

- *CDN*: El CDN o Content Delivery Networks puede ser un recurso excelente para cortar el tráfico de un sitio web ocupado.

- *Database*: Estructuras que son empleadas para almacenar información en particular por aplicaciones web.

- *WAF*: Una responde a Web Application Firewall y se trata nada más y nada menos que un firewall situado entre el cliente y el servidor de forma que intercepta las request, las analiza y decide si las deja pasar o no. Es un método de protección contra ataques convencionales, analiza la request, el número de request por monto de tiempo, etc.

<br />

**Web Servers. Virtual Hosting. Static & Dinamic content. Back-end languages.**

Un *web server* o servidor web es un software que corre en una máquina denominada *servidor* que escucha peticiones entrantes y emplea el protocolo HTTP para compartir contenido web a los clientes. Existen muchos tipos de web servers, Apache, Nginx, NodeJS, etc.

Un servidor web comparte ficheros a partir de lo que se conoce como el directorio raíz (root directory) que su localización por defecto es: /var/www/html o C:\\inetpub\\wwwroot pero que puede configurarse.

Así por ejemplo, si realizamos una request para obtener la imagen https://example.com/image.png, el servidor buscará la imagen en /var/www/html/image.png (C:\\inetpub\\wwwroot\\image.png) y la enviará en la HTTP response.

<br />

Un solo servidor web puede hostear varias aplicaciones web con diferentes dominios a través de *virtual hosts*. El servidor web parsea la HTTP request y en concreto analiza el valor de la cabecera Host para saber qué aplicación web refiere la request.

**A continuación busca el recurso en el host virtual, estos son sólo documetnos de textos almacenados en carpetas de la forma /var/www/website_x**. En principio no hay límite en el número de websites que pueden hostear.

<br />

Dentro del contenido que proporciona un servidro web y al que el cliente tiene acceso distinguimos fundamentalmente:

- Contenido estático, que no cambia en función de la request, como imagenes, código JavaScript, etc.
- Contenido dinámico, que cambia en función de la request, como comentarios, entradas de un blog, etc.

<br />

Aquello que se procesa en el backend suele estar guiado a partir de instrucciones en un lenguaje de programación que el servidor web entiende como PHP, Java, Python, Ruby, etc.

<br />

### 3. Linux Fundamentals.

#### 3.1. Fundamentos de Linux I.

**Presentación**
Linux es un sistema operativo basado en UNIX, es de código abierto (libre de modificar por otros usuarios) y con una gran cantidad de "flavours" o versiones cada una con su propósito, ventajas y desventajas.

<br />

**Comandos importantes**

Existe una herramienta muy importante en cualquier sistema Linux denominada Terminal. Este es un programa que ejecuta comandos que se traducen en instrucciones del sistema operativo y con los que podemos utilizar el mismo sin límites.

Algunos de los comandos más importantes son:

|Nombre|Descripción|
|-|-|
|echo|Sacar por pantalla lo que se le pasa como argumento al comando.|
|whoami|Saca por pantalla el usuario actual de la máquina.|
|ls|Lista el contenido de la carpeta actual.|
|cd|Cambia de directorio al definido por la ruta que se le pasa al comando como argumento.|
|cat|Muestra el contenido del fichero que se le pasa como argumento.|
|pwd|Muestra la ruta desde la raíz hasta el directorio actual.|
|find|Busca un fichero en nombre de una especificación que puede ser el nombre, el tamaño del fichero, la carpeta a buscar, etc. Para más información, man find en la terminal.|
|grep|Filtra el output de un comando mediante un término u otras especificaciones. Para más información man grep.|

<br />

**Operadores**

Existen unos comandos que ejecutan instrucciones sobre el resultado de otros comandos comúnmente conocidos como *operadores*. Algunos de los más comunes son:

|Nombre|Descripción|
|-|-|
|&|Este operador manda a segundo plano el proceso iniciado por el comando a su izquierda.|
|&&|Este comando permite concatenar un segundo comando de forma independiente después de un primero.|
|>|Este comando redirige el outpu (que normalmente saldría por pantalla) a cualquier receptor de output indicado a su izquierda, por ejemplo un fichero.
|>>|Este comando hace lo mismo que el anterior pero lo hace sin sobreesribir, de forma que si redirijiéramos el output sobre un fichero, si este fichero contuviera algo escrito, no se borraría.|
|\||Este comando introduce el output del comando de la izquierda como input al comando de la derecha. Ejemplo: cat lista.txt \| grep teléfono|

<br />

#### 3.2. Fundamentos de Linux II.

**SSH**
También podemos conectarnos a una máquina Linux mediante SSH. SSH es un protocolo cryptográfico que posee una funcionalidad de administración remota que envía los datos cifrados.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220708184350.png' | relative_url }}" text-align="center"/>
</div>

Nos logeamos a una máquina remota en Linux con SSH mediante el siguiente comando:

```bash
ssh username@IPAdress
```

<br />

**Introducción a modificadores**

Muchos de los comandos admiten argumentos y opciones que modifican su comportamiento de una forma concreta. A estos argumentos especiales se les llama modificadores y suelen ir precedidos de uno o dos guiones. Por ejemplo, el comando "ls" que lista ficheros tiene el modificador 'a' que muestra los ficheros ocultos (aquellos cuyo nombre comienza con un punto)

```bash
ls -a
```

El comando anterior listaría los contenidos del directorio actual incluyendo aquellos ficheros ocultos, no mostrados por el "ls" convencional.

Para obtener una ayuda de los distintos modificadores que pdoemos aplicar a un comando tenemos generalemte el modificador "--help" o el comando "man \<comand>" que nos aportará además una pequeña descripción del comando. (Man responde a manual).

<br />

**Continuación de comandos importantes**

|Comando|Nombre completo|Descripción|
|-|-|-|
|touch|touch|Crear un fichero|
|mkdir|make directory|Crea un directorio (carpeta)|
|cp|copy|Copia un fichero (o un directorio con -r, recursively)|
|mv|move|Mueve un fichero o carpeta, renombrar|
|rm|remove|Borra un fichero o carpeta|
|file|file|Determina el tipo de contenido que contiene un fichero|
|su|su|Cambia al usuario especificado en el argumento.|
<br />

**Permisos**

Uno de los modificadores que exiten para el comando "ls" que lista el contenido de una carpeta es el '-l', que expone los detalles de los archivos listados:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220708204114.png' | relative_url }}" text-align="center"/>
</div>

Concretamente, nos interesa el dato proporcionado por la primera columna. Los permisos que los distintos usuarios del sistema tienen.

- Por un lado, estos se dividen en, el usuario propietario, el grupo propietario y otros.
- Por otro lado, los permisos se exponen con:
- r: read (permisos de lectura).
- w: escritura (permisos de escritura).
- x: execution (permisos de ejecución).

De forma que en el ejemplo anterior nos dice que ambos ficheros; file1, file2 son ficheros puesto que hay un guión al princpio de la línea (si fuera una carpeta sería una d), el propietario puede leer y escribir en el documento, el grupo puede leer y cualquier otro usuario puede solamente leer.

Debemos especificar que en linux, un usuario y un grupo no son más que dos ordenes de clasificación distintos, identificados por un "uid" (user id) y un "gid" (group id) respectivamente.

<br />

**Directorio más comunes**

- /etc -> Archivos de configuración.
- /var -> Archivos cuyo contenido varia o se actualiza constantemente, como los logs, etc.
- /root -> Directorio personal de usuario root.4

<br />

#### 3.3. Fundamentos de Linux III.

**Utilidades generales**

Existen un conjunto de comandos especialmente útiles:

- *wget*: Este comando permite descargar ficheros vis HTTP. Los ficheros puede abrirse con un navegador.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709094911.png' | relative_url }}" text-align="center"/>
</div>

<br />

- *scp*: Permite enviar ficheros de un host a otro via SSH.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709100108.png' | relative_url }}" text-align="center"/>
</div>

Y para recuperarlo:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709100143.png' | relative_url }}" text-align="center"/>
</div>

<br />

- *curl*: Tiene el mismo propósito que curl

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709100654.png' | relative_url }}" text-align="center"/>
</div>

<br />

- *WebServers*: Podemos montar un servidor web mediante el siguiente comando:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709100944.png' | relative_url }}" text-align="center"/>
</div>

De forma que desde otro sistema podría acceder a los archivos contenidos en nuestra máquina con wget:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709101223.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Procesos. Definición y manejo.**

Proceso es un término que alude a un programa en ejecución registrado y numerado por el sistema.

El usuario puede acceder a la forma en la que el sistema ordena los procesos mediante un conjutno de comandos, como ps que muestra los procesos:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165536.png' | relative_url }}" text-align="center"/>
</div>

top, que los muestra de una forma dinámica:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165537.png' | relative_url }}" text-align="center"/>
</div>

<br />

Con el fin de gestionar procesos tenemos el comando kill, que los termina. Para empezarlos tenemos los comandos *systemctl* o *service*. Por ejemplo: systemctl start apache2

Las opciones de systemctl son:

- start
- stop
- enable
- disable

Cuando se ejecuta un proceso podemos emplear las teclas Ctrl+C para pararlo, o Ctrl+Z para llevarlo a segundo plano y que no bloquee la terminal. También podemos devolverlo a primer plano con 'fg'.

<br />

**Automatización de procesos**

Para automatizar procesos podemos emplear *cron* que nos permite ajustar la hora y la periodicidad de ejecución de un proceso especificado concreto.

<br />

### 4. Windows Fundamentals.

#### 4.1. Fundamentos de Windows I.

**Introducción al Windows OS**

El sistema operativo Windows es actualmente el dominante tanto en redes corporativas como en redes domésticas. Por esta razón, Windows es uno de los sistemas más atacados.

<br />

**Escritorio de Windows**

El escritorio de Windows emplea el GUI (Graphical User Interface) para mostrar un escritorio interactivo y visual en la máquina.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165538.png' | relative_url }}" text-align="center"/>
</div>

1. The Desktop
2. Start Menu
3. Search Box (Cortana)
4. Task View
5. Taskbar
6. Toolbars
7. Notification Area

<br />

- *Desktop*:

El escritorio es una sección en la que comúnmente encontramos accesos directos a otros artefactos informáticos. Así como carpetas, aplicaciones, etc.

La apariencia del escritorio puede ser modificada al gusto así como los iconos que muestran, está diseñado para ser cómodo, personalizado y ofrecer un acceso rápido a los aparatos con los que el usuario suele intarctuar más amenudo.

<br />

- *The Start Menu*:

El "start" menu es una parte de Windows que da acceso completo a cualquier otra parte del dispositivo

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165539.png' | relative_url }}" text-align="center"/>
</div>

Se divide en la barra de búsqueda, las aplicaciones más recientemente añadidas y una sugerencia de Windows.

<br />

**The File System**

El sistema de ficheros empleado en las versiones más modernas es el *NTFS*, siglas que responden a *New Technology File System*.

Antes se encontraban FAT16/32 (File Alocation System) y el HPFS (High Performance File System).

NTFS se conoce como un "journaling file system" (sistema de ficheros basado en registro). Es decir, **en caso de fallo, el sistema de archivos puede reparar automáticamente las carpetas/archivos en el disco utilizando la información almacenada en un archivo de registro**. Esta función no es posible con sistemas anteriores.

Entre algunas de las características más importantes están:

- Soporta ficheros que pesan más de 4GB
- Puede configurar permisos específicos sobre carpetas y ficheros. Estos son permisos son:
- Full control.
- Modify.
- Read & Execute.
- List folder contents.
- Read.
- Write.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165540.png' | relative_url }}" text-align="center"/>
</div>

Los permisos pueden verse mediante los siguientes pasos: Click-derecho sobre carpeta > Propiedades > Seguridad > Usuario o grupo cuyos permisos se desean ver.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165541.png' | relative_url }}" text-align="center"/>
</div>

- Admite archivos comprimidos (ya sean carpetas o ficheros).
- El sistema de archivos puede cifrarse.
- Alternate Data Streams (ADS), permite archivas metadatos de un fichero sin necesidad de crear otro fichero para almcenar estos metadatos creando más de un flujo de datos para un solo fichero. Desde la perspectiva de la ciberseguridad esto es importante debido a que se podría ocultar datos en un segundo streams de datos.

<br />

**The Windows\\System32 Folders**

La carpeta "C:\\Windows" se conoce tradicionalmente como la carpeta que contiene el sistema operativo de windows, aunque técnicamente y variando convenientemente las "variables de entorno" no tiene por qué encontrarse en esa localización tan concreta.

Dentro de esta carpeta encontramos un montón de archivos:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165542.png' | relative_url }}" text-align="center"/>
</div>

Y entre todas esas carpetas encontramos específicamente la carpeta System 32:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165543.png' | relative_url }}" text-align="center"/>
</div>

Esta carpeta contiene archivos que pueden ser críticos para el sistema operativo, borrarlos podría resultar en una falta de funcionalidad por parte del sistema.

<br />

**User Accounts Profiles and Permissons**

En Windows hay dos tipos de usuarios: Administrator o Standard User.

El tipo de cuenta de usuario determina las acciones que el mismo puede hacer:

- Administartor: puede llevar a cabo cambios: añadir usuarios, borrar usuarios, modificar configuraciones, etc.

- Standard User: Sólo puede hacer modificaciones sobre archivos sobre los que tiene propiedad.

Para ver qué otros usuarios hay en la máquina podemos buscar en "Start Menu", Other User y acudir a System Settings > Other users.

Todos los usuarios tienen asociados una serie de "carpetas personales" (Desktop, Documents, Downloads, ...) estos documentos son accesibles desde el Local User and Group Management: Win+R > lusrmgr.msc

<br />

**User Account Control**

UAC son siglas que responde a User Account Control y se trata de una herramienta de protección y acompasamiento entre un usuario con capacidades de adminstrador y el sistema.

Concretamente, sirve para que cada vez que el usuario lleva a cabo una acción (él o una aplicación que él ejecuta) no tenga que elevarse a administrador sino sencillamente conceder permisos. Cada vez que se requiere salta una ventana que pide al usuario una concesión de permisos. Cada vez que esto pueda ser necesario para una aplicación aparecera un escudo junto al logotipo indicando que la UAC elevara privilegios al ejecutar el programa:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220709165544.png' | relative_url }}" text-align="center"/>
</div>

Una vez intentemos ejecutar el programa no saltara una ventanita como la siguiente:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183746.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Settings and The Control Panel**

*Settings* y *Control Panel* son dos conjuntos de opciones donde podemos modificar preferencias a nuestro gusto Windows 10:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183747.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183748.png' | relative_url }}" text-align="center"/>
</div>

Desde opciones de configuración de cuentas, red, a apariencia del escritorio, etc.

<br />

**Task Manager**

El *Task Manager* (Administrador de tareas) proporciona información sobre las aplicaciones y los procesos que se ejecutan actualmente en el sistema. También hay disponible otra información, como la cantidad de CPU y RAM que se utilizan, que se incluye en Rendimiento.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183749.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 4.2. Fundamentos de Windows II.

**System Configuration**

El *System Configuration* (msconfig) es una utilidad la resolución avanzada de problemas. Su propósito principal es el de ayudar a diagnosticar problemas del inicio del equipo.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183750.png' | relative_url }}" text-align="center"/>
</div>

Es probable que se necesiten permisos de administrador para utilizar esta herramienta.

Una vez hemos abierto esta herramienta encontramos cinco pestañas: General, Boot, Services, Startup, Tools.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183751.png' | relative_url }}" text-align="center"/>
</div>

- *General*: Desde esta pestaña podemos elegir qué dispositivos o servicios se inician desde el inicio del sistema.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183752.png' | relative_url }}" text-align="center"/>
</div>

- *Boot*: Definimos opciones de arranque del sistema.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183753.png' | relative_url }}" text-align="center"/>
</div>

- *Service*: Muestra una lista de todos los servicios (procesos en segundo plano) que corren en el sistema:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183754.png' | relative_url }}" text-align="center"/>
</div>

- *Tools*: Presenta una lista de herramientas que pueden configurarse y que esta instaladas.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183755.png' | relative_url }}" text-align="center"/>
</div>

Notemos el recuadro "Selected Command"

<br />

**Change UAC Settings**

Se pueden cambiar los controles de la UAC para que sea menos restrictiva. Existen diversos niveles:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183756.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Computer Management**

Dentro del panel del *System Configuration* encontramos el *Computer Management* (compmgmt) la cual tiene tres secciones: *System Tools*, *Storage*, *Services & Applications*.

- *System Tools*: Dentro de esta sección tenemos una serie de herramientas:

- *Task Scheduler*: Con esta herramienta podemos programar la ejecución de una tarea.

- *Event Viewer*: Nos permite acceder al registro de eventos. Existen 5 tipos de eventos:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183757.png' | relative_url }}" text-align="center"/>
</div>

Y cuatro tipos de logs:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183758.png' | relative_url }}" text-align="center"/>
</div>

- *Shared Folder*: Un listado completo de todas las carpetas compartidas a otros usuarios.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183759.png' | relative_url }}" text-align="center"/>
</div>

- *Local Users and Grupos*: Nos lleva al archivo "lusrmgr.msc" ya visto en la sección anterior.

- *Perfomance*: Tenemos acceso al Perfomance Monitor (perfom).

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183760.png' | relative_url }}" text-align="center"/>
</div>

- *Storage*: En Storage podemos encontrar una interfaz que nos guía a lo largo del manejo del disco duro:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183761.png' | relative_url }}" text-align="center"/>
</div>

En esta interfaz podemos crear un nuevo espacio, extender una partición, partir una partición o asignar una letra a una parte del espacio vacío.

- *Services & Applications*: Aplicaciones y servicios.

<br />

**System Information**

Windows contiene una herramienta denominada Microsoft System Information (Msinfo32.exe) que recopila información sobre el ordenador y despliega una vista comprensible del hardware, componentes del sistema, entorno del software que puede ser empleado para detectar problemas del ordenador.

La información se divide en tres secciones principalmente:

- Harware Resources.
- Components: Información específica de dispositivos instalados en el sistema.
- Software Enviroment: Variables de entorno y Network.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183762.png' | relative_url }}" text-align="center"/>
</div>

Particularmente importante es la última sección:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183763.png' | relative_url }}" text-align="center"/>
</div>

Y la sección de Network:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183764.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Resource Monitor**

Resource Monitor muestra información de uso de CPU, memoria, disco y red por proceso y agregada, además de proporcionar detalles sobre qué procesos están utilizando identificadores de archivos y módulos individuales. El filtrado avanzado permite a los usuarios aislar los datos relacionados con uno o más procesos (ya sean aplicaciones o servicios), iniciar, detener, pausar y reanudar servicios y cerrar aplicaciones que no responden desde la interfaz de usuario. También incluye una función de análisis de procesos que puede ayudar a identificar procesos bloqueados y conflictos de bloqueo de archivos para que el usuario pueda intentar resolver el conflicto en lugar de cerrar una aplicación y perder datos potencialmente.

Existen cuatro pestañas fundamentalmente:

- CPU
- Disk
- Network
- Memory

<br />

**cmd**

La **cmd** es la terminal de Windows. Existen algunos comandos importantes como:

|nombre|descripción|
|-|-|
|hostname|Despliega el nombre de la computadora.|
|whoami|El nombre del usaurio logeado.|
|ipconfig|Despliga información de red.|
|/?|Despliega información del manual del comando precedente. Por ejemplo: ipconfig /?|
|cls|Clear.|
|netstat|Información de red en TCP/IP. Admite parámetros: -a, -b, -e.|
|net|Opciones de red.|

<br />

**Registry Editor**

El Windows Registry es una base de datos jerarquica empleada para almacenar información necesaria para configurar el sistema para uno o más usuarios, aplicaciones, hardware, etc.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220707183765.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 4.3. Fundamentos de Windows III.

**Windows Update**

Windows Update es un servicio proveido por Microsoft para obtner actualizaciones de seguridad, mejoras de las características y parches para el OS Windows.

Las actualizaciones generalmente se lanzan el segundo Jueves de cada mes (Patch Tuesday).

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711083528.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Windows Security**

Windows Security es una herramienta de Microsoft para proteger el dispositivo

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711083529.png' | relative_url }}" text-align="center"/>
</div>

Existen distintos controles de seguridad:

- Virus & Thread protection.
- Firewall & network protection.
- App & browser control.
- Device Security.

Existen marcadores de status para cada uno de estos campus.

- Verde: OK.
- Amarillo: Recomendaciones de seguridad adicionales.
- Rojo: Alerta.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711083530.png' | relative_url }}" text-align="center"/>
</div>

<br />

**Virus & Threat protection**

Esta sección está dividida en dos partes:

- *Amenazas actuales*: En esta sección podemos encontrar la posibilidad de realizar un escaneo o de comprobar los resultados de escaneos anteriores.

- *Configuración sobre virus y amenazas*: Configuraciones referente a actualizaciones, protección en tiempo real, protección en la nube, etc.

<br />

**Firewall & network protection**

El *firewall* regula el flujo de tráfico entre dispositivos.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711083531.png' | relative_url }}" text-align="center"/>
</div>

Como podemos ver, existen teers tipos de firewalls:

- Domain: Cuando el host puede autenticatrse a un controlador de dominios.

- Private: Para diseñar protecciones contra redes privadas o domésticas.

- Public: Empleado para diseñar protección contra redes públicas.

Al clickar en alguna de estas opciones accedemos a la posibilidad de activar o desactivar el firewall mencionado.

Dentro de Windows Defender Firewall podemos habilitar a una aplicación a través del mismo:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711085251.png' | relative_url }}" text-align="center"/>
</div>

Además puedes ver la configuración actual del perfil a través del botón "Details":

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711085334.png' | relative_url }}" text-align="center"/>
</div>

<br />

**App & browser control**

En esta sección se puede cambiar la configuración de Microsoft Defender SmartScreen. Esta se trata de una herramienta que protege contra ataques de phising o malware de sitios web.

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711102546.png' | relative_url }}" text-align="center"/>
</div>

De forma que si intentamos ejecutar una aplicación desconocida en el sistema aparecerá el siguiente mensaje:

<div style="text-align:center">
	<img src="{{ 'assets/img/THM/Pasted image 20220711102615.png' | relative_url }}" text-align="center"/>
</div>

<br />

**BitLocker**

BitLocker Drive Encryption es una característica de protección de datos integrada en el propio sistema operativo.

<br />

**Volume Shadow Copy Service (VSS)**

El VSS se coordina para realizar una serie de acciones para crear una copua de seguridad consistente.

Si el VSS está habilitado se pueden realizar las siguientes acciones:

- Crear un punto de restauración.
- Llevar a cabo una restauración del sistema.
- Configurar las opciones de restore.
- Borrar puntos de restauración del sistema.


