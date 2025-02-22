---
layout: post
title: Metasploit
subtitle: Introducción a la herramienta Metasploit.
tags: [redteam] 
---

# 1. Metasploit Básico.
## 1.1. Antecedentes.

## 1.1. Exploits. 

Denominamos como **exploit** a un programa informático, una parte de software o una secuencia de comandos que se aprovecha de un error o vulnerabilidad para provocar un comportamiento no intencionado o imprevisto en un software, hardware o en cualquier dispositivo electrónico.

<br />

### 1.1.2. Payload.
El payload es la parte del código que se inyecta y actúa después de haber explotado la vulnerabilidad para conseguir realizar una tarea dentro de la máquina victima como recoger datos, abrir una shell, cifrar, etc.

<br />

## 1.3. Backdoor.

Un backdoor es un mecanismo que proporciona un acceso ilícito a un equipo víctima. 

Estos inicialmente no estan concebidos como un elemento malicioso, existes backdoors que se utilizan en caso de crasheo del sistema y para situaciones de emergencia. Sin emabrgo pueden ser utilizadas de manera maliciosa por un usuario ilegitimo. 

Un backdoor no debe confundirse con un reverse shell. **El backdoor consiste en el acceso ilícito y el reverse shell es el mecanismo que nos da acceso a una terminal del equipo**.

<br />

### 1.1.4. Shellcode.

Una shellcode es un fragmento de código que se introduce en el exploit a modo de payload que se encargará de abrirnos una shell en una máquina víctima.

<br />

## 1.2. Metasploit.

### 1.2.1. Qué es Metasploit Framework.

Metasploit Framework es una herramienta para desarrollar y ejecutar código de explotación contra una máquina de destino remota. Esta cuenta con un mecanismo de inyección de exploits y payloads en una vulnerabilidad detectada sobre una máquina remota. Con él podemos por ejemplo ejecutar una shellcode y demás acciones.

<br />

### 1.2.2. Preparación. 

Metasploit Framework puede ser instalado desde la apt (con apt-get install) y necesita la base de datos PostgreSQL operativa para poder funcionar correctamente. Esta se puede activar antes o después de haber iniciado el programa de Metasploite con los comandos systemctl o service.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220114101535.png' | relative_url }}" text-align="center"/>
</div>


Cuando apagemos la máquina, el servicio de postgre quedará inactivo en él

<br />

### 1.2.3. Iniciando Metasploite. Comandos importantes.

Una vez tenemos instalado metasploit-framework y tenemos activado el servicio de postgre iniciamos el framework con los siguientes comandos: 

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220205202228.png' | relative_url }}" text-align="center"/>
</div>

Se abrirá el entorno de trabajo de Metasploite. 

En él podemos seleccionar los siguientes comandos:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220114110601.png' | relative_url }}" text-align="center"/>
</div>

use: usar exploits.

<br />

### 1.2.4. Ejemplo de uso de Metasploite.

Vamos a hacer un ejemplo de uso con Metasploite y una máquina virtual.

Supongámos que hemos hecho OSINT sobre una máquina concreta, por ejemplo un servidor, y descubrimos que posee la vulnerabilidad: **CVE-2004-2687**.

Una vez sabemos para qué vulnerabilidad buscamos el exploit acudimos a Metasploite no sin antes haber iniciado el servicio de postgre con **systemctl**.

Una vez nos hemos cerciorado de que el servicio de postgre corre satisfactoriamente iniciamos el framework de Metasploite con **msfconsole** y buscamos los módulos asociados a la CVE con **search**.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220116202019.png' | relative_url }}" text-align="center"/>
</div>

De esta forma y tal y como se puede ver en la imágen anterior, Metasploite te devolverá una lista indexada de los modulos asociados a dicha vulnerabilidad. Para operar con uno utilizamos el comando **use** seguido del índice asociado al exploite o del nombre del mismo. Una vez escogido el módulo, pedimos a la consola que nos enseñe los payloads mediante **show payloads**:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220116203604.png' | relative_url }}" text-align="center"/>
</div>

Así, se nos muestra una lista de líneas que describen brevemente en qué consiste el payload en cuestión y cómo está hecho. 

Cada payload tiene su entorno correspondiente que necesita de variables a las que da un valor para que pueda operar de manera correcta y algunos, pese a estar correctamente configurados, pueden no funcionar. Probaremos con un payload con **set payload** seguido del índice o nombre asociado al payload y examinaremos sus opciones con **options**:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220116205657.png' | relative_url }}" text-align="center"/>
</div>

Podemos observar en el listado de variables que aparece que algunas tienen valor y otras y no, pero más importantes, algunas variables como *RHOST* son necesarias para expedir el exploit y por tanto hay que rellenarlas. En este caso RHOST es "remote host" y se trata de la dirección IP de la máquina remota.

Ajustamos este valor con **set rhost  \<IP>** y ejecutamos con **run**.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220116213432.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220116213501.png' | relative_url }}" text-align="center"/>
</div>

Así, podemos comprobar que efectivamente el exploit se ha ejecutado correctamente en la víctima y gracias al payload msfconsole nos devuelve una shell desde la que podemos inyectar comandos permitiéndonos así tomar el control del equipo de la víctima.

Además, conviene añadir que podemos enviar una sesión abierta al "background" para seguir utilizando sobre ella módulos de post explotación u otras herramientas que ya requieran de tener una conexión con la máquina víctima:

- **Meterpreter session**: bg.
- **Non-meterpreter session**: Ctrl + Z.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220131104402.png' | relative_url }}" text-align="center"/>
</div>

Luego podemos acudir a ella con el comando session -i o sessions.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220114110714.png' | relative_url }}" text-align="center"/>
</div>

<br />

# 2.Metasploit Offensive Security.

## 2.1. Introducción a Metasploit.

### 2.1.1. Intro.

Metasploit Framework (MSF) es una de las herramientas de auditoria y pentesting que pueden obtenerse de manera.  Se trata de un marco de trabajo que le permite buscar exploits a vulnerabilidades conocidas y lanzarlos sobre máquinas victimas con la finalidad de tomar control sobre las mismas gracias a la acción de los payloads que cada exploit tiene asociado.

<br />

### 2.1.2. Requisitos de Software de Metasploit.

Para poder hacer uso de Metasploit, además de saber con certeza de que se cumplen los requisitos de hardware en la máquina donde tenemos instalada la herramienta, necesitamos los siguientes componentes:

