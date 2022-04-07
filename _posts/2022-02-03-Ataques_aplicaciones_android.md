---
layout: post
title: Ataques a aplicaciones Android
subtitle: Introducción al sistema operativo Android y sus aplicaciones.
tags: [redteam] 
---

# 1. Sistema operativo Android.

Android es un sistema operativo diseñado para teléfonos móviles que tiene un kernel de linux.

<br />

## 1.2. Fundamentos de las aplicaciones android.

### 1.2.1. Intro.
Recordamos que una aplicación no es más que un programa diseñado con una funcionalidad específica. Así, una aplicación andriod es una aplicación diseñada para el sistema operativo Android, específico de dispositivos teléfonos móviles.

<br />

### 1.2.2. Aspectos fundamentales de la aplicación.

#### 1.2.2.1. Paquetes APK.
Para escribir aplicaciones de Android, es posible utilizar los lenguajes Kotlin, Java y C++, lenguajes de programación orientado a objetos. 

Las herramientas de Android SDK compilan tu código, recursos y datos en un archivo APK; un paquete de Android (análogo a los paquetes .deb, o .rpm en Linux), con extensión **.apk**. En definitiva; **un archivo APK contiene todos los contenidos de una aplicación de Andriod y es el archivo que usan los dispositivos con tecnología Andriod para instalar la aplicación**. 

<br />

#### 1.2.2.2. Seguridad de las aplicaciones android.

Cada aplicación de android reside en su propia zona de pruebas de seguridad y está protegida por las siguientes características de seguridad de Android:

- El sistema operativo Android **es un sistema Linux multiusuario en el que cada aplicación es un usuario diferente**.

- De forma predeterminada, el sistema asigna a cada aplicación un ID de usuario Linux único. Este sólo es conocido por el sistema y es ignorado por la aplicación.

- El código de cada aplicación se ejecuta de manera independiente del resto en su propia máquina virtual.

- Cada aplicación se ejecuta en su propio proceso que se abre cuando Android lo requiere para trabajar con él y lo cierra cuando considera que este ya no es necesario.


Con estas reglas, el sistema Android implementa el **principio de mínimo privilegio**: cada aplicación tiene acceso tan sólo a los componentes que necesita para llevar a cabo su trabajo y nada más. Así, el alcance de una aplicación es límitado y no puede acceder a partes del sistema sobre las que no tiene permiso. Así, estas sólo se ejecutan dentro de una especie de sandbox.

<br />

#### 1.2.2.3. Conectividad entre aplicaciones android.

Sin embargo, y pese a lo anterior, es posible que las aplicaciones android compartan datos entre sí y con el sistema.

- Es posible que dos aplicaciones compartan el mismo ID de usuario de Linux para que cada una pueda acceder a los archivos de la otra. Además, también pueden compartir proceso y VM. Las aplicaciones que comparten el mismo ID deben de estar firmadas con el mismo certificado.

- Una aplicación puede acceder a los datos del usuario almacenados en el dispositivo mediante el **sistema de permisos**. Este no es más que un sistema de acceso basados en tokens (recordamos que un token es un objeto informático que representa el derecho a realizar una acción) que se conceden a la aplicación cuando se le concede **explícitamente** permiso para realizar una tarea (como mirar el correo, fotos, contactos, etc).

<br />

### 1.2.3. Componentes de la aplicación.

Denominamos como **componente de la aplicación** a bloques de creación (piezas de texto, gráficas, etc) que son parte de la aplicación a través de las cuales la misma desarrolla su funcionalidad. Distinguimos entre; Actividades, Servicios, Receptores de emisiones, Proveedores de contenido.

Se tratan en esencia de puntos de entrada hacia la aplicación para el usuario o el sistema y que permiten que la misma aplicación lleve a cabo su función correctamente. Un aspecto exclusivo del diseño del sistema Android es que cualquier aplicación puede iniciar un componente de otra aplicación



<br />

