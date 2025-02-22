---
layout: post
title: Reto máquina Mr.Robot
subtitle: Intrusión y elevación de privilegios en una máquina Debian.
tags: [wargame] 
---
# Informe CTF Elevación Privilegios.

# 1. Desempaquetando la máquina.

En primer lugar, nos descargamos la máquina comprimida con ZIp e intentamos descomprimirla. Como está comprimida con contraseña, nos disponemos a utilizar la herramienta fcrackzip con la lista rockyou.txt sobre el fichero zip descargado que contiene la máquina. Puede ser buena idea, dado que el fichero se llama 250620, que la contraseña sea la línea número 250620 de la lista rockyou.txt que estamos utilizando:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131133717.png' | relative_url }}" text-align="center"/>
</div>
Da resultado y obtenemos la máquina.

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131134035.png' | relative_url }}" text-align="center"/>
</div>

<br />

# 2. Accediendo a la máquina.

## 2.1. Analizando puertos.

Una vez hemos importado satisfactoriamente la máquina toca conseguir alguna credencial de logeo que nos permita acceder a la misma. Con tal finalidad, exploramos los puertos abiertos de la máquina con nmap:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131145856.png' | relative_url }}" text-align="center"/>
</div>

De manera que, por descarte, podemos averiguar que **la IP de la máquina es 10.0.2.18** y que tiene dos puertos abiertos: el http y el ssh.

<br />

## 2.2. Introduciéndonos en la web.
Como tiene el puerto http abierto podemos ver si una página web sería accesible para investigar e intentando acceder desde firefox a la dirección: http://10.0.2.18:80 nos encontramos con que así es:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131150152.png' | relative_url }}" text-align="center"/>
</div>

De esta manera, procedemos a utilizar herramientas de escaneo web como **dirb** para llevar a cabo un repaso de los directorios y ficheros web de la máquina:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131150416.png' | relative_url }}" text-align="center"/>
</div>

Entre los ficheros web disponibles podemos ver que hay el **robots.txt**, que puede contener información sobre ficheros que contengan contenido sensible.

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131150523.png' | relative_url }}" text-align="center"/>
</div>

Dirigiéndonos al subdirectorio **users** observamos que hay información relativa a lo que podrían ser distintos usuarios y concretamente, dentro del también subdirectorio chung encontramos un .jpg con lo que parecen ser sus credenciales de logeo a la máquina:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131150746.png' | relative_url }}" text-align="center"/>
</div>

Probamos a acceder mediante el puerto ssh con estas credenciales para ver si de esa manera podemos acceder a la máquina:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131150940.png' | relative_url }}" text-align="center"/>
</div>

Y con esto habríamos accedido a la máquina.

<br />

# 3. Elevación de privilegios.

## 3.1. Chung.
Dentro del usuario **chung** nos encontramos dentro del directorio **/var/www/html** y eso nos da la opción de seguir indagando dentro de la información contenida en la página web. 

Escarvando dentro del directorio wordpress llegamos a un directorio **secret.php** en el que aparecen las credenciales de otra usuario; angela:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131151724.png' | relative_url }}" text-align="center"/>
</div>

cifradas en base 64. Así, pasándo dicha cadena por un decodificador de base64 obtenemos que sus credenciales son: ..r0ut1n3...

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131152121.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 3.2. Angela.

Una vez dentro del usuario Angela, comenzamos dentro de su directorio de trabajo y después de haber examinado todas sus carpetas, evaluamos sus acciones dentro del historial de comandos contenido en: **.bash_history**

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131152330.png' | relative_url }}" text-align="center"/>
</div>

De entre todos los comandos podemos observar que ejecuta el binario **find** contenido en el mismo directorio de trabajo dentro de la carpeta **bin**. Acudiendo a la misma carpeta y ejecutando los comandos que ella ejecuta obtenemos que:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131152617.png' | relative_url }}" text-align="center"/>
</div>

y por tanto, hemos saltado al usuario **darlene**.

<br />

## 3.3. Darlene.
Una vez hemos accedido al usuario **Darlene**, intentamos acudir al directorio de trabajo de dicha usuario. Una vez dentro, hay un directorio oculto cuyo nombre es **.private** y dentro de la misma hay unas credenciales de otro usuario:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131153026.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131153127.png' | relative_url }}" text-align="center"/>
</div>

<br />

## 3.4. Whiterose.

Una vez nos hemos logeado como el usuario **whiterose** tenemos que empezamos de neuvo en su directorio de trabajo. Examinando de nuevo el historial de comando vemos una secuencia rara:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131153701.png' | relative_url }}" text-align="center"/>
</div>

Y al intentar reproducirla obtenemos un error:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131154406.png' | relative_url }}" text-align="center"/>
</div>

Y al permutar los parámetros, cambiamos de usuario:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131154427.png' | relative_url }}" text-align="center"/>
</div>

Analizando el comando introducido no queda más remedio que asumir que hemos obtenido las credenciales de un usuario nuevo; **darkarmy:d@t@w@arxx**.

<br />

## 3.5. Darkarmy.

De nuevo, estamos con otro usuario y en el directorio de trabajo de otro usuario distinto. Al dirigirnos a nuestro directorio de trabajo y examinar de nuevo su historial de comandos encontramos que ha ejecutado **cmd** y al repetir nostros la instrucción obtenemos:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131154940.png' | relative_url }}" text-align="center"/>
</div>

Y hemos vuelto a cambiar de usuario.

<br />

## 3.6. Eliott.

De nuevo, comenzamos en el directorio de trabajo de otro usuario y al vovler al directorio de trabajo de eliott y examinar su historial de comandos observamos que ha ejecutado dpkg -l con sudo y al intentar reproducir nosotros dicho comando nos encontramos con que no nos pide contraseña para utilizar dpkg -l con privilegios de super usuario. 

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131155311.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, si puediéramos utilizar dpkg para que nos abriera una shell  como root habríamos escalado de privilegios al superusuario.

De esta manera, se tiene que mirándo en gtfobins nos encontramos con el siguiente comando para nuestro propósito:

<div style="text-align:center">
<img src="{{ 'assets/img/Wargames/Pasted image 20220131155431.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, introduciendo dicho comando secuencialmente obtendríamos una shell de root y habríamos escalado de privilegios al super usuario y tomado control total sobre la máquina.