- **KaliLinux**, distro de linux con herramientas de pentesting en el que tendremos instalado la herramienta Metasploite. Puede ser virtual.

- **Máquina Victima**. Una máquina vulnerable sobre la que ejecutar el exploit conveniente. Puede ser virtual.

- **HiperVisor**, programa que regule la comunicación entre ambas máquinas, como VirtualBox.

<br />

### 2.1.3. Arquitectura de Metasploit.

#### 2.1.3.1. Instalación y lugar de almacenamiento.

Metasploit es un software escrito en Ruby. **Metasploit puede instalarse desde la apt mediante el paquete metasploit-framework que se instala en /usr/share/metasploit-framework**.

<br />

#### 2.1.3.2. Sistema de ficheros.

Dentro de la carpeta /usr/share/metasploit-framework encontramos entre otras las carpetas: data, documentation, lib, modules, plugins, scripts, tools.


<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117163125.png' | relative_url }}" text-align="center"/>
</div>

<br />

- **data**: El directorio 'data' contiene archivos editables utilizados por Metasploit para almacenar los binarios necesarios para ciertos elementos como exploits, wordlists, imágenes, etc. 

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117163306.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

- **documentation**: En la carpeta documentation podemos encontrar documentación disponible del entorno de trabajo de Metasploit.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117164506.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

- **lib**: El directorio lib contiene el código que soporta la base del entorno de trabajo.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117165416.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

- **modules**: Es donde se encuentran los modules de MSF para los exploits; auxiliary, post modules, payloads, encoders, etc.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117165553.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

- **plugins**: los plugins para añadir funcionalidades al comportamiento del software Metasploit y pueden ser requeridos para que Metasploit interactúe con aplicaciones externas como nmap.

	<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117165740.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

- **scripts**: Contiene elementos como Meterpreter y otros scripts utilizados en payloads.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117165817.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

- **Tools**: Utilidades de la línea de comandos.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117170130.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

#### 2.1.3.3. Librerias de Metasploit.

Las libreras son fragmentos de códigos que en este caso están incorporados dentro de la aplicación y que nos evitan realizar ciertas tareas manualmente mediante la incorporación de código extra a la hora de expedir los exploits como HTTP request, codificar los payloads, etc. Algunas de las bibliotecas son:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117170827.png' | relative_url }}" text-align="center"/>
    </div>

<br />

### 2.1.4. Modulos: Localizaciones y Tipos.

#### 2.1.4.1. Definición y Localización.

En Metasploit *un módulo es una pieza o bloque de código que implementa una o varias funcionalidades* como ejecutar un exploit concreto o el escaneo de máquinas remotas, etc. Todos los módulos son clases de Ruby.

Estos módulos son buscados en dos localizaciones concretamente:

- **/usr/share/metasploit-framework/modules/**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117171611.png' | relative_url }}" text-align="center"/>
    </div>

-  **~/.msf4/modules/**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117171850.png' | relative_url }}" text-align="center"/>
    </div>

<br />

#### 2.1.4.2. Tipos de módulos.

Entre los tipos de módulos que encontramos tenemos: Exploits, Auxiliary, Payloads, Encoders, Nops.

- **Exploits**: Un Exploit es un tipo de módulo que utiliza un payload. Es decir, que utiliza un fragmento de código adicional. Estos pueden encontrarse en: **/usr/share/metasploit-framework/modules/exploits**:

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117172617.png' | relative_url }}" text-align="center"/>
    </div>

<br />

- **Auxiliary**: Los módulos auxiliares son aquellos que se utilizan para generar acciones que tienen un propósito auxilar a la tarea principal. Entre este tipo de tareas encontramos: Escaneo de puertos, fuzzers, sniffers y más. Acciones que pueden ayudarnos a elegir mejor el tipo de exploit, de payload, etc.

	Estos pueden encontrarse en: **/usr/share/metasploit-framework/modules/auxiliary**:

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117173117.png' | relative_url }}" text-align="center"/>
    </div>


<br />

- **Payload, Encoders, Nops**: 

	- Los **payloads** consisten en código que se ejecuta en la máquina en la que se ha explotado una vulnerabilidad para conseguir algún tipo de beneficio (abrir una términal, información, cifrar, etc).

	- Los **Encoders** son módulos que se aseguran de mantener la integridad del payload mientras llega a su destino.
	
	- Los **Nops** mantienen el tamaño del payload consistente a través de los distintos intentos de Metasploit.

<br />

- **Otros**: Metasploite te da la opción de cargar módulos  de forma externa en "runtime" (cuando el programa se está ejecutando) con el modificador **-m**. 

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117192944.png' | relative_url }}" text-align="center"/>
    </div>


	También se dispone de el comando **loadpath** para cargar módulos en las rutas adecuadas.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220117192802.png' | relative_url }}" text-align="center"/>
    </div>



<br />

## 2.2. Fundamentos de Metasploit.

### 2.2.1. Interfaces de Metasploit. 

Una **interfaz** es un mecanismo o herramienta que conecta dos elementos. Existe un tipo concreto de interfaz que es la interfaz de usuario que ayuda a un humano a interactuar con un programa o dispositivo.

Existen muchas interfaces diferentes para utilizar Metasploit cada una con sus ventajas y desventajas. Además, es conveniente añadir que no existe una interfaz perfecta para usar con la consola de Metasploit, aunque MSFConsole es la única forma admitida de acceder a la mayoría de los comandos de Metasploit. Sin embargo, sigue siendo beneficioso conocer el resto de interfaces.

<br />

### 2.2.2. MSFCLI (msfcomandline).

El **msfcli** proporciona una potente interfaz de línea de comandos para el marco. Esto le permite agregar fácilmente exploits de Metasploit en cualquier secuencia de comandos que pueda crear.

**En versiones más recientes de Metasploit se eliminó msfcli como comando, una forma de obtener una funcionalidad similar es a través de msfconsole -x; concretamente sería:** 

```bash
kali@kali:$
msfconsole -x "\<msfcomand>"
```

<br />

### 2.2.3. MSFConsole.

La consola de MSFConsole es el la interfaz principal que existe para interactuar con el framework de Metasploit.  

Para más información:

https://www.offensive-security.com/metasploit-unleashed/msfconsole-commands/

<br />

### 2.2.4. Exploits.

Como hemos dicho en apartados anteriores, un 'exploit' es un tipo de módulo que va acompañado de otro tipo de módulo denominado 'payload'. En Metasploit hay dos tipos fundamentales de exploits: **Activos y Pasivos**.