#### 1.2.3.2. Actividades.
 
Una *actividad* representa una pantalla individual con una interfaz de usuario a través de la cual el usuario interactúa con la aplicación. **Se trata de un punto de entrada para el usuario**.

Una actividad puede ocupar toda la pantalla del dipositivo o al contrario puede formar parte de otra actividad perteneciente a otra aplicación. Por ejemplo, una aplicación que necesite tomar una foto necesita llamar a la camara que desplegará su propia actividad miniaturizada en la pantalla del dispositivo. 

En general una actividad debe proporcionar las siguientes interacciones:

- Realizar un seguimiento de lo que realmente le interesa al usuario.

- Priorizar los procesos utilizados con anterioridad pues estos pueden contener información que interesa al usuario. 

- Ayudar a la aplicación a controlar la finalización de su proceso para que el usuario pueda regresar a las aplicaciones anteriores con el estado anterior restaurado.


**Por último mencionar que las actividades se implementan como subclases de la clase Activity**.

<br />

#### 1.2.3.3. Servicios

Denominamos como *servicio* al compenente que permite mantener la ejecución de la aplicación en segundo plano. Se trata obviamente de un **punto de entrada para el sistema**.

Como ejemplo podemos tomar que un reproductor de música que utiliza un servicio para ejecutarse en segundo plano y que el usuario siga escuchándo música mientras usa en primer plano una aplicación distinta.

El hecho de que estos no ofrecezcan interfaz de usuario hace que este último sea menos consciente de un proceso en segundo plano lo que ofrece al sistema la posibilidad de tratarlos con mayor libertad. Por ejemplo para pararlos en favor de procesos que son prioritarios para el usuario para luego reiniciarlos posteriormente, otro ejemplo típico de esto consiste en la trasferencia de datos con otro dispositivo. 

La principal razón por la que existen los procesos en segundo plano es debido al *enlace de procesos*, esto es; que hay procesos que corren en favor de otro proceso y que el sistema sabe que están relacionados para el correcto funcionamiento del último. El mismo sistema mantiene en segundo plano procesos como receptores de notificaciones, protectores de pantalla, etc.

Por último, los servicios se implementan como instancias de la subclase Service.

<br />

#### 1.2.3.4. Receptores de emisión.

El *receptor de emisión* es un componente que posibilita al sistema el entregar eventos a la aplicación (incluso fuera del flujo habitual), esto permite a su vez que la aplicación responda a anuncios de eventos de todo el sistema. **Se trata de otro punto de entrada para el sistema**.

El sistema puede entregar emisiones incluso a las aplicaciones que no estén en ejecución. Muchas de estas provienen del propio sistema para el indicar el status del harware, alarmas, etc.

Sin embargo, las aplicaciones también pueden iniciar emisiones de eventos con la finalidad de comunicarse con el usuario a través del dispositivo.

Por lo general un receptor de emisión es simplemente una puerta de enlace a otros componentes y que está diseñado para realizar una cantidad mínima de trabajo.

**Un receptor de emisión se implementa como una instancia de la subclase BroadcastReceiver**.

<br />

#### 1.2.3.5. Proveedores de contenido.

Un *proveedor de contenido* es un componente que administra un conjunto compartido de datos de la aplicación que puedes almacenar en un sistema de archivos, como una base de datos. De esta manera los datos que la aplicación produce quedan almacenados en un sitio al que luego pueden acceder otras aplicaciones si el proveedor de contenido así lo permite.

Se trata de un punto de entrada de la aplicación a un sistema de almacenamiento con la finalidad de guardar datos asociados a un nombre y que se identifica mediante un esquema URI (que no URL).

**Un proveedor de contenido se implementa como una instancia de una subclase de ContetProvider.** 

<br />

### 1.2.4. Activación de los componentes.


#### 1.2.4.1. Intent.
De los cuatro tipos de componentes, tres (actividades, servicios y receptores de emisión) se activan mediante un mensaje asíncrono denominado *intent*. 

