---
layout: post
title: Curso Básico Redes
subtitle: Introducción a conceptos básicos de redes.
tags: [redes] 
---

# 1. Modelo OSI.

El **modelo OSI** acrónimo de Open Systems Interconnection es un modelo conceptual que caracteriza y estandariza la función de comunicación entre sistemas computacionales (es decir, comunicación de redes) sin entrar en consideraciones acerca de la estructura interna de cada máquina desarrollado por ISO (International Standards Organization). Asienta las bases de la comunición operativa entre dos ordenadores mediante una **estructura jerárquica** de capas, es decir, la información pasa a través de las distintas capas hasta que finalmente se envía, este proceso por el que pasa la información se denomina como **encapsulamiento**. Esto es sólo un modelo teórico que está en desuso pero la comunicación entre ordenadores y entre redes se basan esta conceptualización teórica.



1. **Capa física**:  Se trata de la capa responsable de la transmisión y recepción de datos desestructurados a través un medio de transmisión físico, como cables de ethernet, wifi, etc. En esencia convierte los **bits** digitales en señales eléctricas, de radio, etc.

2. **Capa de enlace de datos**: Identifica cada IP con la dirección MAC (supuestamente única para cada dispostivo en dicha red). Detecta y corrige en la medida de los posible errores llevados a cabo en la capa física.
 
3. **Capa de red**: Se genera una especie de mapeado de la red que permite trazar caminos para el tráfico de red (routing).

4. **Capa de transporte**: Establece los protocolos (TCP, UDP) y puertos tanto de fuente como de destino con los que se van a efectuar la transmisión de datos.
 
5. **Capa de sesión**: Se trata de la responsable de establecer y terminar conexiones entre dispositivos.
 
6. **Capa de presentación**: Esta capa se asegura de formatear los datos de forma que la aplicación que va a recibir dichos datos pueda usarlos. Aquí también se produce el cifrado o descifrado de los datos.

7. **Capa de aplicación**: Interacción entre la computadora y el humano donde las aplicaciones pueden acceder al servicio de la red.


Así por ejemplo, supongámos que queremos enviar un email. Entonces el emisor escribiría el email mediante la capa 7 de aplicación que permite al usuario y la aplicación relacionarse, seguidamente los datos pasan a la capa 6 de presentación que se asegura de que los datos que se van a enviar se envían de forma que la aplicación receptora pueda entender dichos datos (y los cifra si es necesario). Seguidamente en la capa 5, de sesión, se inicia una sesión de servidor envío/recibo de emails, en la capa 4 se decide el protocolo de transporte de datos adecuado, en este caso al ser una petición compleja se utilizaría TCP, además también se asienta tanto el puerto de envío como el puerto de recibo. En la capa 3 se identifica el camino que va a seguir la información, en la capa 2 se identifica cada dirección IP con la MAC (física) y seguidamente la infomarción se envía en la capa física o 1. 

**Es decir; Los paquetes que se mandan a través de una red y atraviesan las distintas capas de un modelo de red (TCP/IP u OSI) van a su vez a través de los distintos protocolos de cada una de esas capas, IP, TCP, etc. Estos protocolos a su vez lo que hacen es añadir una cabecera al paquete  (encapsulamiento)  que contiene información relativa al paquete y a cómo debe de ser tratado el mismo de manera que cuando el paquete sale de una máquina en dirección a otra por la red lo que contiene es; por un lado la información del paquete en sí, y después un conjunto de cabeceras que le sirven, primero a los distintos routers para guíarlo a través de distintas redes hasta alguna que se haga responsable del mismo, y después a la máquina recibidora que procesa el paquete leyendo una a una las distintas cabeceras (desencapsulamiento) de los distintos protocolos conforme se asciende por las distintas capas del modelo de red en cuestión.**

Al llegar a la otra máquina, la información es procesada siguiendo los mismos pasos y las mismas capas de manera inversa hasta llegar a la capa 7 donde el usuario podrá leer el email correctamente.

<br />

# 2. Dirección MAC.

Todo dispositivo compatible con la red tiene almenos una ID de hardware única, esta es la dirección de control de acceso a los medios o **dirección MAC**.

Se tratade un identificador único (supuestamente) y que nos permite identificar a un ordenador concreto siempre y en todo momento, y sirve para el proceso de filtrado de redes inalámbricas, de esta forma se evita que extraños accedan a una red porque el router está configurado para aceptar sólamente una serie de direcciones MAC específicas. 

Siempre que una máquina desee ponerse en contacto con otra dentro de una red local necesitará de su dirección MAC o dirección física o de hardware.

La dirección MAC es un número hexadecimal de 12 dígitos (número binario de 6 bytes) representado por notación hexadecimal de dos puntos. (No confundir con IPv6 porque en este son 8 bytes)

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/Redes/MACAddr.png' | relative_url }}" text-align="center"/>
</div>

<br />

Tal y como se en la imágen, los primeros 6 dígitos (o 3 primeros bytes) identifican al fabricante, llamado OUI, los últimos 6 dígitos de la derecha representan el controlador de interfaz de red que es asignado por el fabricante, UAA. 

<br />

# 3. Protocolo ARP. 

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

## 4. Modelo TCP/IP.

## 4.1. Introducción.