<br />

#### 2.2.4.1. Exploits activos.

Un exploit activo explotará una vulnerabilidad de un host específico, se ejecutará hasta copletarse y finalmente terminará. Por ejemplo:

- Los módulos **Brute-Force** se irán cuando una shell quede abierta para la víctima.
- La ejecución de los módulos parará si encuentran un error.
- Puedes forzar a un módulo activo al background con el modificador **-j** al comando *run*.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220119191744.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 2.2.4.2. Exploits Pasivos.

Los exploits pasivos esperan a los hosts entrantes y los explotan a medida que se conectan.

- Los exploits pasivos casi siempre se centran en clientes como navegadores web, clientes FTP, etc.
- También se pueden usar junto con exploits de correo electrónico, esperando conexiones.
- Los exploits pasivos informan shells a medida que ocurren se pueden enumerar pasando '-l' al comando de sesiones. Pasar '**-i**' interactuará con un shell.

<br />

#### 2.2.4.3. Uso de exploits.

Para utilizar los exploits se cuenta con una serie de comandos entre los cuales se encuentran, dado un exploit:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220119233833.png' | relative_url }}" text-align="center"/>
</div>

Además hay una serie de comandos entre los que están:

- **show**: Show es un comando que te puede enseñar todo aquello que esté relacionado con un determinado término, entre los que están: targets, payloads, options, advanced, evasion.

	- **targets**: Sistemas operativos compatibles.
	- **payloads**: Payloads
	- **options**: Opciones del módulo y del payload si está.

<br />

### 2.2.5. Payloads.

Un módulo payload en Metasploit es un fragmento de código que acompaña a los exploits y se ejecuta tras la finalización del exploit que explota la vulnerabilidad y es el encargado de manipular la máquina víctima para el atacante cifrando, devolviendo una shell, etc. A nuestro nivel sólo se utilizaran en conjunto con los exploits.

<br />

## 2.3. Bases de datos.

### 2.3.1. Intro y bases de datos.

Metasploit tiene configurado soporte para el sistema de base de datos PostgreSQL con la finalidad de que el usuario pueda tener un acceso rápido y fácil a todala información relacionada con la prueba de penetración. El sistema permite un acceso rápido y fácil a la información de escaneo y podemos importar y exportar resultados de escaneo de varias herramientas de terceros. Lo importante es que mantiene nuestros datos limpios y organizados.

Algunos de los comandos asociados a la base de datos de Metasploit son:

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220121164120.png' | relative_url }}" text-align="center"/>
</div>

<br />

Por ejemplo; para saber el estado de la base de datos introducimos: **db_status**. Si todo va bien debería de aparecer:

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220121164325.png' | relative_url }}" text-align="center"/>
</div>

<br />

Junto con los comandos aparecen las distintas bases de datos que están disponibles y que son accesibles desde el prompt: 

- **hosts**: Guarda información sobre hosts.
- **loot**: Guarda información "robada" de la máquina víctima.
- **notes**: Guarda notas.
- **services**: Lista todos los servicios con los que nos hemos puesto en contacto en máquinas víctimas.
- **vulns**: Lista información sobre vulnerabilidades presentes en máquinas con las que nos hemos puesto en contacto.

- **workspace**: Lista los espacios de trabajo disponibles.

Podemos obtener más información sobre cómo interactuar adecuadamente con cada una de estas bases de datos escribiendo **help \<comando>** o **\<comando> -h**. Por ejemplo:

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220121170052.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 2.3.2. Utilizando las bases de datos.

#### 2.3.2.1. Montando la base de datos.

En Metasploit hay que montar el servidor postgresql antes de empezar a utilizar la base de datos. Desde la kali, introducimos:

```bash
systemctl start postgresql
```

Y seguidamente:

```bash
sudo msfdb init
```

<br />

Para cerciorarnos de que la base de datos está bien construida ponemos:

```bash
msf > db_status
```

tal y como hemos indicado en la imágen anterior.

<br />

#### 2.3.2.2. Workspaces.

Los workspaces son un mecanismo de Metasploit para organizar diferentes trabajos, datos, etc en regiones distintas con la finalidad de organizar un poco mejor la información relativa a cada proyecto. Cada workspace tiene sus propias bases de datos hosts, vulns, notes...

Con el comando **workspaces** desde la msfconsole se desplegará la lista de directorios de trabajo y con el comando **workspace** operaremos con cada base de datos individualmente gracias a los siguientes modificadores:

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220121172450.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 2.3.2.3. Escaneando e Importando información.

Metasploit permite al usuario interactuar con determinadas aplicaciones externas (nmap, nikto, nessus, etc) a través del marco de trabajo del propio metasploit. Además, podemos importar el report generado por la aplicación (siempre y cuando tenga un formato que Metasploit soporte) en las bases de datos de nuestro directorio de trabajo del marco de trabajo. 

Por ejemplo: En primer lugar ejecutamos un escaneo con Nmap:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122152147.png' | relative_url }}" text-align="center"/>
</div>

En la imágen podemos comprobar que hemos al final añadido el modificador **-oX** para indicar que el formato que se expida tendrá que hacerlo en XML (aunque existen un montón de opciones para cambiar el formato de expedición del report).

Una vez ha terminado, acudimos la ruta donde está nuestro informe y se lo pasamos como argumento al comando db_import del entorno de Metasploit:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122152404.png' | relative_url }}" text-align="center"/>
</div>

Y como la información que se ha recopilado es información acerca de hosts lo más razonable es que toda esta información se haya almacenado en la tabla **hosts** dentro del workspaces en el que hemos hecho el import.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122152534.png' | relative_url }}" text-align="center"/>
</div>

Aunque en realidad la información se divide entre las distintas tablas disponibles del workspace, por ejemplo, en **services** obtenemos:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122152645.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 2.3.2.4. Exportar datos.

También podemos crear copias de seguridad de nuestros datos utilizando el término **db_export**. 

<br />

# 3. Metasploit Avanzado.

## 3.1. Comandos Importantes.

Es conveniente saber el uso de los siguientes comandos.

- **show targets**: Dado un 'exploit' concreto, show targets es un comando que muestra el sistema operativo para el que va dirigido dicho exploit.
- **set target**: Dado un exploit y habiendo listado los targets con el comando anterior este comando permite asociar al exploit un target.
- **set RHOST/RPORT**: Concede un valor a las variables RHOST/RPORT. Hay variables a las que sólo se tiene acceso cuando se carga un módulo concreto (por ejemplo un payload o un auxiliary).
- **show/setpayloads**: Respectivamente, dado un exploit, muestra y selecciona un payload para utilizar con el exploit seleccionado.

