---
layout: post
title: Evasión de Defensas
subtitle: Introduccióna evasión de Antivirus y UAC.
tags: [redteam] 
---

# 1. Antivirus.

### 1.1.1. Antivirus como concepto.

Un antivirus es un programa que tiene la finalidad de detectar malware, código mal intencionado capaz de perjudicar la integridad de la máquina en la que esta se ejecuta.

**Un software antivirus comienza comparando tus archivos y programas informáticos con una base de datos de tipos de malware conocidos**, estas tareas se realizan en segundo plano. Debido a que los hackers crean y distribuyen nuevos virus constantemente, también hará un escaneo de los equipos en la búsqueda de un tipo nuevo o des conocido de amenaza de malware.

Existe una página web que permite testear un antivirus recién adquirido: **eicar**.

<br />

### 1.1.2. Tipos de malware.

- **Keylogger**: Un keylogger es un tipo de malwar que captura las teclas que el usuario pulsa en el teclado conectado físicamente a su dispositivo. Se trata de un subtipo de otro tipo de malware llamado **spyware** que tiene por objetivo espiar al dispositivo afectado.

- **Worm**: Este tipo de malware puede tener muchas funcionalidades, pero lo más característico es su capacidad de replicación y traslación a otros dispositivos.

- **Troyan**: Su nombre proviene del Caballo de Troya de la Eneida de Virgilio, haciendo referencia al hecho de que este tipo de malware en apariencia se presenta como una aplicación inofensiva que en realidad tiene comportamiento insidioso ya que desarrolla tareas perjudiciales para el usuario en segundo plano.

- **Adware**: Este software rastrea el navegador y el historial de descargas del usuario con la intención de mostrar anuncios emergentes o banners no deseados para atraer al usuario a realizar una compra o hacer clic.

- **Ransomware**: Este malware cifra los archivos del disco duro del dispositivo y restringe el acceso del usuario a ellos



<br />

### 1.1.3. Evasión de antivirus.

En esencia lo que tenemos frente a un antivirus no es más que una base de datos que contiene firmas de malware (esto es hashes de estractos de código malware) conocido. Evadir el antivirus consiste en codificar el malware para hacer que la firma de nuestro malware parezca la de una aplicación legítima o que almenos no esté en la base de datos.

<br />

## 1.2. Herramientas de evasión antivirus.

Como hemos indicado en el apartado anterior, vamos a utilizar herramientas que nos van a permitir cifrar nuestro malware para evitar que sea detectado como tal. **Concretamente, vamos a añadir código que no altera la funcionalidad pero que si puede llegar a alterar la firma de nuestro software para que no sea detectada por el antivirus**.

<br />

### 1.2.1. MSFVenom.
Con MSFVenom pode acudir a dos técnicas:

- **Cifrado**: Con MSFVenom podemos cifrar el payload que estamos generando en forma de binario. 

	<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220131212010.png' | relative_url }}" text-align="center"/>
	</div>
	
	Sin embargo, aunque los antivirus no sean buenos escaneando malware cifrado, al descifrarlo y utilizarlo el escaneo en tiempo real sigue suponiendo una línea de defensa fuerte y por tanto 
	
- **Encoders**: También podemos utilizar herramientas de codificación que añden código inservible con la finalidad de alterar la firma. Este código se puede añadir en una serie de iteraciones que nosotros podemos seleccionar. Esto hará que, a mayor número de iteraciones, menos probable será que sea detectado pero más pesara ya que lo que se hace en esencia es añadir código.
	
	<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220131223951.png' | relative_url }}" text-align="center"/>
	</div>

	Entre los encoders más destacados encontramos:
	
	- x86/shikata_ga_nai
	- x86/countdown
	
	<br />
	
### 1.2.2. Metasploit.

Con Metasploit podemos utilizar módulos de evasión de defensas que configurarán un payload para que pase desapercibido ante un antivirus:

Utilizamos metasploit para buscar el modulo correspondiente y ajustar las correspondientes variables.


<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220201131506.png' | relative_url }}" text-align="center"/>
	</div>
	
Seguidamente, subimos el fichero a VirusTotal y observamos el resultado.

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220201131943.png' | relative_url }}" text-align="center"/>
	</div>

<br />

### 1.2.3. Winpayloads.
	
Lo instalamos tal y como se indica en : https://github.com/nccgroup/Winpayloads. La instalación convencional está obsoleta, hace falta dirigirnos a la página web.
Una vez lo hemos instalado seguimos las instrucciones para llevar a cabo los payloads correspondientes:

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220201155053.png' | relative_url }}" text-align="center"/>
	</div>

Una vez terminado el proceso se almacenará nuestro payload en una ubicación y  abrirá un multihandler automáticamente a la espera de que el payload generado se ejecute en la máquina víctima:

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220201155202.png' | relative_url }}" text-align="center"/>
	</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220201155214.png' | relative_url }}" text-align="center"/>
	</div>