El intent es una instancia de la clase Intent qeu define un mensaje que permite activar un componente específico (intent explícita) o un tipo específico de componente (intent implícita).

- Para actividades y servicios una intent define la acción que se va ha realizar y puede especificar el URI de los datos en los que debe actuar.
- En el caso de los receptores de emisión, la intent tan solo define el anuncio que emite pero no gestiona la forma en la que se emite el anuncio y la acción que se ve ha realizar

<br />

#### 1.2.4.2. Content Resolver.

En el caso del cuarto componente, los proveedores de contenido, no se activan con intents. En su lugar se activan mediante solicitudes de un ContentResolver. Este aborda todas las transacciones con el proveedor de contenido. 

<br />

### 1.2.5. El archivo manifiesto.

#### 1.2.5.1. Información general.

El archivo **manifest** de una aplicación android contiene metadatos importantes sobre la aplicación. La aplicación debe declarar todos sus componentes en esta aplicación, es decir; para que el sistema android pueda iniciar un componente de la aplicación, debe reconocer la extencia de ese componente leyendo el archivo de manifiesto de la aplicación.

Además, el manifiesto sirve para:

- Identificar los permisos de usuario que requiere la aplicación.
- Declarar el nivel de API mínimo (en función de las API, interfaz entre aplicaciones, que la aplicación usa). Este es un valor entero que identifica la revisión del marco de trabajo de las APIs.
- Declarar características de hardware y software que la aplicación usa o exige (SO, cámara, etc).
- Declarar las librerias APIs a las que la aplicación tiene que estar vinculada. 

<br />

#### 1.2.5.2. Declaración de componentes.

La tarea principal del manifiesto es informarle al sistema acerca de los componentes de la aplicación. Por ejemplo;

```xml
<?xml version="1.0" encoding="utf-8"?>  
<manifest ... >  
	<application android:icon="@drawable/app_icon.png" ... >  
				<activity android:name="com.example.project.ExampleActivity"
						  android:label="@string/example_label" ... >  
				</activity> 
		... 
	</application>  
</manifest>
```

En el fragmento de código anterior un manifest declara un componente de tipo actividad. De entre las etiquetas aquí presentes destacamos:

- \<manifest>, que se utiliza para declarar el manifiesto.
- \<application>, esta se utiliza junto con un atributo "android:icon" que señala / identifica a la aplicación.
- \<activity>, que identifica el componente a declarar. Además de \<activity> podemos encontrar \<service>, \<receiver>, \<provider>.

**Si incluyes actividades, servicios y proveedores de contenido en tu archivo de origen, pero no los declaras en el manifiesto, no estarán visibles para el sistema y, por consiguiente, no se podrán ejecutar**.


No obstante, los receptores de emisión pueden declararse en el manifiesto o crearse dinámicamente en forma de código como objetos BroadcastReceiver] y registrarse en el sistema llamando a registerReceiver().

<br />

#### 1.2.5.3. Declaración de requisitos de la aplicación.

Existen muchos dispositivos con tecnología Android, pero no todos ofrecen las mismas funciones y capacidades. A fin de evitar que una aplicación se instale en dispositivos que no tienen las funcionalidades que dicha aplicación necesita, es importante que definir claramente un perfil tecnológico para los dispositivos que pueden utilizar una aplicación concreta.

```xml
<manifest ... > <uses-feature android:name = "android.hardware.camera.any" android:required = "true" /> <uses-sdk android:minSdkVersion = "7" android:targetSdkVersion = "19" /> ... </manifest>
```

Así, con dicha declaración, los dispositivos que no tengan cámara o que tengan una versión de android anterior a 2.1  no podrán descargarse la aplicación.

<br />

### 1.2.6. Recursos de la aplicación.