<br />

## 3.2. Meterpreter.

### 3.2.1. Concepto.
Meterpreter es un payload que utiliza una inyección de memoría DLL para ofrecer una shell interactiva que no es necesariamente una consola del sistema operativo, sino una shell propia con comandos intrínsecos de Meterpreter en la que el atacante puede explorar la máquina víctima y ejecutar código e incluso abrir una consola del sistema operativo.

Al utilizar Meterpreter inyección de DLL en memoria opera directamente sobre la memoria y no escribe nada en el disco duro. No  se crean procesos nuevos ya que al operar directamente sobre memoria interactúa sobre procesos existentes y por tanto deja poca huella.

<br />

### 3.2.2. Uso de Meterpreter.
Un framework es un entorno de trabajo que contiene variables, archivos y características que un determinado programa puede utilizar para usarlo correctamente. Meterpreter genera su propio framework (es un subframework del framework de Metasploit), y eso significa que tiene sus propias variables de entorno, comandos ejecutables, etc. Veámos algunos comandos importantes asociados al entorno de Meterpreter y de Metasplotable asociados al uso de Meterpreter.

- **help**: help muestra una lista de comandos que se pueden utilizar y una descripción de los mismos.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220118123712.png' | relative_url }}" text-align="center"/>
    </div>

	Además, pasándole un comando como argumento a help te despliega una ayuda sobre cómo utilizar dicho comando:

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220118124934.png' | relative_url }}" text-align="center"/>
    </div>

	<br />
	
- **sessions**: Una sesión es una terminal que tienes abierta en la máquina víctima. Puedes tener varias sesiones abiertas en función del número de veces que has ejecutado el exploit (puedes ejecutar el exploit y meter la sesión en segundo plano con *background* y volver a ejecutar el exploit). 

	Con el comando *sessions* podemos interactuar con las distintas sesiones que tenemos abiertas, no desde la terminal de meterpreter.

	```meterpreter
	sessions <id>
	```

	Desde el framework de Metasploit podemos también utilizarlo siempre y cuando tengamos sessiones corriendo en segundo plano.
	
	Recordamos: 
	
	- Meterpreter: bg
	- Non-meterpreter: Ctrl + Z

<br />

- **background**: El comando backgroudn se utiliza para mandar una session a segundo plano. De esta forma la session sigue activa pero no bloquea nuestra terminal y nos permite seguir operando con Metasploit para por ejemplo abrir otra session.

<br />

- **show sessions**: Desde Metasploit podemos listar el número de sessiones que tenemos abiertas con meterpreter 

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220118131737.png' | relative_url }}" text-align="center"/>
    </div>

	Como podemos ver en la imágen nos lista las sesiones y nos da información sobre las mismas, el tipo de sesión, usuario, el host, host origen, etc.

<br />

- **multi/handler**: El módulo *multi/handler* es un exploit de tipo **handler**. 

	Los handler son modulos que proveen todas las caracterísitcas del sistema de payloads de Metasploit a exploits que han sido lanzados desde fuera del marco de trabajo. Es decir, es un mecanismo que utilizamos para dependiendo del payload:

	1. Escuchar una conexión por parte del payload (reverse payload).

	2. Iniciará una conexión contra un host en un puerto especificado. (bind payload).

	**Antes de levantar un multi/handler hace falta configurar adecuadamente el payload que definirá el tipo de conexión que se tiene levanta o que se quiere levantar contra la máquina víctima**. Es decir, que si nosotros estamos escuchando un payload configurado como meterpreter_reverse__tcp, el multihandler debe tener configurado un payload equivalente al de esta conexión para que sea compatible.

	Para mandar el multihandler primero debemos haber iniciado una conexión previa con otro exploit y de ahí, con la primera session en el background, configurar las variables del exploit con options y ejecutar el módulo como un "job".
	
	```bash
	run -jz
	```
	
	Los jobs o tareas son procesos que se lanzan y quedan en la máquina víctima corriendo en segundo plano. En este caso, el handler quedará corriendo como una tarea que se quedará escuchando peticiones en un puerto.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220119164920.png' | relative_url }}" text-align="center"/>
    </div>
	
	Evidentemente la sessión no se crea porque se ejecuta en segundo plano como un job. 

	Se puede ver los jobs activos mediante el comando **jobs**.

<br />

# 3.Metasploit: Sistemas y Redes.

## 3.1. Intro.

Metasploit tiene configurado soporte para el sistema de base de datos PostgreSQL con la finalidad de que el usuario pueda tener un acceso rápido y fácil a todala información relacionada con la prueba de penetración. El sistema permite gestionar de mejor manera la información que se almacena en cada prueba de penetración.

La base de datos se estructura en espacios de trabajo que tienen sus propias tablas de información. De esta forma la información se divide en función del proyecto al que esta pertezca y dentro de cada proyecto, se divide en función del tipo de información que es: **host**, **loot**, **services**, etc. 

<br />

## 3.2. Gestionando la base de datos.

### 3.2.1. Preparación. 

La base de datos que está asociada a Metasploit utiliza el servicio PostgreSQL. Antes de comenzar a gestionar la base de datos hay que tener activo el servicio de postgresql:

```bash
systemctl start postgresql
```

```bash
msfdb init
```

Comprobamos que está todo correcto con el comando:

```bash
msf > db_status
```

y si está todo correcto debería de informar que está conectada a la base de datos de Postgresql.

<br />

### 3.2.2. Workspaces.

Los **workspace** son divisiones que se hacen de la información y que están orientados a dividir los proyectos de pentesting que se hacen con metasploit. Con **workspace** listamos todos los workspaces disponibles.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122185202.png' | relative_url }}" text-align="center"/>
</div>

Podemos crear, añadir y en general interactuar con los espacios de trabajo con la ayuda de la información que aparece tras el modificador -h.


<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122185433.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 3.2.3. Importar y exportar datos.

Si tenemos ficheros con información que hemos recopilado utilizando aplicaciones externas (nmap, nikto, Nessus, etc) podemos importarlos para que Metasploit procese la información que contienen a través del formato de fichero que son y reparta dicha información entre las distintas bases de datos mencionadas anteriormente.

El comando es **db_import**:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122191250.png' | relative_url }}" text-align="center"/>
</div>