<br />

### 1.2.4. The FatRat.

TheFatRat es una herramienta de explotación que compila un malware con un payload conocido y luego el malware compilado puede ser ejecutado. 

TheFatRat proporciona una manera fácil de crear puertas traseras y cargas útiles que pueden pasar por alto la mayoría de los antivirus.

Lo instalamos con github; 

```bash
git clone https://github.com/Screetsec/TheFatRat.git
cd TheFatRat
chmod +x setup.sh && sudo ./setup.sh
```
<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220201161032.png' | relative_url }}" text-align="center"/>
	</div>

Seguidamente se instalan las dependencias.

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220201161201.png' | relative_url }}" text-align="center"/>
	</div>

Una vez se termina de instalar lo ejecutamos desde la terminal. Seguimos las instrucciones para generar el payload deseado.

<br />

# 2. Evasión UAC.

## 2.1. UAC.

### 2.1.1. UAC como concepto.

UAC es un acrónimo de User Accounts Control, (Control de las Cuentas de Usuario). **Se trata de un sistema de avisos y seguridad destinado a proteger un equipo contra modificaciones no autorizadas** presente en Windows desde Windows Vista.

Cada vez que una aplicación quiera llevar a cabo una modificación en tu equipo necesita del visto bueno de la UAC que a su vez estará esperando tu consentimiento para permitir la acción del proceso en cuestión.

<br />

### 2.1.2. Como funciona el UAC.

La severidad de este sistema de protección está dividido en cuatro niveles cada cual más restrictivo que el anterior. 

1.   **No notificarme nunca**: Es la configuración menos recomendada, porque desactiva el UAC. 
   
	   Esto quiere decir que no te notificará cuando las aplicaciones intenten instalar otro software o realizar cambios en el equipo. 
	   
	   Ni cuando seas tú quien realice los cambios. 
	   
	   Tampoco atenuará la ventana en ningún momento impidiendo que puedas seguir trabajando.

<br />

2.   **Notificarme solo cuando una aplicación intente realizar cambios en el equipo (no atenuar el escritorio)**: 
  
		Avisándote cuando los programas que tienes en Windows intenten instalar otras aplicaciones o realizar cambios en el equipo. 
		
		No notificándote cuando seas tú quien realice cambios en la configuración. 
		
		No se inmovilizará el resto de tareas, podrás seguir trabajando con el aviso de fondo.
		
<br />
		
3. **Notificarme solamente cuando una aplicación intente realizar cambios en el equipo**: Es el nivel de seguridad que viene configurado por defecto. 
 
	Sólo te notificará cuando los programas que tienes en Windows intenten instalar otras aplicaciones o realizar cambios en el equipo. 
	
	NO te avisará cuando tú realices cambios en la configuración. 
	
	Sí inmovilizará otras tareas hasta que respondas en la ventana emergente.

<br />

4. **Notificarme siempre**. 

	 Te notificará cuando los programas que tienes en Windows intenten instalar otras aplicaciones o realizar cambios en el equipo. 
	 
	 También se te notificará cuando realices cambios en la configuración. 
	 
	 Se inmovilizarán otras tareas hasta que respondas a la ventana de autorización.
	 
En Windows 10 podemos acceder a este gestor mediante **Panel de Control > Cuentas de Usuario > Cuentas de Usuario > Cambair configuracion UAC**.

Cuando esta se encuentra en el máximo nivel, puede suponer un problema a la hora de utilizar ciertas funcionalidades en meterpreter o dentro de la máquina simplemente: 

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220202191309.png' | relative_url }}" text-align="center"/>
	</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220202195227.png' | relative_url }}" text-align="center"/>
	</div>

Sin embargo, podemos utilizar módulos de metasploit para saltarnos esta clase de barreras:

    

- use exploit/windows/local/bypassuac

- use exploit/windows/local/bypassuac_comhijack

- use exploit/windows/local/bypassuac_dotnet_profiler

- use exploit/windows/local/bypassuac_eventvwr

- use exploit/windows/local/bypassuac_fodhelper

- use exploit/windows/local/bypassuac_injection

- use exploit/windows/local/bypassuac_injection_winsxs

- use exploit/windows/local/bypassuac_sdclt

- use exploit/windows/local/bypassuac_silentcleanup

- use exploit/windows/local/bypassuac_sluihijack

- use exploit/windows/local/bypassuac_vbs

- use exploit/windows/local/bypassuac_windows_store_filesys

- use exploit/windows/local/bypassuac_windows_store_reg

De forma que si el UAC no está en el nivel más alto, si la shell que manipulamos es propiedad de alguien que pertence al grupo de administradores y configurando adecuadamente las variables del módulo obtenemos:

<div style="text-align:center">
	<img src="{{ 'assets/img/M8/Pasted image 20220202195125.png' | relative_url }}" text-align="center"/>
	</div>