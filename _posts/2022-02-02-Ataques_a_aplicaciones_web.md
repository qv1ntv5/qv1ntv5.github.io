---
layout: post
title: Ataque a aplicaciones web.
subtitle: Introducción a estructuras y ataques de aplicaciones web.
tags: [redteam]
---

# 1. Introducción a las aplicaciones web, dirb y burpsuite I.

## 1.1. Fundamentos de aplicaciones web.

### 1.1.1. Esquema de protocolos de comunicación de servidores web.

Recordamos que en internet las relaciones entre servidores y clientes existen peticiones y respuestas, estas peticiones y respuestas se concretan mediante un protocolo de comunicaciones entre el cliente y el servidor, el cual está descrito a continuación:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214095352.png' | relative_url }}" text-align="center"/>
</div>

- En primer lugar un cliente, usuario, quiere pedir algo a una página web, de forma que el cliente lleva a cabo un HTTP request al servidor web. 

- Una vez la request llega al servidor, este reaccionará de dos formas distintas dependiendo de si se trata de un servidor estático o un servidor dinámico. Un servidor estático es un servidor que siempre devuelve la misma información ante la misma request, un servidor dinámico es un servidor que, además de la petición, procesa la información que tiene del cliente (cookies, datos del navegador...) que ha pedido la request para generar una respuesta personalizada, por ejemplo youtube o similar. Así, el camino que sigue el servidor se bifurca:
 
	- **Estático**: Si el servidor es estático, se realiza una petición (get o similar) a un directorio con todos los ficheros html a los que la página tiene acceso, este fichero vuelve al servidor y el servidor devuelve una respuesta http al usuario con el contenido html en cuestión (que será igual .

	- **Dinámico**: Si el servidor es dinámico se envía la petición y los datos del peticionario a una aplicacion servidor que gestiona los datos para ofrecer una variante personalizada a las características del usuario. Esta información se devuelve al servidor web que devuelve al cliente una respuesta http. 

<br />

###  1.1.2. Componentes básicos del servidor web.

Dentro de la relación existente entre clientes y servidores nos encontramos con el papel del servidor web, se trata de un software que corre la máquina que tiene el papel de servidor y monta, configura y mantiene la página web a través de la cual el cliente lleva a cabo un request.

El servidor web contiene los siguientes componentes:

- **Documento Root**:

	El documento root es un directorio del servidor web que almacena ficheros html importantes relacionados con las páginas web de un dominio que servirán como respuesta a las peticiones del cliente. Está en /var/www/ y puede haber varios carpetas indexadas dentro.
	
	Puede contiener varios roots file, en función del número de dominios que gestione el servidor web.

	<br />
	
- **Servidor Root**:

	Es el directorio top-level en el que se encuentra toda la configuración, errores, ejecutables, y logs de la página web. Consiste en el código que implementa el servidor a la hora de construir la página web.

	En general consiste de cuatro ficheros donde uno de ellos consiste en el código a implementar y los tres restantes, -conf, -logs y -cgi-bin se usan para almacenar inforamación de configuración, almacenar logs y ejecutables respectivamente.

	Server root se encuentra dentro de la carpeta /etc en linux.
	
	<br />
	
- **Arbol de documentos virtual**:

	Este elemento provee almacenamiento en diferentes máquinas o discos después de que el disco original se llene. Así, aunque haya un fichero html que no se encuentren dentro del document root el path de búsqueda seguirá siendo válido aunque el html estuviera en otro lado gracias a esta herramienta.
	
	<br />
	
- **Virtual hosting**:

	Recordamos que la técnica del hosting es la técnica que permite alojamiento para sitios web, el hosting web en concreto aloja contenidos de tu web y tu correo electrónico para que puedan ser vistos en cualquier dispositivo conectado a internet.

	Ahora que entendemos el hosting es la técnica que permite hacer hosting sobre múltiples dominios o sitios web que se encuentran dentro del mismo servidor. Esto permite compartir los mismos recursos entre varios servidores
	
	<br />
	
- **Proxy**:
	
	Un servidor proxy es un software ejecutado desde una máquina externa al servidor y al cliente que actúa de forma que el cliente envía peticiones al proxy y si esta las acepta entonces son redirigidas al servidor. Los proxys almacenan no sólo la request, sino información general que permite identificar la máquina del peticionario  cierto nivel para poder evaluar si la conexión por parte de la misma es o no deseable ajustándose a ciertos criterios establecidos por la persona que haya configurado.
	
	Es una forma de protegerte a ti como cliente de contenido indeseado o de proteger al servidor web de conexiones indeseadas.

	<br />
	
- **Arquitectura de servidor web de código abierto**.

  Un servidor web es un software que corre sobre la máquina servidor y está constituido de diversos componentes a nivel de programa:
  
	- **Systema operativo sobre el que se ejecuta**: Generalemente Linux.

	- **Motor de servidor web**: usualmente Apache.

	- **Motor de base de datos**: como MySQL es una base de datos.

	- **Lenguaje de programación para la página web**: PHP, Java, etc es el lenguaje de programación que utiliza el servidor web para montar y gestionar la aplicación web. 

  
Una estructura predefinida de código abierto es LAMP (Linux, Apache, MySQL, Php). También otras comop XAMP que no tienen tanta calidad.

Para Windows existe una aplicación servidor web desarrollado por Microsoft, llamado IIS, se trata de un servidor flexible, seguro y fácil de manejar

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214103426.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 1.3. Ataques a páginas web.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214103537.png' | relative_url }}" text-align="center"/>
</div>

Consiste en generr una cantaidad de tráfico elevado de forma que colapsamos el servidor y su capacidad para atender a usuarios legítimos es mínima o nula.

La diferencia entre DoS y DDoS es el número de dispositivos que esten efectuando el ataque, puede ser una sola máquina o una botnet.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214104139.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214104450.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214104459.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214104610.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214104747.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214104816.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214104857.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 1.4. Herramientas de kali.

### 1.4.1. Dirb.

#### 1.4.1.1. Introducción.

Es un escaner de contenido de la aplicación web. Se utiliza con la url base o el dominio si queremos hacer una búsqueda generalizada. Y opciones un fichero con el nombre que queremos buscar (diccionario). Si el diccionario es largo tardará porque este comando hace una prueba por cada palabra del diccionario.

DIRB es un escáner de contenido web. Busca objetos web existentes (y / u ocultos). Básicamente, funciona lanzando un ataque basado en diccionario contra un servidor web y analizando las respuestas.

DIRB viene con un conjunto de listas de palabras de ataque preconfiguradas para un uso fácil en el directorio: /usr/share/wordlist/dirb, pero puede usar sus listas de palabras personalizadas además.

El objetivo principal de DIRB es ayudar en la auditoría profesional de aplicaciones web. Especialmente en pruebas relacionadas con la seguridad. Cubre algunos agujeros no cubiertos por los escáneres de vulnerabilidades web clásicos. DIRB busca objetos web específicos que otros escáneres CGI genéricos no pueden buscar. No busca vulnerabilidades ni busca contenidos web que puedan ser vulnerables.

<br />

#### 1.4.1.2. 	Uso de la herramienta.

Esta herramienta como sigue:

```bash
dirb <dominio/url> [<ficherodiccionario>] [<options>]
```

Es decir, se le pasa un dominio de un sitio web para hacer una búsqueda general o un url para hacer, dentro de un sitio web, una búsqueda más específica. Opcionalmente se le puede añadir o bien opciones o bien un fichero con palabras.

Para explorar más opciones consultar man. Para llevar a cabo el mapeado de algunas secciones web es necesario tener proviligios y acreditarlos mediante las cookies o usuario y contraseña


<br />

### 1.4.2. DirBuster.

Se trata un comando que funciona de manera similar al anterior, se el pasa una url y un fichero con nombres. Sin embargo tiene una interzaf gráfica que lo puede hacer más manejable.

Dentro de /usr/share/dirbuster/wordlist hay listas de ficheros con palabras que se pueden utilizan para dirbuster y para dirb. Ambos funcionan de la misma manera en cuanto al uso del diccionario.

<br />

### 1.4.3. Dymerge.

Herramienta para combinar diccionarios.

<br />

### 1.4.4. Nikto.

Nikto es un escaner de vulnerabilidades Open Source o de fuente abierta, el cual está escrito en el lenguaje Perl específico para sitios web.

Nikto proporciona la capacidad de escanear servidores web en busca de vulnerabilidades.

Nikto se ejecuta como una línea de comando desde la terminal:

```bash
$ nikto -h <IP_Web> -p <PORT>
```

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215211546.png' | relative_url }}" text-align="center"/>
</div>

Aquí podemos ver, analizando la máquina Metasplotaible, que lleva a cabo una evaluación de todos los servicios que encuentra dentro de la máquina y las vulnerabilidades conocidas que detecta en dichos servicios.

<br />

## 1.5. Mapeo de un sitio web con BurpSuite.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211214124736.png' | relative_url }}" text-align="center"/>
</div>

Para mapear un sitio web con BurpSuite de un sitio web basta con interceptar una petición con el proxy y liberar la petición. Burpsuite hará un mapeo automático de los sitios web vinculados a la petición y alguna carpeta adyacente que se puede visualizar en la sección: **Tarjet > Site map**. 

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220108230012.png' | relative_url }}" text-align="center"/>
</div>

<br />

# 2. Burpsuite.

## 2.1. Concepto de Burpsuite.

BurpSuite es una herramienta pentesting web. Contiene los siguientes compoentes claves:

-   Una araña o spider con reconocimiento de aplicaciones, para el rastreo de recursos de la aplicación.

-   Un escáner de aplicaciones web avanzadas para automatizar la detección de numerosos tipos de vulnerabilidad (esta función esta disponible comprando la licencia del software) y llevar a cabo un mapeo de la aplicación web.

-   Un **Proxy intercepta**, lo que le permite inspeccionar y modificar el tráfico entre el navegador y la aplicación de destino.

-   Una **herramienta de intrusión**, para realizar poderosos ataques personalizados de encontrar y explotar vulnerabilidades inusuales.

-   Una **herramienta de repetidor**, para manipular y volver a enviar peticiones individuales.

-   Una herramienta secuenciador, para probar la aleatoriedad de las credenciales de sesión.

-   Capacidad de guardar su trabajo y reanudar el trabajo más tarde.

-   Extensibilidad, lo que le permite escribir fácilmente tus propios plugins, para realizar tareas complejas y personalizadas muy dentro de Burp.

<br />

## 2.2. Proxy.

Un **proxy** es una herramienta de intercepción que se coloca entre un cliente y un servidor e intercepta las peticiones que el cliente manda al servidor. Burpsuite dispone de un proxy integrado con el que podemos interceptar las peticiones que mandamos a un servidor para evaluarlas.

<br />

### 2.2.2. Configuración del proxy.

En primer lugar, lo configuramos en BurpSuite:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211217164204.png' | relative_url }}" text-align="center"/>
</div>

Y seguidamente, en el navegador Firefox, en Edit > Preferences a bajo del todo en Network settings:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104164646.png' | relative_url }}" text-align="center"/>
</div>



<br />

### 2.2.3. Interceptando peticiones.

A continuación, dentro de Proxy, en 'Intercept is on'. 

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104163352.png' | relative_url }}" text-align="center"/>
</div>


A partir de aquí cualquier petición que realizemos será interceptada por nuestro proxy y mostrada a través de la interfaz de la aplicación burpsuite.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104164753.png' | relative_url }}" text-align="center"/>
</div>


Y la página permanecerá en espera a que la request sea liberada del proxy de BurpSuite y redirigida cambiando a 'Intercep is off'. Mientras la request se encuentre en nuestro proxy esta puede ser modificada a conveniencia y al liberarla, se libera modificada.

También se puede interceptar la respuesta que te devuelve el servidor en options e "Interceptar respuesta del servidor".

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104164827.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 2.3. Intruder.

### 2.3.1. Intruder como herramienta.

La herramienta **Intruder** de Burp es una herramienta para automatizar ataques personalizados contra aplicaciones web. Se puede emplear para la realización de una amplia gama de tareas, desde un ataque brute forcing hasta la explotación activa de vulnerabilidades SQLi.

<br />

### 2.3.2. Cómo funciona Intruder

Burpsuite funciona tomando una solicitud HTTP base y modifica la solicitud de varias maneras sistémicas (modificando parámetros, etc) y emite cada versión modificada de la solicitud e intercepta las respuestas de la aplicación para mostrarnosla e informando de detalles interesantes.

Para cada ataque, debe especificar uno o más conjuntos de cargas útiles (payloads; es decir, las partes del código que van a amrcar la diferencia entre solicitud y solicitud y que añadirán la maliciosidad a las request) las posiciones en la solicitud base donde se colocarán las cargas útiles. 

Intruder contiene las siguientes pestañas. Una vez hemos interceptado la solicitud pase con el proxy y la hemos redirigido al intruder con las opciones que se desplegan al pulsar el botón derecho sobre la solicitud:

<br />

![[Pasted image 20220104172026.png]]

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104164827.png' | relative_url }}" text-align="center"/>
</div>

<br />

Vamos a la etiqueta de intruder y encontramos varias pestañas: Target, Positions, Payloads, Options:

- **Target**: En esta pestaña se configura la dirección de la máquina a la que va diriga la request:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104172320.png' | relative_url }}" text-align="center"/>
	</div>
	
	Por defecto, al reenviar la petición desde el proxy siempre vendrá la dirección de destino por defecto y no hace falta tocar nada.
	
	<br />
	
- **Positions**: El apartado de positions define los lugares de la request que se van a alterar y el tipo de ataque que queremos hacer. Estos fragmentos de códigos pueden ser identifcados porque se encuentran entre signos '§' envolviendo al mismo.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104164827.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
	Entre los tipos de ataques se encuentran:
	
	- **Sniper**: Este utiliza un solo conjunto de payloads. (Esto lo entenderemos más adelante en la pestaña de payloads ya que una cosa es la posición de código a alterar con payloads y otra es el número de conjuntos de payloads que se utilizan). 
	
	- **Battering ram**: Este tipo de ataque, al igual que el «sniper», usa un solo payload. Sin embargo, a diferencia del «sniper», éste sí coloca el dato a insertar en todas las posiciones a la vez. Este ataque es útil, por lo tanto, cuando se requiere que se inserte la misma entrada en varios lugares dentro de la petición.
	
	- **Pitchfork**: En este tipo de ataque definimos varios conjuntos de payloadas, uno para posición que se define (hasta un máximo de 20 posiciones). El ataque varia simultáneamente todos las unidades de payloads de cada posición. Es decir, en la primera request enviará el primer payload del conjunto asociado a cada posición en cada posición, en la segunda request el segundo payload del conjunto asociada a cada posición en cada posición y así sucesivamente de forma que el número total de request que se envían coíncide con el número de elementos que posee el conjunto de payloads más pequeño asociado a una posición.
	
	- **Cluster Bomb**: Este también utiliza varios conjuntos de payloads. Pero rota los conjuntos de payloads sobre las posiciones en cada request. Eñ número final de request es el número de payloads en todos los conjuntos de payloads definidos.
	
	<br />
	
- **Payload**: En la pestaña payload definimos uno a uno como son los conjuntos de payloads que se van a enviar en la lista anterior. Podemos elegir entre varios tipos de payloads:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104211354.png' | relative_url }}" text-align="center"/>
	</div>

	
	Las opciones posteriores de la pestaña cambiarán en función  del tipo de payload que elijamos y este a su vez dependerá del tipo de ataque que queramos hacer.
	
	Desde este misma pestaña podemos dar comienzo al ataque desde Start Attack arriba a la derecha de la misma pestaña.
	
- **Options**: Distintas opciones acerca de la configuración del ataque y del tipo de request que hacemos.

<br />

### 2.3.3. Iniciado el ataque...

Mandará una serie de request y devolverá la respuesta del servidor. La forma que tendrá esta depende del tipo de ataque que hagamos.

<br />

## 2.4. Repeater.

El **Repeater** es una herramienta sencilla para manipular y volver a emitir manualmente mensajes HTTP y WebSocket individuales y analizar las respuestas de la aplicación.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220108231116.png' | relative_url }}" text-align="center"/>
</div>

Aquí podemos modificar una petición, enviarla y analizar la respuesta del servidor e incluso renderizar el documento HTML respuesta para ver la página como si la tuviéramos abierta en el navegador.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220108231513.png' | relative_url }}" text-align="center"/>
</div>

<br />

# 3. Fundamentos de las aplicaciones Web.

## 3.1. Arquitectura Web.

### 3.1.1. Internet y Web.

Un sitio web es un sistema que consta de diversos componentes interrelacionados compuestos por las siguientes partes.

Por un lado, **Internet** hace referencia a la red física que conecta diferentes dispostivos. Por otro lado, **Web** hace referencia a la arquitectura lógica que sigue la información y que se desarrolla sobre esa red física.

<br />

### 3.1.2. Funcionamiento de la web.

Para que la **Web** funcione se requiere de una red que provea la funcionalidad que permita que cualquier dispositivo conectado a **Internet** pueda conectarse a un servidor identificado por la URL (Localizador Uniforme de Recursos).

Esta funcionalidad está proveeida por tres partes; el ISP (Proveedor de Servicios de Internet), otra la provee tu dispositivo y otra el servidor web de destino:

1. **Traducción del nombre de dominio a dirección IP (DNS)** del servidor; servicio proveido por ISP. 

2. **Conexión y Transporte (Socket)**; Una vez tenemos la dirección IP del servidor implementamos una conexión con él mediante unas pautas de acción o mecanismo que establecen una conexión segura entre los dos puntos (Socket); que es la parte más compleja del sistema porque implementa un protocolo de corrección de errores que permite transmitir información sobre una **Internet** que pierde datos, que los desordena y a veces incluso los duplica. La inteligencia del socket (cómo se ordena el socket) radica sólo en los extremos de la conexión: el navegador y el servidor. El resto de la red no interviene en este servicio, y eso es fundamental para mantener a Internet como un servicio barato y eficiente.

3. **Enrutamiento de paquetes IP**; El enrutamiento es el proceso durante el cual los paquetes de datos se envían de una máquina o dispositivo (técnicamente denominado nodo) a otra en una red hasta que llegan a su destino. Un servicio básico que debe de prover el ISP.

4. **Protocolo HTTP**; **Gobierna el dialogo que se establece entre el navegador y el servicio web una vez que están conectados**. El protocolo permite intercambiar conetenidos de todo tipo, como texto, imágenes, audio, vídeo, etc. 

En resumidas cuentas; el navegador envía una URL al servidor, quien le responde con el contenido almacenado para esa URL de manera que el navegador lo interprete y decida qué hacer con éste.

<br />

## 3.2. Dominios de Internet y DNS.

### 3.2.1. Dominios de Internet.

Un dominio de internet es una red de internet es una red de identificación asociada a un grupo de dispositivos o equipos conectas a la red de internet.

El **propósito principal de los nombres de dominio** en Internet y del sistema de nombres de dominio (**DNS**), es **traducir las direcciones IP** de cada nodo activo en la red, **a términos fáciles de recordar**.

<br />

### 3.2.2. Protocolo DNS.

El Sistema de nombres de dominio (Domain Name System) es el que se encarga de vincular a cada dominio con una dirección IP. 

El **DNS** no es más que una **base de datos distribuida** es decir, hay multitud de bases de datos en todo el mundo que son replicas unas de otras.

Es sistema funciona de la siguiente manera:

-   Cuando tu escribes el dominio en el navegador, lo primero que hace es comprobar si lo tienes en la caché de tu navegador.

-   Si se trata de una página web que visitas a menudo probablemente esté, de lo contrario se dirige a un servidor DNS donde va a buscar ese nombre.


-   Cuando se localiza el nombre se comprueba qué IP tiene ese dominio y te redirige al servidor configurado con esa IP.

Es común en los sitios web cambiar de servidor por motivos de escalabilidad o por motivos económicos. En estos casos lo que ocurre es que se notifica a los DNS que el dominio ejemplo.com ha cambiado de IP y en cuestión de horas se actualizan todas las bases de datos y cuando se vuelva a solicitar visitar la página web, se redirigirá al nuevo servidor.


<br />

## 3.3. Repartición de la Aplicación Web (Desarrollo Web).

### 3.3.1. Cliente y Servidor.

En internet el intercambio de comunicación consiste de peticiones y respuestas. Los dispositivos conectados a la Web se dividen en clientes y servidores. Estos son conceptos que se entienden de forma conjunta dentro de un sistema de dos o más máquinas. El **cliente** consiste en aquella máquina que ejerce el papel de *peticionario* mientras que el **servidor** consiste en aquella máquina que ejerce el papel de *respondiente* a la petición.

En un sentido menos estricto, podemos diferenciar una estructura arquetípica de cada clase:

- **Cliente**: Los clientes son dispositivos de los usuarios conectados y consumidores de Internet.
- **Servidor**: Los servidores son ordenadores que almacenan páginas web, sitios o aplicaciones que ofrecen servicios a los usuarios.

Los datos se envían desde el servidor al cliente en forma de **paquetes**. Esto significa que **los datos se envían a través de la Web como miles de trozos pequeños**, permitiendo que muchos usuarios pueden **descargar la misma página web al mismo tiempo**.

<br />

### 3.3.2. Partes de la web.
El desarrollo Web es lo que se encarga de dar funcionalidad a la misma y está dividida en dos grandes bloques: **Front-end** (La parte de la web que se ejecuta en el cliente y es lo que ve el usuario), **Back-end** (parte de la web que se ejecuta en el servidor).

- **Front-End**: El Front-end o **interfaz de usuario** se encarga de interactuar con el usuario para **recoger datos e información** y es la parte de la aplicación web que se carga en parte del cliente y se muestra a través del **navegador web**. Se apoya en lenguajes como **HTML**, **CSS** y **JavaScript**. Los datos recogidos son transformados para que el Back-end pueda tratarlos de forma adecuada.

- **Back-End**: el Back-end es el encargado de recoger la información captada del usuario a través del Front-end y tratarla. Se ejecuta en el servidor y se utilizan multitud de lenguajes de programación de alto nivel como PHP, Java, C#, Ruby.

	El Back-end (servidor) utiliza la información recibida para ser almacenada en una base de datos, para comprobaciones, para realizar operaciones matemáticas, validaciones, etc…

	Cuando termina el proceso lo más normal es que mande una respuesta al **Front-end** (cliente), generalmente en forma de **HTML**, para que se muestre en el navegador.


<br />

### 3.3.3. Navegador (Cliente Web). 

Un **navegador** es un programa/aplicación software que te muestra el Front-end de una aplicación web. **Te permite interactuar con la aplicación web** recogiendo datos y acciones para ser transmitidas al Back-end. Un ejemplo es un formulario donde rellenamos los campos y estos son enviados cuando realizamos la acción de pulsar el botón enviar.

**El navegador es capaz  interpretar el código HTML, CSS y JavaScript** para mostrarnos una versión gráfica del código.

Si miramos una página web de un sitio y, en cualquier navegador, pulsamos _botón derecho -> ver código fuente_, podremos ver qué es un HTML e incluso el CSS y JavaScript.


<br />

### 3.3.4. Servidor web.

**La principal función de un servidor Web es almacenar los archivos de un sitio** (HTML, CSS, JavaScript, multimedia, etc.) **y emitirlos por Internet** para poder ser visitado por los usuarios. También **se encarga de interpretar los lenguajes de programación que se utilizan en el Back-end**.

No deja de ser un **software** instalado en un ordenador **que procesa una aplicación del lado del servidor**, realizando conexiones con el cliente y **generando una respuesta** en cualquier lenguaje o aplicación del lado del cliente. Este ordenador tiene unas características especiales que lo hacen fiable, seguro, con gran capacidad y potente.

**Cada servidor tiene asignada una dirección IP** para que pueda ser fácilmente identificado por los servidores de DNS.

Normalmente este servicio lo ofrecen las compañías de Hosting, que a cambio de una tarifa mensual o anual te ofrecen un servicio para alojar tus sitios web en sus servidores.

Sin los servidores Web, la Internet tal como la conocemos no existiría. **Los servidores son el depositario de todo el contenido que existe en internet**.

<br />

### 3.3.5. Protocolo HTTP. Comunicación en la red.

HTTP (Protocolo de Transferencia de HiperTexto) es el principal protocolo de transferencia en Internet. Son las normas y reglas que permiten la transferencia de paquetes, se trata del protocolo a través del cual se transmiten los archivos HTML entre el cliente y el servidor.


<br />

## 3.4. ¿Qué sucede exactamente?

Cuando escribes una dirección web en el navegador:

1.  El navegador va al servidor DNS y encuentra la dirección real del servidor en el que el sitio web está alojado.
2.  El navegador envía un mensaje de petición HTTP al servidor, pidiéndole que envíe una copia de la página web para el cliente. Este mensaje y todos los datos enviados entre el cliente y el servidor, se envían a través de tu conexión a Internet usando el protocolo TCP/IP.
3.  Siempre que el servidor apruebe la solicitud del cliente, el servidor enviará al cliente un mensaje indicando que todo es correcto y comenzará a enviar los archivos de la página web al navegador como una serie de pequeños trozos llamados paquetes de datos.
4.  El navegador reúne los pequeños trozos, forma un sitio web completo (Front-end) y te lo muestra para que puedas interactuar con él mandando de vuelta la información al back-end del servidor generando así un bucle de intercambio de información.

<br />

# 4. Autenticación web

## 4.1. Métodos de autenticación.

### 4.1.1. Concepto.

La **autenticación web** es el proceso de seguridad que permite a usuarios verificarsus identidades con la finalidad de ganar acceso a información que no está expuesta públicamente.

<br />

### 4.1.2. Tipos de autenticación.

De entre los métodos que podemos utilizar para llevar a cabo procesos de autenticación estan:

1. **Autenticación HTTP básica**: Esta es una forma de autenticación que requiere que un cliente proporcione un nombre de usuario y una contraseña al realizar una solicitud.

	El cliente debe enviar el encabezado de autorización junto con cada solicitud que realice. Estas credenciales se guardan dentro del servidor de la siguiente manera: 

	-   El nombre de usuario y la contraseña se concatenan en una sola cadena: "username:password".
	-   esta cadena está codificada con Base64
	-   la "Basic" palabra clave se coloca antes de este valor codificado

	Así por ejemplo: 
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215114150.png' | relative_url }}" text-align="center"/>
	</div>

	Sin embargo, este proceso de autenticación tiene una serie de desventajas como:
	
	- El nombre de usuario y la contraseña se envían en cada solicitud con lo que se exponen potencialmente incluso aunque se manden através de una conexión seguridad.
	
	- Si el sitio al que se conecta posee un cifrado débil un atacante podría romperlo y de esa forma, las credenciales se expondrían de inmediato.
	
	- No hay forma de cerrar la sesión del usuario usando la autenticación básica.
	
	- La caducidad de las credenciales no es trivial; debe pedirle al usuario que cambie la contraseña para hacerlo.


	Para conocer la fortaleza de una contraseña podemos utilizar la página web: https://www.passwortcheck.ch/passwortcheck/passwortcheck
	
	<br />
	
2. **Cookies**: Las "cookies" son ficheros de texto con pequeñas piezas de datos que identifican a tu máquina.  Así, cuando un servidor recibe una solicitud HTTP en la request puede enviar un encabezado "Set-Cookie" para acreditarse y poder acceder sin necesidad de introducir credenciales. En esencia, las cookies son variables que contienen valores que definen una serie de datos sobre la entidad que las posee. Originalmente un servidor entrega cookies a un usuario que ha visitado la aplicación web en función del nivel de privilegios que el usuario a demostrado tener (con un logeo o algo por el estilo). De esta manera, las cookies se guardan en el navegador y las siguientes veces que el usuario visite dicha página no será necesario que se logee pues con mostrar las cookies será suficiente para que el servidor pueda identificar el tipo de usuario que es respecto de la página.

	En esencia, 

	

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215164005.png' | relative_url }}" text-align="center"/>
	</div>

	Es necesario puntualizar una serie de cosas:
	
	- Intentar utilizar siempre HttpOnly, así se evitará que aparezca document.cookies, que se trata de un documento que actúa a modo de captador y configurador de los valores realies de las cookies y que las expone potencialmente.
	 
	- Utilizar siempre cookies firmadas, de esta forma un servidor puede saber si una cookie ha sido modificada.

	Sin embargo, existen contras; con las cookies nos arriesgamos a tener un ataque CSFR (Cross Site Request Forgery), el cual fuerza al navegador web de una víctima, validado en algún servicio a enviar una petición a una aplicación web vulnerable, la cual procesa la petición en nombre de la víctima.

<br />

3. **Tokens**: En primer lugar, un **token** es un objeto informático (ya sea software o hardware) que representa el derecho a realizar una acción. Existen varios tipos de tokens: Session Token, Security Token, Access token... De manera que una aplicación que posea un token determinado tiene el derecho (sin introducir ningún tipo de credencial) de llevar a cabo una determinada acción dentro de un dispositivo.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215180957.png' | relative_url }}" text-align="center"/>
	</div>
	
	Para el caso que nos ocupa nos interesan los JWT (JSON Web Token) que es un token tipo JSON (lenguaje de programación web derivado de JavaScript) utilizado por y entre aplicaciones web. Contiene las siguientes partes:
	
	- Encabezado que contiene el tipo de token y hash (firma que garantiza la integridad del token).
	- Carga útil.
	- Firma.
	
	La ventaja es que no se introduce ni por parte del usuario ni contenido en la propia petición ningún tipo de credencial, sin embargo estos tokens necesariamente deben de almacenarse localmente lo cual hace que se tenga que hacer un esfuerzo adicional para protegerse de ataques XSS, (Cross Site Scripting).
	
	
- **Firmas**: Ya sea utilizando cookies o tokens, ambos son objetos que van desde el navegador al servidor y viceversa. Esto los expone a una posible interception por parte de un tercero. La mejor forma de prevenir esto es firmar el objeto mediante un hash, de manera que se pueda comprobar la integridad del documento. 

- **Contraseñas de un sólo uso**: 

<br />

## 4.2. Herramientas para ataque de fuerza bruta.

### 4.2.1. Introducción. Ataque de fuerza bruta.

Hasta ahora hemos visto métodos de autenticación de páginas web, formas de ganar acceso a documentación no expuesta al público de dicha web. Entre los diversos ataques que se pueden hacer para poder obtener acceso a dichos documentos tenemos el ataque de fuerza bruta, que sirve específicamente para la autenticación con credenciales y algunos tipos de cookies. 

Este tipo de ataque consiste en probar una multitud de combinaciones de usuarios y contraseñas probables para hacerse con las credenciales auténticas a base de ensayo y error sobre una web concreta. Es especialmente eficaz en los casos en los que se sabe con certeza que las credenciales tienen baja fortaleza (contraseña / usuario pequeño, ausencia de mayúsculas, dígitos, caracteres especiales, palabras coherentes, etc).

<br />

### 4.2.2. Comandos de creación de diccionarios.

Las herramientas con las que vamos a trabajar necesitan de una lista de "strings" que pasar como usuario y contraseña a la web a la que estamos intentando ingresar y evidentemente nos conviene utilizar listas de palabras confeccionadas de forma personalizada basada en información que poseemos sobre la víctima sobre la que estamos preparando el ataque. Para ello tenemos como ayuda los siguientes comandos en Kali.

- **crunch**: crunch es un comando que opera con caracteres para formar palabras atendiendo a unas condiciones que se pueden especificar mediante modificadores/opciones especiales. Viene preinstalado por defecto con kali y la sintaxis básica es:

	```bash
	crunch <min-len> <max-len> [<charset string>] [options]
	```
	
	Así, crunch nos generará palabras entre un mínimo y un máximo de caracteres basándose en los caracteres/dígitos proporcionados y las opciones que le queramos pasar.
	
	
- **cewl**: El comando **cewl** lleva a cabo una lista de strings basadas en los datos del documento html de esa página web que se obtiene de la URL de la misma. Posteriormente pueden ser utilizados como posibles nombres de usuarios o contraseñas.

- **cupp**: El comando **cupp** es un comando que genera una lista de nombres que tienen probabilidad de ser usuario o contraseña de un individuo atendiendo a sus datos personales (nombre, apellido, edad, nickname, pareja, etc). También viene instalado en Kali por defecto.
	
	La lógica de este comando reside en que un individuo poco preparado para el mundo de la informática tendrá una contraseña débil basada en elementos fáciles de recordar y que estén asociados al mismo o a sus hábitos.

	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215184643.png' | relative_url }}" text-align="center"/>
	</div>
	
	Como se puede ver en la imágen, hemos introducido "cupp -i" en kali, nos ha pedido una serie de datos en relación a la víctima y ha contruido un fichero en el que ha guardado las palabras que ha contruido.
	
- **Dymerge**: Dymerge es un comando que construye una lista a apartir de varias listas de palabras distintas. Es una forma de agrupar diccionarios para ejecutar el ataque en menos pasos. No está disponible desde la apt, hay que instalarlo desde Github:

	Instalamos DyMerge clonando el repositorio desde la página: https://github.com/k4m4/dymerge.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215104224.png' | relative_url }}" text-align="center"/>
	</div>

	Una herramienta simple pero poderosa, escrita exclusivamente en Python, que toma listas de palabras dadas y las fusiona en un diccionario dinámico que luego se puede usar como munición para un ataque exitoso basado en diccionario (o de fuerza bruta).

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215180957.png' | relative_url }}" text-align="center"/>
	</div>

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211215110615.png' | relative_url }}" text-align="center"/>
	</div>

<br />

### 4.2.2. Brute forcing con Burp Suite e Hydra.

El ataque bruteforcing (fuerza bruta) consiste en utilizar una aplicación de repetición de request para llevar a cabo múltiples request de logeo en bateria para ver si encontramos combinaciones de palabras que sean usuario y contraseña.

Esto es especialmente útil cuando la fortaleza de la contraseña es débil, esta basada palabras de diccionario o en conceptos relacionados a la persona, como su edad, su nombre, etc. Todos datos accesibles mediante un buen OSINT y la utilización de las herramientas descritas más arriba.

Una vez tenemos una lista de palabras que se ajusta a nuestra víctima, tenemos que utilizar una herramienta que se encarge de preparar un ataque basado en diccionario y en la web. Para ello disponemos de Hydra y BurpSuite.

<br />

#### 4.2.2.1. Hydra.

Hydra es una herramienta de ataque de fuerza bruta con diccionarios integrada como comando de linux disponible ne la APT. Esta herramienta realiza una serie de request en cadena cambiando una serie de parámetros basándose en una lista que se le proporciona mediante modificadores permitiéndo especificar protocolo y método de request.

Para entender cómo funciona sería conveniente ver un ejemplo a través de los siguientes pasos:

- Iniciamos la máquina OWASP para tener activo la aplicación web DVWA. 

	<br />

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220103230510.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- En ella vamos al sitio de logeo tipo POST.
- Vamos a llevar a cabo un ataque tipo fuerza bruta para intentar obtener las credenciales de logeo mediante una HTTP request con hydra.

<br />

Seguimos los siguientes pasos: 

1. Primero confeccionamos una lista de nombres que pueden ser probables usuarios y contraseñas. Esto puede hacerse mediante herramientas de kali como cewl, crunch o cupp pero por el bien de la simplicidad operaremos con una lista más sencilla separada en users.txt y passwords.txt 

<br />

2. Además, necesitamos el protocolo y el tipo de request para que hydra lleve a cabo el ataque. El tipo de request al ser una página http será evidentemente http, pero ¿qué método?. Obviamente, al tener que acreditarse mediante el relleno de campos dentro de la página se tratará necesariamente de un POST, pero para salir de dudas podemos comprobar que tipo de request se mandan utilizando DevTools e inspeccionando el apartado de NETWORK, mandamos un logeo de la página y vemos qué request se ha enviado, en nuestro caso un POST.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220103234618.png' | relative_url }}" text-align="center"/>
	</div>
	
	Además de mostrarnos qué tipo de request es podemos ver también la forma y los parámetros que se envían en la petición, el payload, abajo a la derecha.
	
<br />

3. Una vez tenemos los ficheros diccionario construimos el comando. Este comando necesita de los siguientes elementos para poder llevar a cabo el ataque de forma satisfactoria:

	- Fichero de usuario tras el modificador -L (o -l si sólo es un nombre).
	- Fichero de contraseñas tras el modificador -P (o -p si sólo es un nombre).

	Seguidamente pasámos la ip de destino y el método de request que será http-post-form.

	A continuación y por último se añaden tres cadenas de caracteres entre comillas separadas por punto y coma que tienen el siguiente propósito:
	
	- **/dvwa/login.php** -> Saber en qué parte de la estructura de ficheros de la ip se encuentra el sitio en el que preparar el ataque.
	
	- **username=^USER^&password=^PASS^&Login=Login** -> Le mandamos la estrucutra de parámetros que debe de ir cambiando en cada request a partir de los diccionarios proporcionados. Los parámetros se indican mediante las palabras que van entre ^. Esta información está basada en el Request Payload de la imágen del apartado anterior.
	
	- **Login failed**. Este es el mensaje que aparece cuando el logeo resulta fallido y ayuda a hydra a identificar cuándo una petición de logeo no es válida.

<br />

En definitiva, todos estos datos ayudan a hydra a entender cómo es la estructura de la web y así optimizar su ataque. El comando completo es:
	
<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104000119.png' | relative_url }}" text-align="center"/>
</div>

y el resultado que nos da es:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104000215.png' | relative_url }}" text-align="center"/>
</div>

Informándonos  de que ha encontrado unas credenciales válidas **admin:admin**.

<br />

#### 4.2.2.2. BurpSuite.

Para llevar a cabo el ataque de fuerza bruta con burpsuite. Vamos a hacer uso de tres herramientas de esta aplicación. 

En primer lugar vamos a la DVWA y la abrimos y nos saldrá la parte de logeo. Vamos a descubrir la contraseña haciendo un ataque de fuerza bruta con diccionario.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220108234323.png' | relative_url }}" text-align="center"/>
</div>

En primer lugar intentamos llevar a cabo una petición de logeo introduciendo credenciales ficticias en todos los campos e interceptamos la petición con burpsuite.

- **Proxy**: Usamos el proxy para interceptar la petición, analizarla y poder así configurar las requests que se realizará en el ataque de fuerza bruta. Esta petición contiene la información necesaria para efectuar un intento de logeo al servidor. Vamos a enviar esta información al Intruder. 

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220108234754.png' | relative_url }}" text-align="center"/>
	</div>

	<br />
	
- **Intruder**: Una vez con la información en el intruder definimos el tipo de ataque (pitchfork), ajustamos las posiciones de los payloads; que serán el username y el password y el conjunto de los payloads que irán en cada posición; que en este caso será una lista de nombres que habremos construido con las herramientas vistas más arriba:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109001024.png' | relative_url }}" text-align="center"/>
	</div>
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109001045.png' | relative_url }}" text-align="center"/>
	</div>

	En las opciones de los resultados le damos a seguir redirecciones siempre. Esto en este caso es conveniente porque de esta manera si nos logeamos nos redirigirán a otra página y si fallamos nos quedaremos en la misma. Esto alterará el tamaño del documento html recibido por el servidor y será un buen indicio para diferenciar entre un acierto y un fallo, ya que los fallos, la mayoría tendrán un documento html idéntico con el mismo peso. Exploraremos aquellas respuestas que ocupen un tamaño diferente de la media.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109004855.png' | relative_url }}" text-align="center"/>
	</div>
	
	Iniciamos el ataque y obtenemos:
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109005030.png' | relative_url }}" text-align="center"/>
	</div>
	
	Por la razón que hemos comentado antes, vamos a explorar las opciones que tienen un tamaño distinto enviándolas al repetidor mediante el botón derecho.
	
	
	<br />
	
- **Repeater**: Una vez en el repetidor le damos a la etiqueta que indica "seguir redirección" y nos encontramos que, renderizando el documento html:

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109005323.png' | relative_url }}" text-align="center"/>
	</div>
	
	Tenemos acceso.
	
	Su hubiéramos seguido el mismo procedimiento con las otras request se nos hubiera redirigido a la página de logeo.
	
<br />

# 5. Command execution y Webshell

## 5.1. Ejecución de comandos en web.

### 5.1.1. Concepto y características principales.

La ejecución de comandos, "OS Comand Injection" (Operating System Command Injection) o Shell Injection **consiste en una vulnerabilidad de seguridad web que permite a un atacante ejecutar comandos de sistema operativo de forma arbitraria en el servidor que está ejecutando el servidor web**, comprometiendo de esta forma a la página web, a sus datos, e incluso a otras partes de la estructura del servidor.

Esto se lleva a cabo en una página web en la que podemos introducir "input" que luego se procesa en servidor y se puede explotar si y sólo si la página o el servidor son **vulnerables**.

Generalmente la vulnerabilidad es debida a una mala configuración/programación del servidor web o de la aplicación (falta de saneamiento de código,...).

Destacamos los siguientes apartados:

<br />

- **Error-Based**: 
 
	 Las vulnerabilidades de inyección de comandos están basados en errores de configuración de la aplicación en el que se consige que se devuelva un output distinto del predefinido por el programador que desvela información sensible del servidor. Se trata de algo así como una *extensión* del uso de la aplicación por parte del atacante. 
	 
	 Por ejemplo: si tenemos una aplicación que recoge un string del usuario y lo ejecuta con el comando ping:

	```bash	
	$ ping -c3 <String>
	```

	El atacante puede aprovecharse y poner un caracter que concatene comandos en la terminal del servidor, por ejemplo ";pwd":
	
	```bash	
	$ ping -c3 ;pwd
	
	ping: unknown host uid=0(root) gid=0(root) groups=0(root)
	```
	
	La *falta de saneamiento* en el código de la web (web vulnerable) provocará que esta no filtre nuestra petición y pasará dicha línea directamente al servidor que la ejecutará llevando a cabo dos comandos cuyo output nos mostrará por pantalla: ping y pwd, revelando así información sensible de la máquina.
	
	<br />
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211221103307.png' | relative_url }}" text-align="center"/>
	</div>
	
	<br />
	
- **Blind Attacks**: 
 
	 Algunos ataques de inyección de comando son "ciegos", esto es; la web no verbaliza el procesamiento del comando que se ejecuta dentro del servidor. 

	Un mecanismo eficiente para comprobar que se este ejecutando el comando es introducir un "ping" de bucle invertido: 
	
	```bash
	$ & ping -c 10 127.0.0.1 &
	```
	
	e intentar obtener alguna respuesta de la máquina. Si la esta tarda en responder puede que sea porque esté procesando el ping y por ello tarde en devolver la respuesta al request, indicio de que existe tal vulnerabilidad.

	También puede intentar redirigir la salida del comando inyextado a un archivo dentro de la raíz web que luego puede recuperar usando su navegador. Por ejemplo, si una aplicación proporciona recursos estáticos desde la ubicación /var/www/static puede enviar la siguiente entrada:
	
	```bash
	$ & whoami > /var/www/static/whoami.txt &
	```
	
	Se redirige el output al fichero especificado, visible desde el navegador sobre la dirección: https://vulnerable-website.com/whoami
	
	
	<br />

- **Code vs Command Inyection**: 
 
	Es preciso diferenciar este ataque/procedimiento de la **inyección de código** (Code Injection); en este el atacante agrega su propio código que luego será ejecutado por la aplicación. En contraposición la inyección de comandos permite al atacante *extender* la funcionalidad de la aplicación para ejecutar comandos sin necesidad de inyectar código.
	
	<br />

**En esencia, el cometido de un atacante que pretenda realizar este procedimiento consiste en buscar webs vulnerables (mal programadas) con campos que requieran de la introducción de algún tipo de dato que luego vaya a forma parte de alguna línea de código potencialmente ejecutable por el servidor con la finalidad de explotar dicha vulnerabilidad introduciendo comandos a conveniencia.**

<br />

Debido a esto, por razones de seguridad sólo debemos de iniciar un servidor web con privilegios de súper usuario en contadas excepciones. Lo ideal es ejecutarlo desde un usuario de servicio.

<br />

### 5.1.2. Procedimientos de inyección de comandos.

Se pueden usar una variedad de metacaracteres de shell para realizar ataques de inyección de comandos del sistema operativo.

Varios caracteres funcionan como separadores de comandos, lo que permite encadenar los comandos. 


- **Windows/Unix**:

	 - & (background).
	 - && (AND).
	 - \| (PIPE).
	 - \| \| (OR).
	 
	 <br />
	 
- **Unix**:

	- ; (separador de instrucciones)
	- \n o 0x0a (Nueva línea)

	En los sistemas basados en Unix, también puede usar comillas invertidas o el carácter de dólar para realizar la ejecución en línea de un comando inyectado dentro del comando original:

	-   \`comando inyectado\`
	-   $( comando inyectado )

<br />

### 5.1.4. Prevención de inyección de comandos.

Si se considera inevitable llamar a los comandos del sistema operativo con una entrada proporcionada por el usuario, entonces se debe realizar una validación de entrada sólida. Algunos ejemplos de validación efectiva incluyen:

-   Validación con una lista blanca de valores permitidos.
-   Validando que la entrada sea un número.
-   Validando que la entrada contiene solo caracteres alfanuméricos, sin otra sintaxis o espacios en blanco.


<br />

## 5.2. Commix.

El término Commix es una abreviatura para (COMMand Injection eXploiter), se trata de un comando que permite testear aplicaciones web para encontrar bugs, errores o vulnerabilidades relacionadas con ataques de inyección de código analizando y explotando dichas vulnerabilidades.

Usando esta herramienta es muy fácil encontrar y explotar una vulnerabilidad de inyección de código. Commix nos permite explotar estas vulnerabilidades abriendo 

Para poder utilizar commix se necesitan tres parámetros: 

- El primero es la url, que se especifica tras el modificador **-u**.

- El segundo es el "data", o los datos de la request que hacemos a la página cuando buscamos datos **en el tipo POST**, en el GET no es necesario. Estos datos se introducen después del modificador **--data**. El "data" puede encontrarse revisando la solicitud de logeo en el apartado Network en inspeccionar elemento.

- El tercer parámetro son las cookies. Esto permite a Commix configurar la request con las preferencias que nosotros utilizamos al llevar a cabo la request de forma manual, no son imprescindibles en general pero si pueden serlo en casos particulares, por ejemplo; si estamos intentando inyectar comandos en un sitio después de habernos logeado con unas credenciales. En dicho caso necesitaríamos de las cookies para que las solicitudes de commix no queden bloquedas por falta de autenticación.

	Al igual que ocurre con el apartado anterior, las cookies se pueden obtener revisando la request en el apartado network/Cookies.
	
<br />

Un ejemplo de uso sería:

<br />


<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220104152037.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 5.3. WebShell

Tener una shell en la web para ejecutar comandos.

![[Pasted image 20211216121000.png]]

Código para inyectar y crear una shell. 

![[Pasted image 20211216121144.png]]

<br />

# 6. SQL Injection, SQLmap y Burpsuite III

## 6.1. Qué es SQL Injection.

SQL Injectión, también conocido como SQLI es un vector de ataque común que utiliza código SQL malicioso para "backend databases", **esto es; bases de datos a las que se accede indirectamente desde una aplicación externa en lugar de desde la aplicación almacenada dentro de la base de datos o la manipulación directa de los datos a bajo nivel**, para acceder a información/privilegios no autorizados.

Este vector de ataque sirve para cualquier base de datos SQL, los sitios web son los objetivos más frecuentes y la repercusión que pueden llegar a tener es muy grande, comprometiendo listas de usuarios, información sensible, borrado de tablas, etc.

<br />

## 6.2. Tipos de Inyección SQL.

Los ataques SQL típicamente caen bajo tres categorías: In-band SQLi, Inferential SQLi, Out-of-band SQLi. Puedes clasificar los tipos de inyecciones SQL basándote en los métodos utilizados para acceder y recopilar información sobre el backend database.

Todas estas técnicas se ven con un poco más de profundidad en la sección de SQLMap.

<br />

### 6.2.1. In-band SQLi (Classic).

En el ataque en banda (In-band) los atacantes utilizan un canal de comunicación con la base de datos para lanzar sus ataques y obtener resultados de vuelta. Dentro de este método hay dos variantes:

<br />

1. **Error-based SQLi**: Los atacantes llevan a cabo acciones que causan que la base de datos genere mensajes de error. Estos mensajes de error en realidad proporcionan información (filtraciones de error) y esta es aprovechada por los atacantes para recolectar información sobre la estructura de la base de datos.

<br />

2. **Union-based SQLi**: Esta técnica toma ventaja del operador UNION de SQL el cual fusiona varias statement de select generadas por la base de datos para obtener una única respuesta HTTP. Esta respuesta puede contener datos que el atacante puede aprovechar.

	<br />
	
### 6.2.2. Inferential (Blind) SQLi.  
	
El atacante envía *payload*, esto es; mandar más código del necesario, normalmente con un fin poco lícito y provocar un comportamiento distinto de la aplicación, como por ejemplo inyección de comandos o para conseguir más información de la aplicación, está pendiente de la reacción del servidor y de esa forma recopila información sobre su estructura y mecanismos.

Este método se denomina *blind* o ciego porque el resultado de la información no se devuelve al sitio web del atacante, con lo que este no tiene forma de saber cómo evoluciona el proceso. Por esta razón, las únicas pistas que el atacante sigue consiste en las reacciones del servidor y por ello este tipo de ataques son más lentos.

Podemos clasificar estos ataques como:

<br />

1. **Booleanos**: Cuando el atacante introduce una consulta SQL solicitando a la aplicación que devuelva un resultado. Este resultado variará en función de si la consulta es o no cierta, proceso sobre el cual se obtiene información.

<br />

1. **Time-based**: El atacante envía una consulta SQL a la base de datos que hace que la base de datos espere durante un periodo de tiempo antes de reaccionar, periódo de tiempo en el que el atacante puede dilucidar si su consulta es cierta o falsa sin dependender de los datos devueltos por la base de datos.
	
<br />

### 6.2.3. SQLi fuera de banda.

El atacante solo puede llevar a cabo esta forma de ataque cuando ciertas funciones están habilitadas en el servidor de base de datos utilizado por la aplicación web. **Esta forma de ataque se utiliza principalmente como alternativa a las técnicas SQLi inferenciales y en banda.**

SQLi fuera de banda se realiza cuando el atacante no puede usar el mismo canal para lanzar el ataque y recopilar información, o cuando un servidor es demasiado lento o inestable para realizar estas acciones. Estas técnicas cuentan con la capacidad del servidor para crear solicitudes DNS o HTTP para transferir datos a un atacante.

<br />

## 6.3. Ejemplos de SQLi.

Recordamos que el lenguaje SQL es un lenguaje estandarizado que se utiliza para acceder y manipular bases de datos. Una *consultas SQL* consiste en el uso del lenguaje SQL para ejecutar comandos sobre los objetos/datos de una base de datos concreta.

Una consulta SQL desde una página web que pide la introducción de un dato puede tener el siguiente aspecto:

```SQL
SELECT ItemName, ItemDescription
FROM Item
WHERE ItemNumber = "& Request.QueryString("ItemID")"
```

Donde ItemID consiste en el input que provee el usuario.

El request http GET sería: http://www.estore.com/items/items.asp?itemid=x Donde x sería el valor a introducir.

**Es conveniente mencionar que la sintaxis del código de todos los ejemplos siguientes puede cambiar en función del SGBD o de la página web que se esté atacando. Lo importante de los siguientes ejemplos es entender los mecanismos por los cuales se consigue extraer más información de la debida del servidor vulnerable.**

<br />

### 6.3.1. Condiciones siempre ciertas.

Un atacante que desee ejecutar una inyección SQL manipula una consulta SQL estándard para explotar vulnerabilidades de entrada no validadas en una base de datos.

Sobre la consulta anterior por ejemplo, un posible atacante introducidiría algo así: 

```SQL
SELECT ItemName, ItemDescription
FROM Item
WHERE ItemNumber = "" or 1=1 -- "
					 ----------
					  --ItemID
```

Donde el request sería: input http://www.estore.com/items/items.asp?itemid=999or1=1 

Es decir, **Item ID =  " or 1=1 --**

Así pues, lo que hacemos es aprovecharnos de una mala configuración del servidor web (falta de saneamiento) para introducir código que modifique la consulta que se realiza en la base de datos.

En este caso en concreto podemos observar que el código que nosotros introducimos consta de tres partes:

- ": Unas comillas que cerrarán el contenido que va a llevar ItemNumber.
-  or 1 = 1 para introducir un condicional y una tautología.
-  --: Un comentario para librarnos de las comillas " originales.

De esta manera hemos modificado la sentencia que le pasamos al WHERE (recordemos que este es un parámetro que hará que el sistema evalue la sentencia que viene inmediatamente después como verdadera o falsa para filtrar aquellos registros que no interesan).

Así el sistema va evaluando la condición que le hemos pasado para cada registro, pero como nuestra condición siempre es cierta (ya que se trata del operador OR con una tautologia) para todos los registros entonces nos mostrará todos los registros por pantalla. 

<br />

### 6.3.2. Caracteres filtrados incorrectamente.

Los atacantes también pueden sacar partido de caracteres que no se filtran correctamente para alterar comandos:

```SQL
SELECT ItemName, ItemDescription
FROM Items
WHERE ItemNumber = " "; DROP TABLE USERS # "
					 ----------------------
					 		--ItemID
					
```

En la sentencia anterior, el "punto y coma" se utiliza para separar dos instrucciones. De forma que el request sería: input http://www.estore.com/items/items.asp?itemid=999;DROPTABLE

De nuevo el principio es el mismo que en el apartado anterior cerramos la sección del contido de ItemNumber y, aprovechándonos de la falta de saneamiento del servidor web, concatenamos a la anterior la sentencia DROP TABLE USERS y comentamos el resto para evitar una falta de sinstaxis.

Como resultado la tabla entera puede ser borrada. 

<br />

### 6.3.3. UNION SELECT statement.

Recoradmos que el statement UNION une varias consultas con SELECT para recolectar datos de dos tablas diferentes. Así por ejemplo: 

```SQL
SELECT ItemName, ItemDescription
FROM Items
WHERE ItemID = ' ' UNION SELECT Username, Password FROM Users
				 --------------------------------------------
							   --ItemID

```

input http://www.estore.com/items/items.asp?itemid=999UNIONSELECTuser-name,passwordFROMUSERS

Con UNION concatenamos dos sentencias SELECT (**con la salvedad de que debemos asegurarnos de el número de columnas que pedimos con el segundo SELECT es el mismo que pide el primer SELECT**, esto puede averiguarse metiéndo "null" a modo de columna separando cada columna de la siguiente por una coma y encontraremos el número de columnas correcto en el momento en el que nos deje de salir error de que SELECT no tiene el mismo número de columnas. Si se trata de un ataque ciego podemos añadir a modo de columna el término: SLEEP(10), sabremos entonces que hemos introducido el número de columnas válido cuando el navegador tarde 10 segundos en responder, será indicio de que nuestro código se ha ejecutado y de que no ha habido ningún error).

```SQL
SELECT ItemName, ItemDescription
FROM Items
WHERE ItemID = ' ' UNION SELECT null, SLEEP(10) FROM Users;
				 ------------------------------------------
				   				--ItemID
```

(en este caso tenemos 2 columnas pero si hubiéramos tenido más hubiéramos tenido que meter más nulls hasta que se ejecute la consulta).

<br />

## 6.4. SQLMap.

### 6.4.1. Instalando SQLMap. Funcionalidades básicas.

SQLMap es un herramienta de penetración de código abierto que automatiza el proceso de detectar y explotar fallas o vulnerabilidades de SQLi y la toma de control de los servidores de bases de datos. 

Esta se encarga de realizar peticiones a los parámetros de una URL que se le indiquen, ya sea mediante una petición GET, un POST, en las cookies, etc. Es capaz de explotar cualquier vulnerabilidad SQLi.

Las versiones más recientes pueden generar fallos, se recomienda utilizar desde la versión 8 hacia abajo. Esta se puede obtener como un script de python.

Soporta las siguientes técnicas vistas introductoriamente en las secciones anteriores: 

- **Boolean-based blind**: Se trata de una técnica ciega que intenta extraer información carácter a carácter. Esto se consigue insertando, tras la consulta válida, una consulta de tipo SELECT que comprobará si el caracter solicitado se corresponde con el carácter almacenado en la BD generando una respuesta booleana por parte del servidor (correcta o incorrecta).

	<br />

- **Boolean-Time-based blind**:  Esta sigue el mismo mecanismo que la técnica anterior, esta vez la consulta SELECT introduce un delay en la BD que sólo tendrá lugar si la condición del SELECT se cumple satisfactoriamente. Aportando de nuevo una información de caracter booleana.

	<br />

- **Error-based**: Esta técnica consiste en provocar deliberadamente mensajes de error de la base de datos en la web y buscar en ellos filtraciones.

	<br />
	
- **UNION query-based**: Esta técnica se basa en añadir una consulta que empiece con UNION ALL SELECT, revelando información sensible sólo si la aplicación web vuelca toda la información de la DB sobre la web atacada.

	<br />
	
- **Stacked queries**: Esta técnica consisten en apilar distintas consultas mediante ';'. Sin embargo sólo funciona en los casos en los que la web permite la ejecución múltiple de consultas. 


Por otra parte, SQLMap soporta la mayoría de los motoroes de bases de datos más conocidos como Oracle, MySQL, PostgreSQL, etc.

<br />

### 6.4.2. Manual de uso básico de SQLMap.

Como ya hemos indicado anteriormente, SQLMap es una herramienta que realiza peticiones a los parámetros de una URL. Por tanto para funcionar se necesitan los siguientes elementos:

- **URL**: 
- **Cookies**: que se pueden obtener de una herramienta de pentesting como BurpSuite o dándole a inspeccionar elemento > GET/POST > Network > Cookies.
- **Data**; si se tratáse por ejemplo de un POST, esto ayuda a la herramienta a conocer la estructura de la página web para entender qué parámetros modificar. Con el GET no es necesario.
- El **modificador**/opción para hacer que SQLMap nos proporcione la información deseada. 


```bash
sqlmap -u <URL> --cookie=<cookie> [--data=<data>] -option
```

El data es específico del tipo HTTP - POST. Va entre corchetes porque si se trata de un GET no sería necesario ponerlo. Recordamos que el data se puede obtener desde Inspeccionar Elemento > Network > POST > Request.
Estas opciones y sus respectias explicaciones pueden obtenerse de man si se tiene instalado como paquete de python en linux o con -h si sólo se tiene como script de python descargado desde github.

<br />

Más información en https://github.com/sqlmapproject/sqlmap/wiki/Usage 

<br />

### 6.4.3. Ejemplos de uso SQLMap.

Una vez nos hemos instalado SQLMap, o lo hemos obtenido como un script de python, lo abrimos en la terminal y desplegamos la ayuda para intentar ver cómo funciona.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109010532.png' | relative_url }}" text-align="center"/>
</div>

Por otra parte, abrimos la aplicación web Mutillidae ( dentro de la vm ). Entramos en la sección "SQL injection". Una vez en ella necesitamos saber qué tipo de request hace para entender cómo debemos de poner los datos en sqlmap. Sencillamente vamos a inspeccionar elemento > Network y realizamos un request que se quedará guardado para que nosotros podamos inspeccionarlo:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109013203.png' | relative_url }}" text-align="center"/>
</div>

En dicha foto podemos comprobar que se trata de un GET, cuáles son los parámetros, las cookies, etc.

Una vez disponemos de esta información montamos el comando sqlmap de la siguiente manera:

```bash
./sqlmap.py  -u "http://10.0.2.15/mutillidae/index.php?page=user-info.php&username=USER&password=PASS&user-info-php-submit-button=View+Account+Details" --cookie="PHPSESSID: kijd2n5u9kddk343cu5pkei7u0;showhints:1"
```

y le damos a enter para saber si esta página web es o no vulnerable. Despúes de un proceso de espera obtenemos:

![[Pasted image 20220109013759.png]]

Con lo que SQLMap nos está informando de que ha encontrado diversos puntos de inyección y que debemos de selecionar uno para hacer la operación requerida. 

Como no le hemos indicado que lleve a cabo ninguna operación en la URL la terminal nos devuelve al prompt seleccionemos la opción que seleccionemos. **Debemos añadir a la misma distintos modificadores para tener acceso a la información de la base de datos**. Por ejemplo:

1. Si queremos ver la base de datos que se está utilizando actualmente añadiríamos: --current-db

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109014228.png' | relative_url }}" text-align="center"/>
	</div>

2. Si queremos ver las tablas que componen una base de datos sencillamente indicamos con -D el nombre de la base de datos y el modificador --tables: -D nowasp --tables

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109014620.png' | relative_url }}" text-align="center"/>
	</div>

3. Si dada una tabla quisiésemos ver sus columnas entonces tendríamos: -D nowasp -T accounts --columns

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109014918.png' | relative_url }}" text-align="center"/>
	</div>
	
4. Si para una base de datos concreta quisiéramos ver un esquema completo de todas sus tablas: -D nowasp --schema
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109015652.png' | relative_url }}" text-align="center"/>
	</div>
	
	
5. Si quisiéramos volcar el contenido de una tabla ponemos: -D nowasp -T accounts --dump

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220109020751.png' | relative_url }}" text-align="center"/>
	</div>

<br />

## 6.5. Uso de BurpSuite para ejecutar SQLi strikes.

### 6.5.1. Intro. Operador UNION.

Una vez se ha establecido que una base de datos es vulnerable a un ataque SQLi a menudo es útil explotar dicha vulnerabilidad para demostrar sus potenciales implicaciones.

El operador UNION se utiliza para combinar los resultados de dos o más sentencias SELECT de forma que en ciertos tipos de vulnerabilidades, el operador UNION se utiliza para llevar a cabo una búsqueda adicional y obtener más resultados de la cuenta.

Pasos:

- **Configurar el proxy de BurpSuite e interceptar una petición que se le mande a dicha web**.
- **Una vez tenemos interceptada la petición que mandamos la primera tarea es identificar el número de columnas que se devuelven en la búsqueda original ya que cada SELECT unido mediante UNION debería de devolver el mismo número de columnas**.

<br />

# 7. Teoría de Bases de Datos y SQL.

## 7.1. Introducción a las Bases de Datos.

El objetivo principal de las bases de datos es el de unificar los datos que se manejan y los programas/aplicaciones que los manejan. Las bases de datos aparecen para solucionar los siguientes inconvenientes:

- **Dependencia hacia los datos**: Anteriormente el diseño de los programas se ajustaban a los datos que iban manejar, había una dependencia de los programas respecto de los datos y cualquier cambio en la estructura de ficheros hacía que el programa se tuviera que modificar y volver a compilar lo cual era un esfuerzo adicional.

- **Redundancia de información**: Como cada aplicación se genera de acuerdo a los datos que maneja pero distintos sectores pueden manejar en parte los mismos datos se produce una redundancia de datos que son comunes en sectores distintos provocando un esfuerzo extra de almacenamiento, actualización con su correspondiente problema de incoherencia de datos (fallos de actualización).

Las bases de datos independizan los datos de las aplicaciones a través de un mecanismo de gestión de datos que los vuelve seguros y accesibles para cada dispositivo que pretenda usarlos, esto es; el Sistema Gestor de Bases de Datos (SGBD).

Este proporciona:

- **Accesibilidad de los datos**: Cambios en la estructura de la base de datos no provoca una alteración en las aplicaciones que los utilizan.
- **Integridad de los datos**: Como se reduce el número de veces que se manipulan los datos se reduce el riesgo de que se produzcan incovenientes fruto de actualizaciones u otras operaciones.
- **Seguridad**: Además, el SGBD implementa un sistema de control de acceso que evita manipulaciones indeseadas o no autorizadas.

<br />

## 7.2. Definición de Base de Datos.

### 7.2.1. Base de Datos.

Una **Base de Datos** resulta en una organización de información de acuerdo a un **modelo de datos** que persigue reflejar las relaciones/restricciones existentes entre los objetos del mundo real en dicha organización y que consigue independencia, integridad y seguridad de los datos.

Estrictamente hablando, una base de datos es una colección de información relacionada con un asunto, tema o actividad específicos y con la que se puede operar, cuyo contenido se puede ordenar, imprimir, modificar, borrar y demás operaciones y algunas más complicadas.

<br />

### 7.2.2. Modelo de Datos. Modelo Relacional.

### 7.2.2.1. Modelo de Datos.

Llamaremos como **Modelo de Datos** al conjunto de conceptos y reglas que nos llevarán a reflejar las relaciones que existen entre los datos que estamos organizando y las operaciones que podemos ejecutar sobre ellos.

<br />

#### 7.2.2.2. Modelo Relacional. Representación gráfica y componentes.

Por otra parte existen una multitud de modelos de datos aplicables para el diseño de bases de datos, entre ellos el más utilizado es el **Modelo Relacional** el cual es un modelo de base de datos basado en la lógica de predicados y la teoría de conjuntos.

En él la representacion gráfica de un elemento básico es la **tabla**:

<br />



<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211220133601.png' | relative_url }}" text-align="center"/>
</div>

<br />

Una tabla se compone de filas y columnas de forma que las filas se corresponden con los **registros** y las columnas con los **campos** y estos objetos están relacionados de la siguiente manera: 

- Un *registro* representa la información global que se tiene de un elemento concreto la cual se encuentra clasificada entre los diferentes *campos* de la tabla.

	<br />

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211220135219.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

- De esta forma, la combinación entre un registro y un campo en una tabla nos proporciona el *atributo* (valor) de un elemento concreto.

	<br />

	<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Imagen6012 1.png' | relative_url }}" text-align="center"/>
	</div>

	<br />

- De forma que el número de filas nos dice el número total de elementos que tenemos en la tabla y el número de campos nos dice el número de datos distintos que tenemos para un elemento particular genérico de la tabla. Así pues podemos asumir las siguientes conclusiones:

<br />

**Un CAMPO es la unidad mínima de información que se gestiona en una base de datos**.

**El valor que un campo toma para un elemento concreto se denomina ATRIBUTO**

**Un conjunto de atributos referentes al mismo elemento conforma un REGISTRO**.

**Un conjunto de registros de distintos elementos termina conformando una TABLA**.

**Varias tablas pueden llegar a conformar una BASE DE DATOS**

**Varias bases de datos están gestionadas por un SGBD** 

 <br />

Además de los elementos fundamentales campo, registro, tabla, podemos encontrar los siguientes elementos:

- **Índice**: Él indice es un identificador único de cada registro que permite un rápido acceso a los mismos y mejora la velocidad de las operaciones entre los mismos.

- **Clave Primaria**: La calve primaria se trata de **un campo o un conjunto de ellos  que permite identificar de forma inequívoca a un registro de la tabla**; se trata, para cada elemento de la tabla, de un atributo único, no nulo. 

	Del ejemplo anterior se puede deducir que nuestra clave primaria de la tabla es el campo DNI; se trata del único campo que no puede ser nulo y que no puede estar repetido para ningún registro pues no existen dos personas con el mismo DNI.

- **Clave Ajena**: Se trata de un mecanismo referencial entre dos tablas que tiene por finalidad evitar la redundancia de información en la misma base de datos.

<br />

#### 7.2.2.3. Profundizando en el modelo relacional.

Hablábamos de que el modelo relacional está basado en la lógica de predicados y en la teoría de conjuntos, en concreto sobre el concepto de relación matemática.

La **relación matemática** entre dos conjuntos no es más que una asociación entre los elementos de ambos conjuntos (en este caso entre el conjunto de los *registros* y el conjunto de los *campos*) de forma que:

- **No existen dos registros idénticos**; de esto se encarga el concepto de clave primaria; este es único para cada registro e identifica cada registro de manera unívoca con lo que consecuentemente si hubieras dos registros iguales compartirían clave primaria y serían por definición el mismo.

- **Ni el conjunto de los registros ni el conjunto de los campos son conjuntos ordenados**: Es decir, no existe ninguna forma de jerarquizar los elementos presentes en ambos conjuntos entre sí.

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211220194025.png' | relative_url }}" text-align="center"/>
</div>

<br />

De esta manera, una base de datos relacional es un conjunto finito de asociaciones con una serie de restricciones (reglas de integridad):

- **Existencia de la clave primaria como campo no nulo.**

- **Existencia de clave ajena como campo no nulo.**

- **Integridad referencial**: Los valores de campo de una clave externa referencias valores reales del campo de dicha clave externa en otra tabla. 


<br />

## 7.3. Fundamentos de SQL.

### 7.3.1. Qué es SQL.  Los sublenguajes de SQL.

El SQL es un acrónimo para Structured Query Language, se trata del lenguaje de manipulación de bases de datos más utilizados por los SGBDs (Sistemas de Gestion de Base de Datos). La parte fundamental del mismo es un estándar global, pero dentro de SQL existen varias divisiones, sublenguajes; conjuntos de instrucciones que atienden a distintas funcionalidades:

- **DDL** (Data Definition Language - Lenguage de Definción de Datos): Se encarga de la creación de objetos (tablas, reports, consultas y formas) de la base de datos.

- **DML** (Data Manipulation Language - Lenguaje de Manipulación de Datos): Utilizado para consultar, generar o actualizar información preexistente en una base de datos.

- **DCL** (Data Control Language - Lenguaje de Control de Datos): Se encarga de controlar el acceso a los objetos y datos para gestiones autorizadas en un SGBD.

- **TCL** (Transaction Control Language - Lenguage de Control de Transacciones): Se encarga del manejo de las **transacciones** (agrupación de un conjunto de operaciones en una sóla) en una base de datos. 

- **DQL** (Data Query Language - Lenguage de Consulta de Datos): Los comandos de SQL que se utilizan para recuperar datos de la base de datos se denominan colectivamente DQL. Todas las declaraciones SELECT vienen bajo DQL.

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20211220185141.png' | relative_url }}" text-align="center"/>
</div>

<br />

Los SGBDs tienen sus propias exitensiones propietarias adicionales que generalmente sólo se utilizan en su sistema, los comandos y declaraciones que se comentan a continación son un estándard.

**Por defecto y por cuestiones pedagógicas, en estos apuntes se utilizarán los comandos para interactuar con un objeto básico como una tabla (TABLE), pero existen muchos objetos con los que interactuar y todos ellos portan su propia extensión junto al comando y sus características sintácticas propias. 
Por ejemplo, para eliminar un rol utilizaríamos un DROP, pero no un DROP TABLE sino un DROP ROLE, sin embargo, esto excede por mucho las pretensiones de este escrito y se dejará a parte.**

<br />

### 7.3.2. Operaciones básicas de DDL.

Como ya hemos indicado, el subconjunto de instrucciones DDL de SQL se encarga de gestionar todo lo relativo a la generación o eliminación de objetos dentro de una base de datos. Entre los comandos más importantes tenemos: CREATE, DROP, ALTER, TRUNCATE, RENAME.

<br />

- **CREATE**: 

	El statement CREATE TABLE se utiliza para crear nuevas tablas en una base de datos. Se le pasa como argumentos el nombre de la nueva tabla (table_name) y los campos que esta va a contener con el tipo de dato que va a ser dicho atributo (varchar: palabras, int: enteros, char: caracteres, etc ):

	```SQL
	CREATE TABLE _table_name_ ( 
	 _column1 datatype_, 
	 _column2 datatype_, 
	 _column3 datatype_, 
	 .... 
	);
	```

	También podemos utilizar CREATE TABLE para copiar una tabla ya existente:
	
	```SQL
	CREATE TABLE _new_table_name_ AS 
	SELECT _column1, column2,..._ 
	FROM _existing_table_name_ 
	WHERE ....;
	```
	
	Este comando crea la tabla '*new_table_name*' copiando las columnas especificadas '*column1, column2,...*' de la tabla existente '*existing_table_name*' y '*WHERE*' añade más condiciones opcionales.
	
	<br />
	
- **DROP**: 

	 El statement **DROP TABLE** es utilizado para eliminar una tabla de una base de datos.

	```SQL
	DROP TABLE table_name;
	```
	
	Evidentemente y como no puede ser de otra forma, la eliminación de una mesa resultará en la pérdida de la información de todo lo que haya contenido en la tabla.
	
	<br />
	
- **ALTER**: 

	El statement **ALTER TABLE** es utilizado para añadir, borrar o modificar las columnas o restricciones (tipo de datos de columna, etc) en una tabla existente:
	
	- Añadir columna:
	
		```SQL
		ALTER TABLE _table_name_  
		ADD _column_name datatype_;
		```
	
	- Eliminar columna: 
		
		```SQL
		ALTER TABLE Customers  
		DROP COLUMN _column_name_;
		```
		
		<br />
		
	 Para cambiar el tipo de datos de una columna en una tabla; use la siguiente sintaxis en función del sistema de gestión de base de datos que utilice:
		
		
	- ** SQL Server / MS Access:**
		```SQL
		ALTER TABLE _table_name_  
		ALTER COLUMN _column_name datatype_;
		```
	- **Mi SQL / Oracle (versión anterior 10G):**

		```SQL
		ALTER TABLE _table_name_  
		MODIFY COLUMN _column_name datatype_;
		```
	- **Oracle 10G y posterior:**
		```SQL
		ALTER TABLE _table_name_  
		MODIFY _column_name datatype_;
		```
		
<br />
		
- **TRUNCATE**: 

	El statement **TRUNCATE TABLE** borra el contenido de una tabla, pero no borra la tabla en sí misma. Podríamos decir que "limpia"	una tabla:
	
	```SQL
	TRUNCATE TABLE Categories;
 	```
<br />
	
### 7.3.3. Operaciones básicas de DML.

Las operaciones DML trabajan sobre los datos almacenados en nuestro SGBD, **sobre datos que ya existen**.

En general a las operaciones básicas de manipulación de datos que podemos realizar con SQL se les denomina **operaciones CRUD** (de _**C**reate, **R**ead, **U**pdate and **D**elete_, esto es; Crear, Leer, Actualizar y Borrar, sería CLAB en español, pero no se usa). 

Hay cuatro instrucciones para realizar estas tareas: INSERT, SELECT, UPDATE, DELETE, EXECUTE/CALL.


-   **INSERT INTO**: 
   
	Inserta nuevos registros en una tabla. Se corresponde con la “C” de CRUD.
	
	Más concretamente, INSERT INTO es el estándard de INSERT en la mayoría de las variantes de SQL, esto significa que aunque para casos particulares bastaría con poner INSERT siempre será mejor poner INTO pues de esa forma nos aseguramos de que el código sea portable. 
	
	Elementalmente, hay dos formas de utilizar INSERT INTO:

	- Especificando los campos:

		```SQL
		INSERT INTO _table_name_ (_column1_, _column2_, _column3_, ...)  
		VALUES (_value1_, _value2_, _value3_, ...);
		```

		Si lo que se pretende es insertar valores en un grupo concreto de campos.

	- O sin especificar campos si lo que se pretende es actualizar todos los campos. 

		```SQL
		INSERT INTO _table_name_  
		VALUES (_value1_, _value2_, _value3_, ...);
		```
	
	<br />
	
-   **READ**: 
   
	   Muestra información sobre los datos almacenados en la base de datos. Dicha información puede pertenecer a una o varias tablas. Es la “R”. Se utiliza SELECT perteneciene al sublenguaje DQL.

	<br />

-   **UPDATE**: 
  
	Modifica los registros de una tabla. Es, obviamente, la “U”. 
	
	```SQL
	UPDATE _table_name_  
	SET _column1_ = _value1_, _column2_ = _value2_, ...  
	WHERE _condition_;
	```
		
	Hay que observar que la cláusula **WHERE** en el statement **UPDATE** especifica qué registros deben actualizarse, implementa una condición que filtra registros indeseados. Si omite la cláusula **WHERE**, se actualizarán todos los registros de la tabla.
	
	<br />
	
-   **DELETE**: Borra registros de una tabla. Se corresponde con la “D”.

	```SQL
	DELETE FROM _table_name_ WHERE _condition_;
	```
	
	De nuevo hay que observar que la cláusula **WHERE** en el statement **UPDATE** especifica qué registros deben actualizarse, implementa una condición que filtra registros indeseados. Si omite la cláusula **WHERE**, se borrarán todos los registros de la tabla.
	
	<br />
	
- **EXECUTE**: El comando EXECUTE permite ejecutar procedimientos. **CALL** se utiliza como un acortamiento de EXECUTE en muchos casos:

	```SQL
	EXECUTE proc()

	CALL proc()
	```
	
	Un procedimiento no es más que un conjunto de instrucciones que puede ser insertado manualmente o puede estar guardado bajo un nombre (procedimiento almacenado).
	
	<br />

### 7.3.4. Operaciones básicas de DCL.

#### 7.3.4.1. Concepto de DCL.

El DCL (Data Control Language) es un sublenguaje de SQL con una sintaxis muy parecida a un lenguaje de programación computacional que se utiliza para controlar el acceso a los datos almacenados en una base de datos. En concreto, se utilizan para conceder o quitar permisos/autorización a cualquier usuario de un entorno de base de datos multiusuario.

Solo el **administrador de la base de datos o el propietario del objeto** de la base de datos pueden proporcionar / eliminar privilegios sobre un objeto de la base de datos.

<br />

#### 7.3.4.2. Antecedentes: Benficiarios, Privilegios y Roles.

Antes de conocer los comandos y su sintaxis debemos de conocer los siguientes conceptos:

- **Privilegios**: Entre los privilegios de acceso que podemos conceder a un determinado usuario encontramos:

	- **Privilegios de Sistema**: CREATE, ALTER y DROP (crear, alterar y eliminar) objetos de la base de datos.
	
	- **Privilegios de Objetos**: EXECUTE, SELECT, INSERT, UPDATE o DELETE (EJECUTAR, SELECCIONAR, INSERTAR, ACTUALIZAR o BORRAR) datos de los objetos de la base de datos.
	
		Además podemos encontrar:

		- **ALL**.
		- **WITH GRANT OPTION**: Permite a los usuarios designados conceder derechos de acceso a otros usuarios.

		- **ROL**: Conjunto de privilegios que son otorgados. Los roles son una colección de privilegios o derechos de acceso. Cuando hay muchos usuarios en una base de datos, resulta difícil otorgar o revocar privilegios a los usuarios. Por lo tanto, si define roles, puede otorgar o revocar privilegios a los usuarios, otorgando o revocando privilegios automáticamente. Entre los roles más destacables se encuentran:

			- **CONNECT**: CREATE TABLE, CREATE VIEW, CREATE SYNONYM, CREATE SEQUENCE, CREATE SESSION etc.
			
			- **RESOURCE**: CREATE PROCEDURE, CREATE SEQUENCE, CREATE TABLE, CREATE TRIGGER etc. Este tiene como objetivo principal restringir el acceso a los objetos de la base de datos.

			- **DBA**: Todos los privilegios.


	Es decir, que algunos permisos nos permiten operar con objetos de la base de datos y otros comandos nos permiten operar directamente con los datos de los objetos de la base de datos.

<br />

- **Beneficiario de los permisos**: De entre todas las entidades a las que podemos conceder o eliminar un privilegio de los anteriores encontramos:

	- **Usuario**: Concede acceso a un usuario en concreto especificado mediante el nombre del mismo.
	- **Público**: Concede acceso a cualquier usuario.

	

<br />

#### 7.3.4.3. Comandos DCL.
	
- **GRANT**: Se utiliza para conceder privilegios sobre un objeto a un usuario o conjunto de usuarios. Tiene por sintaxis:


	```SQL
	GRANT privilege_name 
	ON object_name 
	TO {user_name |PUBLIC |role_name} 
	[WITH GRANT OPTION];
	```
	
	El comando anterior concede el privilegio '*privilege_name*' sobre el objeto '*object_name*' al beneficiario '*{user_name | PUBLIC | role_name}*'

	
- **REVOKE**: Este comando quita derechos de acceso / privilegios a ciertos usuarios sobre objetos de la base de datos.

	```SQL
	REVOKE privilege_name 
	ON object_name 
	FROM {user_name |PUBLIC |role_name}
	```


<br />

### 7.3.5. Operaciones básicas con TCL.

#### 7.3.5.1. Introducción al TCL.

El TCL (Transmission Control Language) o Lenguaje de Control de Transacciones es un subconjunto de instrucciónes de SQL que permiten manejar transacciones en una base de datos relacional, por lo que es importante primeramente aclarar el concepto de transacción.

<br />

#### 7.3.5.2. Concepto de transaccion en una base de datos relacional.

Sabemos por lo que hemos visto hasta ahora que sobre los datos de una base de datos podemos operar de diversas maneras. Muchas veces puede ocurrir que estas operaciones sean operaciones complejas; concatenaciones de operaciones simples que pueden en algún punto de la cadena no ser viables y dicha inviabilidad no se vislumbra hasta que no se ha alcanzado dicho punto, momento en el cual ya se han llevado a cabo un monto de operaciones sobre los datos provocando alteraciones que en el mejor de los casos exponen a los datos a problemas de incoherencia de datos.

Por ello y para evitar riesgos, las bases de datos relacionales manejan el concepto de **transacción** que no es más que agrupar todas las operaciones en un solo concepto de tal manera que se le indica a la base de datos que haga todas las operaciones o no haga nada una vez se ha verificado la viabilidad de la operación.

<br />

#### 7.3.5.3. Implementación del TCL.

Para poder implementar el concepto de transacción adecuadamente SQL define una serie de comandos que permiten: crear una transacción y grabar los datos o deshacer todas las operaciones de la transacción.

- **BEGIN TRASACTION**: Para comenzar una transacción.
- **COMMIT**: Para grabar todos los cambios en la base de datos.
- **ROLLBACK**: Para deshacer todos los cambios de toda la transacción.


Los comandos pueden ser ligeramente distintos dependiendo de cada motro el concepto siempre es en esencia el mismo:

1. Se define el inicio de una transacción.
2. Se realizan las operaciones necesarias (UPDATEs, INSERTs, DELETEs, etc).
3. Se intentan grabar todos las operaciones hechas hasta cierto punto.
4. Si existe un error se deshace la operación como estaba (ROLLBACK).

<br />

### 7.3.6. Operaciones con DQL.

DQL se utiliza para recuperar los datos de una base de datos. Este sólo contiene un único comando que tiene una gran extensión: SELECT.

<br />

#### 7.3.6.2. SELECT.

El comando SELECT se utiliza para seleccionar datos de una base de datos. El dato devuelto por SELECT se muestra en lo que se conoce como una tabla de resultados.

```SQL
SELECT _column1_, _column2, ..._  
FROM _table_name_;

OUTPUT

 -------------------
| column1 | column2 | ....
---------------------
| atrib1  | atrib2  | ....
---------------------
    ....      ....
```

Si por otra parte queremos seleccionar toda la table escribiríamos:

```SQL
SELECT * FROM table_name;
```

<br />

#### 7.3.6.3. SELECT INTO.

El comando SELECT con la extensión INTO copia los datos desde una tabla a otra tabla nueva:

```SQL
SELECT _column1_, _column2_, _column3_, ...  
INTO _newtable_ [IN _externaldb_]  
FROM _oldtable  
```

Este comando selecciona un conjunto de columnas que transporta desde '*oldtable*' a '*newtable*'.

<br />

#### 7.3.6.4. Cláusula WHERE.

La cláusula **WHERE** se utiliza para filtrar registros que cumplan una determinada condición.

```SQL
SELECT _column1_, _column2, ..._  
FROM _table_name_  
WHERE _condition_;
```

En el comando anterior sólo se seleccionarían los atributos pertenecientes a las campos especificados y a los registros que cumpliesen con la condición impuesta por el where. 

<br />

# 8. Cross-Site Scripting (XSS).

## 8.1. Ataques XSS.

### 8.1.1. Introducción.

El Cross-Site Scripting (XSS) es un tipo de ataque por inyección, se trata de un tipo de ataque que aprovecha fallas de seguridad en sitios web para ejecutar scripts maliciosos en el frontend de la aplicación web, esto es; el navegador del cliente. Con esto se puede robar credenciales, redirigir al usuario a un sitio malicioso, etc.

Vamos a presentar tres vectores de ataque utilizados para explotar dicha vulnerabilidad

<br />

### 8.1.2. Tipos de XSS.

Como comentabamos anteriormente, XSS es un tipo de ataque a las aplicaciones web que busca infiltrar un script malicioso en el navegador del cliente. Comúnmente este proceso se basa en la confiaza que tiene el sitio web en cuestión en la entrada de datos, se le envía una URL con el payload precargado a la víctima.

- **XSS Reflejado**: Un ataque XSS reflejado (Reflected XSS) consiste en inyectar el payload (la carga útil, el contenido malicioso) en algún parámetro de la solicitud HTTP que se envía al servidor, la aplicación web procesa esta solicitud y despliega el código malicioso en algún punto determinado del documento HTML respuesta. De esta manera, se altera el documento HTML y cuando la respuesta llega al cliente, el código malicioso dentro del documento se procesa alterando el normal comportamiento del navegador.

<br />

- **XSS Persistente**:  El XSS persistente (Persistant XSS) es un ataque que tiene un fundamento similar al XSS Reflected que tiene como característica fundamental que el propio servidor almacena el contenido malicioso en un lugar de almacenamiento y este contenido es adherido en cada solicitud que se devuelve como respuesta a la petición de uno o varios clientes que piden una solicitud a dicho servidor a través del mismo sitio web.

	Un ejemplo clásico podría ser, por ejemplo, los comentarios de un foro de dudas, entradas de un blog, ... En general cualquier apartado donde se pueda introducir información que vaya a ser almacenada y devuelta por el servidor en el documento html para cada solicitud de manera permanente.

<br />

- **XSS DOM**: El XSS basado en DOM es un tipo de ataque XSS que se basa en alterar el entorno DOM de la página web con la finalidad de hacer que el navegador del cliente se comporte de manera maliciosa.

	El DOM es una API que interpreta el contenido de un documento HTML como objetos para que estos puedan ser tratados por el navegador de una manera más sencilla a la hora de renderizar el documento y también permite que ciertos lenguajes de programación basados en scripts como JavaScript puedan ejecutar procesos con los elementos del documento HTML y el navegador. 

	Esto es importante porque en este caso el documento HTML no cambia porque la respuesta HTTP no se ve afectada, no se añade código extra, sencillamente se manipulan los objetos del navegador para conseguir un comportamiento malicioso. Sin embargo el código del lado del cliente se ejecuta de manera diferente debido a la modificación del entorno en el que este se está ejecutando.


<br />

## 8.2. Ejemplos.

Para realizar estos ejemplos, vamos a emplear una máquia virtual preparada que contiene una herramienta web vulnerable (OWASP BWE, Mutillidae) con distintas secciones que nos ayudarán a ver cómo funcionan los ataques XSS.

<br />

### 8.2.1. DNS Lookup.

    

Por un fallo de configuración de la web, el código que nosotros introducimos se añade en bruto al documento html que el servidor devuelve. Como no se filtra de ninguna manera, podemos poner un elemento html o un script de javascript y cuando el navegador renderice el documento respuesta se encontrará con un elemento html más que renderizar. De esta forma hemos conseguido alterar el normal funcionamiento del navegador demostrando que el sitio web es vulnerable. Se trata de un XSS Reflected porque añadimos una carga HTML al documento que enviamos al servidor y que este nos devuelve pero esta no se almacena en el servidor.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301202954.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.2. Text File Viewer.  

En este caso nos encontramos ante un sitio web que nos da a elegir entre un conjunto de ficheros que luego despliega por pantalla:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301203204.png' | relative_url }}" text-align="center"/>
</div>

Inspeccionando el elemento HTML de la opción que se muestra como posibilidad en el desplegable **Text File Name** nos encontramos con que el nombre que antecede como cabecera a la muestra del documento: **File: http://www.textfiles.com/hacking/auditool.txt** es igual al valor (value) que tiene el elemento html correspondiente a la opción del menú desplegable; **Intrusion Detection in Computers by Victor H. Marshall,** en el apartado inspeccionar elemento:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301203242.png' | relative_url }}" text-align="center"/>
</div>

Esto puede ser indicio de que el servidor sencillamente coge el valor de la opción y lo recoloca en otra parte del documento html que devuelve como respuesta al cliente.

Si alguien modificara el texto del valor del elemento html y el servidor no filtrase ciertos tipos de caracteres se podría modificar el documento HTML para que el navegador se comportase de manera diferente al elegir dicha opción en el menú como se muestra a continuación:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301203304.png' | relative_url }}" text-align="center"/>
</div>

Ya que dicho fragmento de texto se recolocaría en bruto en otra parte del documento html a la espera de que el navegador lo renderizase. De manera que el servidor es vulnerable a una inyección de código HTML o JavaScript.

<br />

### 8.2.3. SQL Injection.

En este caso nos encontramos ante una pantalla con una serie de campos en los que se puede meter información. De nuevo, esta información se muestra de vuelta en el navegador y como el servidor es vulnerable el código no se filtra con lo que podemos cambiar el comportamiento del navegador introduciendo elementos HTML como scripts de JavaScript.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301203413.png' | relative_url }}" text-align="center"/>
</div>

En la imágen podemos observar que hemos introducido un hola en uno de los campos y el hola se nos devuelve en forma de resultados encontrados debajo de los campos.

Si ahora ponemos \<h1> hola obtenemos…

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301203441.png' | relative_url }}" text-align="center"/>
</div>

Recordamos que la inyección SQL es también posible en esta página pero esa no sería un XSS strike ya que el objetivo del anterior consiste modificar el **frontend** de un sitio web puesto que la víctima sería un cliente de dicha página mientras que en el ataque SQLi la víctima es el servidor y el objetivo del ataque es el backend de la aplicación web.

<br />

### 8.2.4. Set Backgroun Color.

Esta vez estamos ante una aplicación que nos cambia el fondo de una parte de la pantalla. De nuevo, el código que nosotros introduzcamos se nos devolverá en bruto a modo de resultado y podemos aprovecharlo para introducir elementos html dentro del documento html respuesta que será renderizado por el navegador alterando así el comportamiento del mismo.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301203759.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.5. HTML5 Storage.
En esta página nos encontramos con un página que añade valores a una tabla en tiempo real. Esto significa que la página ejecuta código javascript que modifica el DOM (el entorno gráfico de la página) de la misma sin necesidad de llevar a cabo ninguna request. Además observamos que el nombre de los términos que nosotros introducimos sale por pantalla inmediatamente después de pulsar el botón:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204044.png' | relative_url }}" text-align="center"/>
</div>

De esta manera, si nostros introducimos en el primer campo un elemento HTML, y la página no filtra determinados caracteres, aunque no se produzca ninguna request, como la tabla se modifica el tiempo real, nuesttro código se ejecutará en tiempo real permitiendo alterar el comportamiento del navegador a través del DOM.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204102.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.6. Capture Data Page.
En este caso nos aparece un mensaje que nos informa de que la página web está diseñada para capturar y almacenar todos los datos en un fichero y en una base de datos que posteriormente se renderiza a una tabla.

Además nos pide que asumamos que la persona que está viendo los datos de la tabla lo hace a través de un navegador web sobre el fichero “captured-data.php”.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204144.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204153.png' | relative_url }}" text-align="center"/>
</div>

Teniendo en cuenta que se cogen los parámetros que se envían con nuestra request HTTP y se introducen en un fichero que se transforma en una tabla, podríamos probar a enviar, dentro del valor de algún parámetro, un elemento html para ver si este se ve destacado en la propia tabla.

  

De ser así, significa que la tabla no filtra ciertos caracteres a la hora de construir como documento html y que por tanto se trata de una página vulnerable a un ataque de inyección html.

Envíamos en nuestra URL un parámetro denominado payload=\<h1>HOLA\</h1> y vamos al enlace donde se construye la tabla.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204247.png' | relative_url }}" text-align="center"/>
</div>

Efectivamente, nuestro mensaje se ve resaltado y se trata de un sitio vulnerable a un ataque de inyección html pero no sólo eso. Como podemos observar se recogen casi todos los parámetros que se envían en el cuerpo de la request (Cookies, Refer,...), bastaría con modificar cualquiera de ellos adecuadamente para poder generar un ataque.

  

Además, esto no tiene que ver con la desarrollo del ejercicio en sí, pero tanto esta página como la página principal del ejercicio tienen un elemento html “href” que permiten que el sitio enlace documentos que no se encuentran dentro del esquema de ficheros de la página web. Esto significa que la página podría tener una vulnerabilidad conocida como vulnerabilidad de inclusión de ficheros, que se aprovecha de una mala programación de la web para crear una referencia sobre un archivo remoto que contiene información sensible.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204310.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204314.png' | relative_url }}" text-align="center"/>
</div>


### 8.2.7. Document Viewer.

En este caso estamos ante una página que nos da a elegir entre distintos ficheros a visualizar.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204522.png' | relative_url }}" text-align="center"/>
</div>

Y detalle importante a modo de cabecera se nos informa del nombre del fichero que estamos visualizando, en este caso “robots.txt”.

  

Inspeccionando el elemento html de la opción que elegimos a la hora de visualizar el documento podemos observar que el término “robots.txt” aparece como valor de “value” del elemento html. Esto de nuevo puede ser indicio de que este término se toma del valor de “value” y se transporta a otra parte del documento html sin filtrar caracteres inadecuados pudiendo ser, por tanto, vulnerable a un ataque de html inyection.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204540.png' | relative_url }}" text-align="center"/>
</div>

Un cambio de robots.txt a un elemento html como \<h1>HOLA\</h1> podría confirmar nuestra sospecha y así tenemos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220301204615.png' | relative_url }}" text-align="center"/>
</div>

Demostrando así que se puede alterar el correcto funcionamiento del navegador del usuario.

<br />

### 8.2.8. XML Validator.
Esta es una página que válida el código xml que nosotros introducimos y nos lo devuelve por pantalla.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180437.png' | relative_url }}" text-align="center"/>
</div>

Aquí se nos plantea un problema ya que las marcas que comienzan una etiqueta html y una etiqueta xml son las mismas ‘<>’ y por tanto, aunque intentemos colar algún elemento html con la esperanza de que eso modifique el documento html al reproducirse el mismo texto más abajo, eso no surtirá ningún efecto debido a que al filtrarse el xml también se filtra el html y el documento no se modifica.

Sin embargo, la página tiene un gap de información puesto que cuando no metemos código html nos aparece un mensaje de error mostrándonos cómo es la función que procesa el código que nosotros introducimos.

Si nosotros introducimos el término ‘TEST’ y nos acercamos un poco, podemos ver que el código que nosotros introducimos se procesa mediante la llamada a una función predefinida “loadXML” que se cierra con unas comillas y unos paréntesis.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180459.png' | relative_url }}" text-align="center"/>
</div>

 De esta forma, y aprovechándonos de este exceso de información, podemos introducir nuestro código en dos partes:

1. La primera parte cierra la función para que nuestro texto no sea procesado y posteriormente filtrado como código XML.

2. La segunda contendrá nuestro payload.

Así:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180524.png' | relative_url }}" text-align="center"/>
</div>

y al validar se obtiene:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180539.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.9. Poll Question.

En esta pagina se me da elegir entre diversaas opciones para votarlas y el nombre de la opción elegida se muestra abajo por pantalla. De nuevo este nombre se recoge del valor del elemento html de la opción y este valor puede modificarse desde inspeccionar elemento.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180610.png' | relative_url }}" text-align="center"/>
</div>

Cambiando dicho valor, introduciendo un elemento html y pulsando el botón obtenemos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180623.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180754.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.10. Register User.
Nos encontramos ante una página que crea una cuenta para un usuario con contraseña y demás elementos.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180919.png' | relative_url }}" text-align="center"/>
</div>

Cuando creamos una cuenta aparece un mensaje por pantalla que contiene el nombre del usuario para el que le hemos creado la cuenta.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180932.png' | relative_url }}" text-align="center"/>
</div>

Esto puede ser indicio de que el nombre del usuario se recoge y se traslada a otra parte del documento html sin filtrar caracteres maliciosos, lo cual expondría a la web a un ataque de HTML injection.

Introduciendo un elemento html en el campo de username, por ejemplo: \<h1>hola \</h1> obtenemos:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302180948.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.11. Browser Info.

En esta página se nos muestra información de nuestro navegador recopilado en una tabla. En dicha tabla se puede observar que una de las cosas que se recopila son las cookies.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302181027.png' | relative_url }}" text-align="center"/>
</div>

Así pues, yendo al apartado inspeccionar elemento Storage > Cookies > PHPSESSID tenemos el valor de las cookies y cambiando este por un elemento html y refrescando la página:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302181109.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302182052.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.12. Those Back Button.
En esta página se muestra un texto hablando de que los botones ‘back’ no son más que accionadores que ejecutan código javascript que te llevan a otra página mediante una href. Si por algún casual algún atacante consiguiera modificar el script que dicho botón ejecuta podría hacer que el navegador ejecutase código indeseado.

Concretamente, inspeccionando el elemento del botón podemos concretar cómo es este script que el botón ejecuta cuando es pulsado:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302182342.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, quitando ese código y poniendo un alert en su lugar obtenemos:



<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302182354.png' | relative_url }}" text-align="center"/>
</div>

y pulsando el botón:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302182407.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.14. Styling with Mutillidae.

En esta página nos topamos con un mensaje que nos dice “Sytling with Mutillidae” y fijándonos un poco en la URL observamos que el mismo comentario aparece al final de la URL, indicio de que la página está diseñada para mostrar pantalla la última parte de la URL.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302182730.png' | relative_url }}" text-align="center"/>
</div>

Si este fragmento de texto se pone sin filtrar caracteres especiales, esta página podría ser vulnerable a un ataque de HTML injection. Así, tenemos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302182937.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 8.2.15. Password Generator.

En este caso nos encontramos una página que genera contraseñas aleatorias dirigidas a un usuario. Observamos por otra parte que este usuario por defecto es “anonymous”, el mismo término que aparece en la URL como valor del parámetro de username.

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302183030.png' | relative_url }}" text-align="center"/>
</div>

Esto puede ser indicio de que en el documento html aparece el valor en bruto del parámetro GET username. Así pues asignamos a través de la URL al parámetro username el valor \<h1>HOLA\</h1> y obtenemos:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302183056.png' | relative_url }}" text-align="center"/>
</div>

Demostrando así que la página es vulnerable a un ataque de html injection.

También existe otra forma de demostrar la existencia de esta vulnerabilidad, mediante un ataque DOM XSS alterando la funcionalidad del botón “Generate Password” que ejecuta un código javascript que genera una contraseña sin necesidad de enviar una request.

Inspeccionando el elemento y alterando el código JavaScript que ejecuta se obtiene:

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302183117.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M3/Pasted image 20220302183123.png' | relative_url }}" text-align="center"/>
</div>