En la imágen anterior podemos ver cómo se usa el comando en cuestión y que formatos de ficheros están permitidos. El más común es xml aunque conviente observar que también se encuentra  Nessus XML, Nmap XML, Nikto XML. 

Veamos varios ejemplos:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122230933.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122231000.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220122231018.png' | relative_url }}" text-align="center"/>
</div>

<br />

Aquí utilizamos Nmap para generar un análisis y guardar el informe en un formato xml gracias al modificador: **-oX**. Este informe posteriormente es recuperado por Metasploit al importar el fichero y podemos ver cómo la información se reparte entre las bases de datos del espacio de trabajo.

Por otra parte, tenemos también el comando db_nmap utilizado de la siguiente manera:

```bash
db_nmap <comando nmap>
```

Que guardará los resultados obtenidos directamente en las bases de datos de tu directorio de trabajo sin necesidad de importat ningún archivo.


<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220123003356.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220123003115.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220123003540.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220123003635.png' | relative_url }}" text-align="center"/>
</div>

<br />

En este ejemplo generarmos un informe con nessus, lo exportamos con Nessus en formato .nessus y el fichero resultante lo importamos desde Metasploit.

<br />

### 3.2.4. Tablas de las bases de datos.

Como ya mencionábamos antes para cualquier espacio de trabajo existen tablas en los que se almacena la información que se recopila o se importa desde ese directorio de trabajo:

- **hosts**: Información de los hosts.
	- Información: **host -h**
	-  Ver columnas: **host -c \<IP>**
	-  Añadir host: **host -a \<IP>**
	-  Buscar host: **host -S**
	- Cambia la variable RHOSTS al resultado de la búsqueda -> **host -R**
	- Exportar a CSV -> **-o ruta.csv**

- **Servicios** -> Información de protocolos y servicios detectados.
	- Información: **services -h**
	- Ver columnas -> **-c**
	- Ver puertos -> **-p**
	- Añadir servicios -> **services -a**
	- Buscar Servicios -> **services -S**

- **Credenciales** -> Cookies o usuario:contraseña. 
	- Añadir a mano -> **creds -a IP -p PUERTO -u USUARIO -P PASSWORD**

- Vulnerabilidades (vulns): 
	- Buscar vulnerabilidad: **vulns -S MS11-030**

- Notas (notes)
- Loot (Hashes)

<br />

## 3.3. Módulos asociados a aplicaciones/servicios.

### 3.3.1. Complementos de Metasploit.
Existen módulos que están estrechamente relacionadas con determinadas aplicaciones informáticas o servicios. 

El comando **load** podemos cargar en el entorno de metasploit complementos de aplicaciones como por ejemplo: **Nessus**, **Openvass**. Después de haber cargado con *load* bien Nessus o bien OpenVas.

- **Nessus**: 

	```bash
	nessus_connect username:password@127.0.0.1:8834
	```
	
- **OpenVas**: 

	```bash
	openvas_connect username password 127.0.0.1 9390 
	```

<br />

### 3.3.2. Módulos auxiliares interesantes.
Ahora vamos a ver una serie de módulos auxiliares que van a hacer uso de servicios o aplicacions externas para efectuar acciones que pueda ayudarnos a aprovechar una vulnerabilidad más allá de un simple exploit de penetración, como por ejemplo: listado, escaneos, ataques de fuerza bruta en distintos protocolos. Veámos algunos de ellos divididos por servicios.

Si nosotros escaneáramos una máquina con nmap y obtuviésemos que tiene puertos abiertos podríamos probar a recopilar información utilizando los siguientes módulos en función del servicio que riga el puerto.

1. **MySQL**.

	
	**Fuerza bruta utilizando fichero PASS_FILE** ->  auxiliary/scanner/mysql/mysql_login
	
	Podemos ver que configurando las opciones y corriendo el exploit auxiliary obtenemos:
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125150831.png' | relative_url }}" text-align="center"/>
    </div>

	Hay un logeo con username root y sin contraseña.
	
	<br />

	**Conecta con credenciales y hace una enumeración de ficheros** -> auxiliary/admin/mysql/mysql_file_enum
	
	Esto en primera instancia se conecta con un usuario y una contraseña y si tiene éxito en el logeo lleva a cabo una enumeración de ficheros a través de una lista que se le pasa como una variable. Sería por tanto conveniente tener unas credenciales de acceso antes de utilizar este módulo.


    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125152301.png' | relative_url }}" text-align="center"/>
    </div>
	
	<br />
	
	**Volcado de esquema de todas las tablas de la base de datos** -> auxiliary/scanner/mysql/mysql_schemadump
	
	Realiza un volcado de las tablas de la base de datos. Funciona mejor con credenciales pero es algo prescindible.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125153254.png' | relative_url }}" text-align="center"/>
    </div>
	
	<br />
	
2. **FTP**.

	**Fuerza bruta usando determinadas características** -> auxiliary/scanner/ftp/ftp_login 

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125161322.png' | relative_url }}" text-align="center"/>
    </div>
	
	Podemos observar que entre las opciones hay variables muy interesantes como BLANCK_PASSWORDS, THREADS, BRUTEFORCE_SPEED, VERBOSE... y más, conviene siempre mirar las opciones.
	
	Por ejemplo, configurando:
	
	BLANCK_PASSWORDS -> true
	USERNAME -> Anonymous
	PASSWORD ->
	
	Obtenemos:

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125161733.png' | relative_url }}" text-align="center"/>
    </div>
	
	Y ya tenemos una contraseña lista.
	
	<br />

3. **Samba**: Samba es un servidor que opera sobre el protocolo SMB y que se utiliza para conectar.
	    
	**Ver los recursos (pipes) compartidos** -> auxiliary/scanner/smb/pipe_auditor 	
	
	<br />
	
	**Ver los recursos Samba compartidos** -> auxiliary/scanner/smb/smb_enumshares
	
	<br />
	
	**Listado de usuarios del sistema** -> auxiliary/scanner/smb/smb_enumusers
	
	<br />
	
	**Fuerza bruta de Samba usando PASS_FILE** -> auxiliary/scanner/smb/smb_login
	
	De nuevo, se trata de la misma lógica que en ataques de fuerza bruta de cosas anteriores.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125170435.png' | relative_url }}" text-align="center"/>
    </div>

	<br />