TCP/IP es un conjunto de protocolos que permiten la comunicación entre los ordenadores pertenecientes a una red. La sigla TCP/IP significa Protocolo de control de transmisión/Protocolo de Internet. Proviene de los nombres de dos protocolos importantes incluidos en el conjunto TCP/IP, es decir, del protocolo TCP y del protocolo IP.

Tiene las siguientes características: 

- Fue desarrollado con posterioridad al modelo OSI.
- Consta de 5 capas: la capa de aplicación, la capa de transporte, la capa de network, la capa de enlace de datos y la capa física.

- La capa de "networking" en el modelo TCP/IP se corresponde con las primeras dos primeras capas del modelo OSI, y la capa de aplicación se corresponde con las tres últimas, tal y como se puede ver en la imágen de a continuación. 

- TCP/IP a resumidas cuentas se trata de un protocolo jerárquico hecho de "módulos interativos" y cada uno de ellos provee una funcionalidad específica.

<div style="text-align:center">
<img src="{{ 'assets/img/Redes/TCPvsOSI.png' | relative_url }}" text-align="center"/>
</div>



<br />

## 4.2. TCP/IP model vs OSI model.

| TCP/IP | OSI |
|-|-|
| TCP/IP tiene cuatro capas | OSI tiene 7 capas |
| TCP es más confiable | OSI es menos confiable |
| TCP/IP no tiene lazos demasiado estrictos | OSI tiene lazos más estrictos |
| TCP/IP tiene una aproximación horizontal | OSI sigue una aproximación vertical |
| TCP/IP primero desarrolla los protocolos y seguidamente desarrolla el modelo |OSI desarrolla primero el modelo y luego los protocolos. |
| La capa de red del modelo TCP/IP proporciona servicios sin conexión | OSI proporciona tanto servicios con conexion como servicios sin conexion|
|  No se pueden reemplazar lazos sencillamente en TCP/IP | En OSI los protocolos están mejor cubiertos y son más fáciles de reemplazar |



## 4.3. Capas del modelo TCP/IP.

### 4.3.1. Capa de acceso de red.

Esta capa de corresponde con la combinación de la 'capa física' y la 'capa de enlace de datos' en OSI. Busca la dirección del hardware y los protocolos presentados en esta capa permite la transmisión física de datos. Algunos de los protocolos de comunicación presentes en esta capa son:

-  **ARP**:  Atiende a Address Resolution Protocol (Protocolo de resolución de direcciones). Su trabajo consiste en averiguar la dirección del host conocida su IP. Este tiene varios tipos, reverse ARP, Proxy ARP, ARP Gratis y ARP inverso.
-  PPP:
-  Ethernet
-  Controladores de interfaz.

<br />

### 4.3.2. Capa de red.

Esta capa tiene una funcionalidad paralela a las funcionalidades de la capa de red del modelo OSI. En ella se definen protocolos que son responsables de la transmisicón lógica de datos sobre la red entera. Los principales a tener en cuenta en esta capa son los siguientes:

- **IP**: Atiende a Internet Protocol (Protocolo de red interna) es responsable de repatir paquetes desde la fuente al host destinatario buscando las direcciones de IP correspondientes en las cabeceras de los paquetes. IP tiene dos versiones:

	- **IPv4**: Es la que más sitios webs utilizan actualmente.
	- **IPv6**: Es una mejora respecto de IPv4 que tiene más gasto y poco a poco está sustituyendo a IPv4. Con ella desaparecera la dicotomía de entre las IPs públicas e IPs privadas.
	
-  **ICMP**: Atiende a Internet Control Message Protocol (Protocol de control de mensajes de Internet). Este protocolo está encapsulado dentro de los datagrams de los IP y es responsable de proveer a los host con información sobre problemas de red. 

-  **ARP**:  Atiende a Address Resolution Protocol (Protocolo de resolución de direcciones). Su trabajo consiste en averiguar la dirección del host conocida su IP. Este tiene varios tipos, reverse ARP, Proxy ARP, ARP Gratis y ARP inverso.

<br />

### 4.3.3. Capa de "host a host".

Esta capa es análoga a la capa de transporte del modelo OSI. Es responsable de la comunicación de un extremo a otro de la entrega de datos sin errores. Protege a las capas superiores de las posibles complicaciones de los datos. Los dos protocolos principales presentados en esta capa son los siguientes:

- **Protocolo de control de transmisión (TCP)**: Es conocido por prover comunicación fiable libre de errores entre finales de sistemas. Realiza secuenciación y segmentacíon de datos. 
- **Protocolo de datagrama del usuario (UDP)**: UDP por otro lado no proporciona tales características ni resulta ser un sistema tan fiable. Se trata del protocolo de referencia si tu aplicación no requiere un sistema de transporte muy fiable.

## 4.3.4. Capa aplicación.

Esta capa realiza las funciones de las tres capas superiores del modelo OSI: aplicación, sesión y presentación.

Es responsable de la comunicación nodo a nodo y controla las especificaciones de la interfaz de usuario. Algunos de los protocolos presentes son:

- HTTP/S:
- FTP / TFTP:
- SMTP (envío) /POP/IMAP (recibir):
- BOOTP/DHCP:
- DNS:
- SSH (conexión máquina remoto).
- SSL y TLS.






