En Android, una aplicación está compuesta por más que código; requiere de **recursos independientes del código fuente**, como imágenes, archivos de audio, etc. Estos recursos presentan la gran ventaja de facilitar la actualización de muchas caracerísticas de la misma sin la necesidad de alterar todo el código.

Para cada recursos que se añade en el proyecto de tu aplicación, las herramientas de compilación SDK definene un ID con número entero único que puedes usar para hacer referencia al recursos desde tu código de aplicación.

<br />

## 1.3. Genymotion e interactuación con Android.

### 1.3.1. Genymotion.

Genymotion Desktop es un emulador de Android que incluye un conjunto completo de sensores y funciones para interactuar con un entorno virtual de Android

Actúa con Virtualbox, aunque lo más recomendable es no instalarse la versión de Virtualbox que este te ofrece puesto que suele estar anticuada.

También es importante revisar la compatibilidad de Windows con este virtualizador pues la característica: **Plataforma de máquina virtual** y la plataforma del hipervisor de windows tienen que estar desactivadas.

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220119114821.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 1.3.2. Adb y Fastboot.

Las herramientas **adb** y **fastboot** son comandos que te permiten tomar el control de un teléfono android desde un PC vía USB. Son necesarios en algunso procesos de modificación y pueden ser muy útiles en un evento de crasheo de terminal o "crash" del dispositivo.

- **Fastboot**: Fastboot es un aplicación de kali que nos permite interactuar con los mecanismos del instalado, encendido/apagado de un dispositivo android, (por ejemplo bloquear o desbloquear el 'bootloader', un programa encargado de arrancar el sistema al iniciarse).

	Se trata de una aplicación que se encuentra como paquete dentro de la apt y que por tanto puede descargarse mediante un comando como:

	```bash
	sudo apt-get install fastboot
	```

	En Fastboot podemos encontrar los siguientes subcomandos importantes:

	- **Fastboot devices**: Con este comando comprobamos que el dispositivo esté correctamente conectado al ordenador.
	- **Fastboot oem unlock**: Este comando se usa para desbloquear el cargador de arranque y va seguido de la clave que generalmente debe proporcionar el fabricante.
	- **Fastboot flash**: Este comando va a compañado de una serie de subopciones con los que podemos cargar el teléfono. Antes de continuar debemos saber que: flashear es escribir en memoria flash, una memoria flash es una memoria que puede borrarse y reprogramarse eléctricamente como un USB. Escribir en memoria flash permite por ejemplo instalar un nuevo sistema operativo:

		Un ejemplo sería flash Partitions.
		
<br />

- **adb**: Esta herramienta, abd, es la que nos ayudará a realizar tareas dentro del sistema telefónico como copiar archivos, hacer copias de seguridad y demás. De nuevo, está disponible en la apt y podemos instalarlo mediante un comando como:

	```bash
	sudo apt-get install abd
	```

	Entre los principales subcomandos podemos encontrar:

	- **connect**: Para conectarse un dispositivo movil al ordenador.
	- **devices**: Con este comando comprobaremos si el dispositivo está o no correctamente conectado al dispositivo.
	- **push/pull**: Los cuales nos permitirán respectivamente enviar o descargar archivos del dispositivo.
	- **install/uninstall**: Instalará aplicaciones en el smartphone.
	- **shell**: El comando cargará una shell desde la que controlar el dispositivo. Dentro de una shell nos moveremos dentro de un sistema operativo que no es linux y que por tanto tiene comandos distintos. Entre los más utilizados están
		- **pm list packages**: Listados 
		- **pm path \<paquete>**: Ruta donde está el paquete.
		- **getprop**: Propiedades del sistema.
		- **dumpsys**: Vuelva todo el contenido del sistema.
		- **su**: Cambiamos a root.

<br />

### 1.3.3. Apktool.

Apktool es una herramienta que nos permite desempaquetar y empaquetar archivos como .apk.

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112124555.png' | relative_url }}" text-align="center"/>
</div>


<br />

## 1.4. Comandos de análisis estático.