4. **SSH**.

	**Fuerza bruta con SSH** -> auxiliary/scanner/ssh/ssh_login
	
	De la misma forma que hemos operado en el resto de casos anteriores, se ajustan los parámetros y se lanza el módulo.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125192059.png' | relative_url }}" text-align="center"/>
    </div>
	
	Aparte de este existen más comandos SSH que realizan más o menos las mismas funcionalidades que en los casos anteriores, fuerza bruta, enumeración de usuarios, recopilación de información etc.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125192259.png' | relative_url }}" text-align="center"/>
    </div>
	
	<br />
	
5. **Telnet**.

	**Fuerza bruta Telnet** -> auxiliary/scanner/telnet/telnet_login    
	
    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220125194004.png' | relative_url }}" text-align="center"/>
    </div>

	<br />
	
6. **VNC**.

	**Fuerza bruta de VNC usando PASS_FILE** -> auxiliary/scanner/vnc/vnc_login

	De nuevo, otro ataque de fuerza bruta:

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220126144954.png' | relative_url }}" text-align="center"/>
    </div>


<br />

## 3.3. Servidores falsos.
Otras de las herramientas que podemos utilizar con metasploit es abrir un servidor falso en nuestra máquina y de esa forma montar una treta para que un individuo 'cliente' se conecte y poder así perjudicarle. 

Los **servicios disponibles** son: 

 <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220126150810.png' | relative_url }}" text-align="center"/>
    </div>


La mayoría son iguales, veámos algún ejemplo. En las opciones, las variables básicas son:

  <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220126180803.png' | relative_url }}" text-align="center"/>
    </div>

De entre las cuales las más importantes,  están **RedirectURL**, **SRVPORT**, **SRVHOST**, etc.

Por ejemplo, introducimos RedirectURL= https://www.google.com de manera que, cuando corremos:

 <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220126182124.png' | relative_url }}" text-align="center"/>
    </div>

Y al conectarnos desde la máquina obtenemos:

 <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220126182539.png' | relative_url }}" text-align="center"/>
    </div>

Nos piden usuario y contraseña y se nos ofrece, en este caso están en ambos en blanco. 

Otros módulos del mismo poseen más opciones como modificar las contraseñas, cambios de protocolo etc. 

 <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220118101845.png' | relative_url }}" text-align="center"/>
    </div>


<br />

# 4. Metasploit: Aplicaciones web.

## 4.1. Intro.
Se tiene que Metasploit tiene configuradas tanto herramientas configuradas a nivel de entorno (comandos: db_nmap, ...) como a nivel de módulos (exploits, payloads, auxiliary, ...).

<br />

## 4.2. WMap.
### 4.2.1. Intro.
**WMAP** es un escáner de vulnerabilidades de aplicaciones web rico en funciones que se creó originalmente a partir de una herramienta llamada SQLMap. Esta herramienta está integrada con Metasploit y nos permite realizar escaneos de aplicaciones web desde la consola de Metasploit. 

<br />

### 4.2.2. Entendiendo WMap.
Primero creamos una nueva base de datos para almacenar los resultados de nuestro escaneo WMAP, cargamos el complemento **wmap** con **load**.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127111031.png' | relative_url }}" text-align="center"/>
    </div>


y ejecutamos la ayuda con **help** para ver qué nuevos comandos están disponibles para nosotros. Entre los distintos apartados que obtenemos con la ayuda tenemos el siguiente:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127111225.png' | relative_url }}" text-align="center"/>
    </div>

Todos estos comandos contribuyen a formar parte de una configuración holística de la herramienta. Así, tenemos por ejemplo la forma de hacer las siguientes acciones:

-> **Añadir sitio web**: wmap_sites -a http://IP 
-> **Listado de sitios**: wmap_sites -l
-> **Añadir URL**: wmap_targets -t http://IP/path
-> **Ver los plugins**: wmap_run -t
-> **Ejecución WMAP**: wmap_run -e
-> **Listado de vulnerabilidades**: wmap_vulns -l (Aunque puede haber información repartida en otras tablas de la base de datos).

Para entender mejor esta información podemos ver un ejemplo.

<br />

### 4.2.3.  Ejemplo de WMap.

En este ejemplo vamos a encender la máquina Metasploitable y vamos a intentar escanear el sitio web Mutillidae de esta máquina.

Los pasos a seguir que tomamos son, añadir la dirección del servidor, añadir la URL del sitio web a analizar, ejecutar wmap: 

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127115632.png' | relative_url }}" text-align="center"/>
    </div>

Podemos explorar, una vez finalizado el proceso, los resultados con wmap_vulns -l:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127115948.png' | relative_url }}" text-align="center"/>
    </div>

<br />

## 4.3. Módulos auxiliares para aplicaciones webs.
Tenemos módulos auxiliares para ayudarnos a recopilar información sobre aplicaciones web ya sean ficheros webs, contraseñas, robots.txt, etc.

<br />

### 4.3.1. Escaneo de directorios o ficheros web.

Existen módulos que nos pueden facilitar el listado de directorios o ficheros dentro de un directorio web:

<br />

**Dirbuster sobre directorios** -> auxiliary/scanner/http/dir_scanner 

Puede utilizarse sobre sobre dominios pero lo más probable es que no funcione correctamente, lo más aconsejable es hacer el cambio de dominio a ip y de ahí ejecutar el módulo.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127140932.png' | relative_url }}" text-align="center"/>
    </div>

<br />

**Dirbuster sobre ficheros** -> auxiliary/scanner/http/files_dir

De nuevo, lista ficheros de un sitio web basándose en extensiones como .xml, .exe, etc.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127143133.png' | relative_url }}" text-align="center"/>
    </div>

<br />

**Busca en archive.org versiones anteriores de la web guardadas** -> 
auxiliary/scanner/http/enum_wayback

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127143833.png' | relative_url }}" text-align="center"/>
    </div>


<br />

Además, tenemos el **módulo de fuerza bruta para servidores http** visto en el tema anterior de este mismo módulo para metasploit.

<br />

### 4.3.2. Craking de contraseñas para aplicaciones web. 

En el contexto de la seguridad informática, el **cracking contraseñas** (descifrado de contraseñas) es el proceso por el cual se pretende recuperar una contraseña partiendo con la información que nos proporcionan datos almacenados o transmitidos por un dispositivo de forma codificada. 

Un enfoque común es intentar adivinar repetidamente la contraseña y compararla con un **hash criptográfico** disponible de la contraseña, es decir; la contraseña cifrada mediante un algoritmo de hash. Esta generalmente se encuentra almacenada e algún lugar del servidor y el obtenerla permite al atacante realizar múltiples pruebas offline hasta dar con la contraseña para luego hacer un único logeo en online y conseguir acceso a la máquina.

JOHN THE RIPPER OBSOLETO, INSTALAR DE GITHUB PARA KALI.

<br />

## 4.4. MSFVenom.

MSFVenom es una instancia de línea de comandos que se utiliza para generar y expedir todo tipo de payload y código shell disponible en Metasploit. Puede utilizarse con Android, Windows y Linux. Se utiliza con el siguiente comando desde la consola de kali:

```bash
$ msfvenom –p <payload> <LHOST> <LPORT> > <BIN>
```

Esto en esencia lo que hace es convertir un payload en un binario que se ejecutará en la máquina víctima para perjudicarla. Por ejemplo, un payload "backdoor" se pasa por MSFVenom para convertirse en binario y ser mandando a una máquina víctima done se ejecuta. Posteriormente el atacante utiliza un handler desde Metasploit para conectarse a la sesión y conseguir así acceso a una shell de la máquina víctima.

Tenemos diversas opciones:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127155329.png' | relative_url }}" text-align="center"/>
    </div>

Así por ejemplo no sólo tenemos la opción de utilizar payloads, podemos añadir encoders, time-outs, etc, para poder modificar personalmente nuestro ataque. Veamos un ejemplo:

En primer lugar, elegimos un payload y un puerto y montamos el ejecutable malicioso con MSFVenom:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127174730.png' | relative_url }}" text-align="center"/>
    </div>

Con **-p** indicamos que queremos utilizar un payload y con **-f** definimos el formato del binario construido, en este caso **.exe**. Seguidamente, utilizamos SCPWindows ara conectarnos a la kali y extraer el binario.

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127175113.png' | relative_url }}" text-align="center"/>
    </div>


Ahora, lo que hemos utilizado como payload es un reverse shell, este es un payload que consigue una shell que se abre en nuestra máquina kali que, a su vez, está escuchando la conexión de la máquina víctima que realiza la request de conexión. Por tanto, en primer lugar montamos el multi/handler con metasploit, configurando el payload adecuado (que coincida con el que nosotros hemos montado con msfvenom), el puerto y el host local y lanzamos:


<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127180452.png' | relative_url }}" text-align="center"/>
    </div>

Seguidamente ejecutamos el .exe en windows y:

<div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220127180652.png' | relative_url }}" text-align="center"/>
    </div>

Es importante observar que estamos, de nuevo, utilizando una reverse shell. Si estuviéramos utilizando una bind_shell el orden de la operación se invertiría, seríamos nosostros los que pediríamos una conexión a un puerto de la máquina víctima en escucha que nos concedería acceso una shell y por tanto, **primero ejecutaríamos el malware y luego nosotros correríamos el handler desde metasploit**, primero la máquina escucharía peticiones en un puerto y luego nosotros lanzaríamos una conexión a dicho puerto.


Este mismo resultado se puede repetir en cualquier sistema operativo siempre y cuando se genere un ejecutable con el formato adecuado.

<br />

# 5. Post-explotación.

## 5.1. Post-explotación.
### 5.1.1. Concepto.

En un ataque a una máquina denominamos como **Post-Explotación** a la etapa del ataque posterior a la explotación y ejecución del payload consistente en la recopilación de información sobre la máquina, el usuario, los contactos que mantiene el usuario y en general cualquier tipo de dato relevante que podamos encontrar en la misma.

La herramienta **meterpreter** de Metasploit dispone de una serie de módulos que se encargarán, según el sistema operativo, de automatizar la recopilación de información en distintos sectores del sistema operativo. Los dividimos en: **Generales**, **Linux**, **Android**.

<br />

### 5.1.2. Módulos generales.
La información la obtenemos de las páginas: 

https://www.offensive-security.com/metasploit-unleashed/windows-post-gather-modules/

https://www.offensive-security.com/metasploit-unleashed/windows-post-capture-modules

Destacamos cinco módulos: **env, firefox_creds, ssh_creds, execute, check_malware**.

- **env**: El módulo **env** es un módulo que recopila las variables de entorno del sistema operativo que corre en la máquina víctima.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120143121.png' | relative_url }}" text-align="center"/>
    </div>


	Sin embargo, si el comando no funciona podemos realizar una operación que nos devuelva un resultado similar pidiendo a meterpreter que nos abra una shell (según el sistema operativo que toque) y pasando el comando **printenv** en Linux o enseñar varible a variable en Windows.

<br />

- **firefox_creds**: El modulo *firefox_creds* recopila credenciales (contraseñas, cookies) relacionadas con firefox 

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120144932.png' | relative_url }}" text-align="center"/>
    </div>


	Tal y como indica, estos datos se almacenan en la base de datos: **loot**.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120145518.png' | relative_url }}" text-align="center"/>
    </div>


	**localizada en ~/.msf4/loot**.
	
<br />

- **ssh_creds**: Este módulo recopilará la información del directorio **.ssh** de la máquina víctima. Además, ficheros como knwon_hosts, authorized_keys también so descargados.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120145836.png' | relative_url }}" text-align="center"/>
    </div>


	De nuevo, almacena los datos en la base de datos **loot** y también podemos especificamente situarnos en esa subcarpeta para volcar los ficheros robados.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120150152.png' | relative_url }}" text-align="center"/>
    </div>


<br />

- **execute**: Meterpreter es una herramienta que abre su propio entorno con sus propios comandos que hace que meterpreter genere instrucciones que inyecta en la máquina vulnerable. No se trata de un acceso directo a una terminal del sistema operativo de la máquina objetivo.

	Para ello, para introducir comandos propios del sistema operativo, disponiemos del comando **execute**, que inyecta un comando propio del sistema operativo de la máquina en cuestión. Podemos por ejemplo crear un fichero:


     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120152500.png' | relative_url }}" text-align="center"/>
    </div>

     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120152515.png' | relative_url }}" text-align="center"/>
    </div>

	Aquí he creado un fichero mediante el comando *touch* y pasándole como argumento el nombre del fichero que es *TEST* mediante el modificador -a.
	
	Podemos desplegar las opciones de este comando con **execute -h**.

<br />

- **check_malware**: Este módulo nos permite analizar si un fichero es malware subiéndolo a VirusTotal. Se tiene que correr desde Metasploit, no desde meterpreter a modo de módulo tipo POST.

     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120155741.png' | relative_url }}" text-align="center"/>
    </div>

	Evidentemente podemos reproducir el mismo resultado descargándonos el fichero y subiéndolo manualmente a VirusTotal.