### 1.4.1. Jadx-gui.

La herramieta **jadx-gui** es un decompilador que traduce el documento classes.dex a código ava para poder examinar de forma estática una aplicación. 

Al ejecutar este comando desde la terminal se abre una interfaz gráfica que nos da acceso a la aplicación y dentro de esta debemos abrir el paquete de la aplicación escrito en lenguaje Java.

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112135652.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112135719.png' | relative_url }}" text-align="center"/>
</div>

Desde navegación podemos buscar una clase en concreto. Esto nos puede facilitar mucho la búsqueda.

<br />

## 1.4 Ejercicio Insecurity Bank.

### 1.4.1. Synopsis.
Teemos el paquete "apk" de una aplicación android "InsecureBankv2". Como ya hemos visto el paquete .apk de una aplicación contiene el código a compilar para instalar correctamente la aplicación (código fuente, librerías estáticas...).

Vamos a desempaquetar el apk para tener acceso a su código fuente, analizarlo, buscar algún parámetro para volver maliciosa la aplicación, reempaquetar el código fuente de vuelta en un archivo .apk y seguidamente firmarlo para que parezca un paquete apk normal.

<br />

### 1.4.2. Pasos.
En primer lugar tenemos el .apk descargado en kali. Vamos a llevar a cabo un análisis estático del paquete APK para poder examinar su código en busca de puntos débiles.

Para ello empleamos la herramienta **jadx-gui** y buscamos el fichero LoginActivity utilizando la pestaña *Navegación*. En él, vemos una línea de código en el que analiza el valor de un elemento string cuyo nombre es "is_admin" y vemos que evalua con un condicional "if" si tiene por valor no. 

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112135940.png' | relative_url }}" text-align="center"/>
</div>

De esta manera ya sabemos que existe un elemento que se denomina is_admin puesto por defecto en no que probablemente permita a la aplicación ejecutarse en modo administrador. Así, vamos a buscar ese elemento en el .apk y cambiar dicho valor de "no" a "yes". 

Procedemos a desempaquetarlo con apktool:

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112135142.png' | relative_url }}" text-align="center"/>
</div>

El código fuente de esta aplicación está dentro de la carpeta smali. De todas formas, buscaremos en todos los ficheros de la carpeta InsecureBankv2 en busca del término "is_admin".

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112141754.png' | relative_url }}" text-align="center"/>
</div>

Con este comando primero buscamos todos los ficheros que pueda haber en la carpeta actual, y esa lista de rutas de ficheros se la pasamos a xargs que cogera cada línea y se lo pasará como un argumento al comando que le indiquemos, en este caso, grep.

Podemos comprobar así que en el fichero **strings.xml** tenemos el elemento xml tipo string con valor no. Vamos a dicho fichero, buscamos el elemento y le cmbiamos el valor

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112142246.png' | relative_url }}" text-align="center"/>
</div>

Una vez llevada a cabo la modificación, vamos a la carpeta de Software y volvemos a empaquetar la carpeta como un APK. Y seguidamente lo firmamos con un firmador de aplicaciones hechas con Java. En este caso, nos valdrá con utilizar uber-apk-signer que puede descargarse desde Github: https://github.com/patrickfav/uber-apk-signer/releases/

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112143245.png' | relative_url }}" text-align="center"/>
</div>

Y ya tendríamos nuestra apk modificada y lista para utilizar en un dispositivo android.


También po 

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220110131500.png' | relative_url }}" text-align="center"/>
</div>

enjarify -o classes.jar classes.dex (el .jar no se puede abrir con java -jar).
find . -type f | xargs grep is_admin 

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220110134913.png' | relative_url }}" text-align="center"/>
</div>

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220110150932.png' | relative_url }}" text-align="center"/>
</div>

apktool d InsecureBankv2 -o InsecureBankv2_mod.apk  

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220110154420.png' | relative_url }}" text-align="center"/>
</div>

<br />