<br />

### 5.1.3. Módulos Windows.

Disponemos de una serie de módulos específicos para Windows.

- **keylog_recorder**: Se trata de una herramienta que captura las teclas pulsadas por el usario en el teclado conectado a la máquina víctima.


     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120161349.png' | relative_url }}" text-align="center"/>
    </div>

     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120161827.png' | relative_url }}" text-align="center"/>
    </div>

	Como podemos ver en la imagen, no se ha capturado ninguna tecla en el fichero especificado debido a que el teclado de una máquina virtual en ningún caso es físico. Pero ahí deberían aparecer los caracteres tecleados.

<br />

- **arp_scanner**: Efectúa un escaneo arp para ver qué máquinas (IP y MAC) se han puesto en contacto con la máquina. Se corre utilizando un módulo post:

	
     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120165858.png' | relative_url }}" text-align="center"/>
    </div>

	Pese a que lo que hace es realizar alguna prueba de conectividad, este comando deja poco rastro.
	
<br />

- **checkvm**: Este módulo checkea si la máquina víctima es o no una máquina virtual.

     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120170059.png' | relative_url }}" text-align="center"/>
    </div>

	Es conveniente observar que no ofrece ningún tipo de información acerca del sistema operativo que corre, sólo si es o no una máquina virtual. Esto además también se puede hacer comprobando la MAC del dispositivo, **08:00:27** es por ejemplo propia de virtualbox.

<br />

- **credential_collector**: Recopilar credenciales

     <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120171501.png' | relative_url }}" text-align="center"/>
    </div>

	Con este comando recopilamos los hashes de los usuarios y los tokens/objetos de autorización.

	Por motivos de seguridad, es posible que desee almacenar las contraseñas en formato hash. Esto evita la posibilidad de que alguien que obtenga acceso no autorizado a la base de datos pueda recuperar las contraseñas de todos los usuarios del sistema. 
	
	El **Hashing** realiza una transformación unidireccional en una contraseña, convirtiendo la contraseña en otra Cadena, llamada contraseña hash . "Unidireccional" significa que es prácticamente imposible ir hacia el otro lado: convertir la contraseña codificada nuevamente en la contraseña original. 

	**El valor de la contraseña hash no se cifra antes de que se almacene en la base de datos. Cuando un miembro intenta iniciar sesión, el módulo de personalización toma la contraseña proporcionada, realiza un hash unidireccional similar y lo compara con el valor de la base de datos. Si las contraseñas coinciden, entonces el inicio de sesión es exitoso.**

	También tenemos el modulo **hashdump**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120193257.png' | relative_url }}" text-align="center"/>
    </div>


<br />

- **enum_applications**: Listado de aplicaciones instaladas
    
    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120173756.png' | relative_url }}" text-align="center"/>
    </div>

	Y como se puede observar, el resultado se almacena en un fichero.

<br />

- **enum_logged_on_users**: Listado de usuarios que han iniciado sesión

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120171602.png' | relative_url }}" text-align="center"/>
    </div>

	Y de nuevo el resultado se almacena en un fichero.

<br />

- **enum_shares**: Listado de recursos compartidos. 

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120174644.png' | relative_url }}" text-align="center"/>
    </div>


<br />

- **usb_history**: Dispositivos USB conectados 

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120194110.png' | relative_url }}" text-align="center"/>
    </div>

<br />

- https://www.offensive-security.com/metasploit-unleashed/windows-post-manage-modules/

- **Borrado de usuario** -> post/windows/manage/delete_user USERNAME=hacker

<br />

## 1.4. Sistemas de Linux.

- Comprobar si es una VM ->**post/linux/gather/checkvm**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120194848.png' | relative_url }}" text-align="center"/>
    </div>

<br />

- Listado de ficheros de configuración de aplicaciones -> **post/linux/gather/enum_configs**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120220013.png' | relative_url }}" text-align="center"/>
    </div>

<br />

- Recopilar estado de red y equipos -> post/linux/gather/**enum_network**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120220037.png' | relative_url }}" text-align="center"/>
    </div>

<br />

- Software Blue Team -> post/linux/gather/**enum_protections**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120221948.png' | relative_url }}" text-align="center"/>
    </div>

<br />

- Información de sistema -> post/linux/gather/**enum_system**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120222443.png' | relative_url }}" text-align="center"/>
    </div>

<br />

- Registro de comandos (shell, mysql, vim...) -> post/linux/gather/**enum_users_history**

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220120223616.png' | relative_url }}" text-align="center"/>
    </div>

<br />

## 5.2. Persistencia.

### 5.2.1. Concepto.
En informática el término **persistence** o persistencia se refiere al estado de un sistema que sobrevive (persiste) más del proceso que lo ha creado. En lo referente al malware tiene que ver con cómo este persiste más alla de la terminación del proceso que lo ha crea cuando se apaga el ordenador, cuando se cambia de sistema operativo, etc.

<br />

### 5.2.2. Uso y ejemplo.

Metasploit y Meterpreter tienen una opción con la que podemos volver un malware persistente. Veamos un ejemplo de uso:

- En primer lugar, utilizamos la máquina WIndowsploitable y utilizamos un exploit contra ella, por ejemplo EternalBlue, con un payload de Meterpreter:

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220212121425.png' | relative_url }}" text-align="center"/>
    </div>

	<br />
	
- Seguidamente, utilizamos el comando que tiene meterpreter para generar persistencia:

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220212121710.png' | relative_url }}" text-align="center"/>
    </div>
	
	Observemos que esta opción lo que hace es generar un script en el que almacena un payload y que está constantemente ejecutando en una localización del sistema operativo. Dicho payload es: **windows/meterpreter/reverse_tcp**.
	
	<br />
	
- Así pues, supongámos que perdemos la sesión actual que tenemos abierta con Meterpreter. Entonces, sabemos que el payload que se ha guardado en el ordenador a modo de script es un reverse shell y por tanto podemos recuperar la conexioń levantando un **multihandler** y configurando el payload adecuadamente de forma que coincida con el payload que está pidiendo una conexión desde la máquina víctima.

    <div style="text-align:center">
	<img src="{{ 'assets/img/M6/Pasted image 20220212122208.png' | relative_url }}" text-align="center"/>
    </div>
	
	Y así habríamos recuperado la conexión. Observemos que de nuevo, es importante hacer coincidir ambos payloads.

	