# 2. Pentesting y Análisis Dinámico.

## 2.1. MobSF.

### 2.1.1. Instalación.

**MobSF** es un acrónimo de Mobile Security Framework, es decir; Marco de trabajo de Seguridad para Móviles. Es una herramienta de pentesting, análisis de malware y marco de evaluación de seguridad capaz de realizar análisis estáticos y dinámicos para aplicaciones moviles (Android, IOS, Windows).

Una vez clonamos el repositorio:

```bash
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF
```

que se instalará en una carpeta. Nos introducimos en dicha carpeta. Y ejecutamos los siguientes comandos:

```bash
pip install -r requirements.txt
```

```bash
sudo apt-get install python-venv
```

```bash
./setup.sh
```

A partir de aquí, el paquete habrá quedado instalado y para ejecutar MobSF hace falta correr el script ./run.sh 

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112160307.png' | relative_url }}" text-align="center"/>
</div>

En dicha imágen podemos observar que crea un servidor que se puede acceder desde el loopback en el puerto 8000.


<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112160722.png' | relative_url }}" text-align="center"/>
</div>

<br />

### 2.1.2. Análisis Estático e Informe.
En la pestaña "Upload & Analyze" seleccionamos una aplicación movil para analizarla. De esta forma se nos abrirá una interfaz:



<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112162003.png' | relative_url }}" text-align="center"/>
</div>

Aquí podremos ya generar un informe pdf del análisis estático.

<br />

### 2.1.3. Análisis Dinámico e Informe.

#### 2.1.3.1. Concepto.

El Análisis Dinámico consiste en ejecutar una aplicación dentro de un dispositivo y analizar el comportamiento del dispositivo durante la ejecución para comprobar la actividad de la aplicación.

<br />

#### 2.1.3.2. Preparación.

Para llevar a cabo este análsis dinámico debemos de tener instalado Genymotion y configurada una máquina virtual android. 

Una vez tenemos la máquina virtual android encendida y de que tanto la kali como el
adaptador 2 de la instancia máquina android de VirtualBox tienen el mismo tipo de red, hay que configuar a MobSF para reconozca su IP. Hay que modificar dos parámetros:

- La variable de entorno ANALYZER_IDENTIFIER='\<IP>:5000'
- Modificar el valor de la variable ANALYZER_IDENTIFIER con '\<IP>:5000' dentro del fichero /home/kali/.MobSF/config.py

La IP del máquina android se puede ver en el marco superior de la ventana de la máquina virtual:


<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220119115153.png' | relative_url }}" text-align="center"/>
</div>

Una vez llevadas a cabo esta operación MobSF debería de poder reconcer nuestro dispositivo y sólo debemos presionar el apartado Dynamic Analyzer y nos aparecerá la siguiente sección.

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112233336.png' | relative_url }}" text-align="center"/>
</div>

<br />

#### 2.1.3.3. Inicio del análisis dinámico.

Una vez hemos preparado el terreno, le pulsamos al botón verde que aparece a la derecha de nuestra aplicación apk y comenzará la instalación y ejecución del paquete en  nuestro dispositivo. (En este momento, dependiendo de la versión de nuestra máquina Android, se nos pedirá en el propio dispositivo que instalemos unos modulos. De ser el caso los instalamos y reiniciamos).

Seguidamente se ejecutará la aplicación en el dispositivo y en MobSF se mostrará:


<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220112234602.png' | relative_url }}" text-align="center"/>
</div>


Y le damos al botón verde correr aplicacion. Jugamos con la aplicación un poco y luego le damos al botón de generar informe:

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220113000946.png' | relative_url }}" text-align="center"/>
</div>

Se nos redirigirá a esta página y abajo a la izquierda podremos ver cómo descargar el report realizado:

<div style="text-align:center">
	<img src="{{ 'assets/img/M4/Pasted image 20220113001415.png' | relative_url }}" text-align="center"/>
</div>
