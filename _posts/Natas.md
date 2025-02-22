---
layout: post
title: Natas
subtitle: Wargame de Overthewire Natas.
tags: [otw] 
---

# Natas OverTheWire

## Intro
Natas enseña nociones básicas de seguridad web por parte del servidor. Cada nivel de Natas consta de su propio sitio web ubicado en _http://natasX.natas.labs.oticalwire.org_, donde X es el número de nivel. No hay inicio de sesión SSH. Para acceder a un nivel, ingrese el nombre de usuario para ese nivel (por ejemplo, natas0 para el nivel 0) y su contraseña.

La contraseña del siguiente nivel siempre se encuentra en el nivel anterior.

<br />

## Level0
Se trata del nivel inicial, en la propia página aparece las credenciales de logeo y la URL.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123357.png' | relative_url }}" text-align="center"/>
</div>

Una vez nos logeamos dentro de natas0 aparece un mensaje que nos informa de que podemos obtener la contraseña del siguiente nivel dentro de esta página:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123407.png' | relative_url }}" text-align="center"/>
</div>

Evidentemente, en lo que vemos en nuestro navegador no observamos nada parecido a una contraseña. Podemos probar entonces a intentar inspeccionar el código fuente de la página presionando las teclas Ctrl + Shift.

De esta forma se desplagará DevTools (si estamos con Chrome) en el panel de Consola. Yendo directamente al panel de Elementos para inspeccionar el código HTML de la web. Dentro de este, en el “body” del documento web podemos encontrar un comentario donde nos indica la contraseña para el siguiente nivel.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123435.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level0 → Level1
Una vez accedemos al siguiente nivel, logeandonos con la contraseña y el usuario adecuado, se nos redirige a una página dónde nos dice que el botón derecho para acceder a la opción Inspeccionar Elemento está bloqueada.


<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123502.png' | relative_url }}" text-align="center"/>
</div>

Sencillamente pulsando Ctrl + U, o Ctrl + Shift + J volvemos a inspeccionar el código fuente y a encontrarnos con un comentario en el body HTML del documento web donde nos indica cuál es la contraseña.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123513.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level1 → Level2
Al acceder al siguiente nivel nos encontramos con un mensaje que nos dice que no hay nada en esta página. Inspeccionando el código HTML del documento podemos observar que junto al mensaje hay asociada una imágen que se referencia desde la misma aplicación web pero en otro directorio.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123550.png' | relative_url }}" text-align="center"/>
</div>

Así pues; añadiendo a la url de nuestro navegador .../files/pixel.png vamos a parar a la imágen e inspeccionando el código fuente no parece que esta tenga ningún dato de relevancia, con lo que este fichero se descarta y pasamos al directorio superior: .../files/

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123625.png' | relative_url }}" text-align="center"/>
</div>


Aquí parece que hemos encontrado dos ficheros, el fichero .png ya inspeccionado y un fichero .txt. Al ir a ese fichero .txt nos aparecen una lista de contraseñas. Entre ellas la contraseña para acceder al nivel natas3.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123637.png' | relative_url }}" text-align="center"/>
</div>

Este apartado ya nos está indicando que el servidor web que almacena los sitios de cada nivel de natas no contiene sólo los ficheros natas de manera aislada si no que contiene además subdirectorios y ficheros que pueden contener información relevante.

<br />

## Level2 → Level3

En este apartado de nuevo nos vuelve a aparecer el mismo mensaje sobre que no hay nada en esta página. Al inspeccionar el código fuente nos encontramos con un comentario que nos informa de que no va a haber más filtraciones de información, y que ni siquiera Google será capaz de darnos esa información esta vez.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123719.png' | relative_url }}" text-align="center"/>
</div>

Este comentario quizá este tratando de decirnos que la solución vuelve a estar en algún fichero o subdirectorio de esta página web pero que no nos va a dar pistas para decirnos cuál es. Con la herramienta **dirb** llevamos a cabo un mapeado básico de la web:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123731.png' | relative_url }}" text-align="center"/>
</div>

En el mapeado básico que hemos hecho (acreditando nuestras credenciales) de la página web encontramos que el fichero **robots.txt** es accesible. Abriendo la dirección …/robots.txt encontramos que:


<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123748.png' | relative_url }}" text-align="center"/>
</div>

El robots.txt impide a los buscadores o rastreadores de web que acceder al subdirectorio de la web / _s3cr3t /,_ e aquí la referencia acerca de google no sería capaz de encontrar la solución.

Introduciéndonos en tal directorio encontramos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303123802.png' | relative_url }}" text-align="center"/>
</div>


Y dentro del archivo users.txt obtenemos la contraseña para el siguiente nivel:

natas4:Z9tkRkWmpt9Qr7XrR5jWRkgOU901swEZ


<br />

## Level3 → Level4

Al acceder al nivel de natas4 se nuestra muestra el siguente mensaje.


<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124032.png' | relative_url }}" text-align="center"/>
</div>

Y al refrescar la página


<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124045.png' | relative_url }}" text-align="center"/>
</div>

Es decir, que el servidor web evalúa nuestro sitio de llegada y en función de ello decide si nos deja pasar o no.

Sería por tanto buena idea interceptar nuestra petición con un proxy, modificar nuestra referencia de envío y liberarla para engañar a la web acerca de nuestra procedencia. Esto puede realizarse mediante la herramienta **burpsuite**.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124105.png' | relative_url }}" text-align="center"/>
</div>

De esta manera, modificamos el apartado Referer que contiene nuestra dirección de origen auténtica y cambiamos el 4 por un 5 y liberamos la petición obteniendo:


<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124116.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level4 → Level5

En este problema, al logearnos se nos informa de que no nos hemos autenticado adecuadamente. Hay por tanto algo en nuestra petición que hemos de revisar.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124144.png' | relative_url }}" text-align="center"/>
</div>

Interceptando dicha petición con el proxy de burpsuite y revisando las cookies que se envían con nuestra petición observamos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124155.png' | relative_url }}" text-align="center"/>
</div>

De entre todas las cookies hay un parámetro que llama la atención; loggedin=0. Cambiando el 0 por un 1 y liberando la petición obtenemos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124205.png' | relative_url }}" text-align="center"/>
</div>

Las cookies son un conjunto de parámetros que el servidor concede a un usuario de la página a modo de recordatorio del tipo de usuario que es y de los privilegios que tiene de acuerdo a un logeo previo del mismo.

Estas consisten en un conjunto de parámetros que el servidor evalúa y en función de los cuales concede al usuario una serie de privilegios, un simple cambio en los parámetros puede cambiar los privilegios de dicho usuario que es justo lo que ocurre en este problema.
 
Pese a habernos autenticado, el servidor no nos da los privilegios adecuados y esto se guarda en las cookies de forma que cuando reenviamos la petición de logeo, interceptamos dicha petición y ajustamos las cookies (cambiando loggedin de 0 a 1) se nos concede acceso.

<br />

## Level5 → Level6
En este caso, se nos pide que además del logeo, introduzcamos una contraseña secreta. A modo de ayuda nos ponen un código fuente que contiene el algoritmo que procesa la contraseña que introducimos.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124255.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124259.png' | relative_url }}" text-align="center"/>
</div>

    

En el podemos observar que compara el input que recoge de la página con una variable denominada secret y que además incluye un fichero llamado secret.inc en .../includes/secret.inc. Yendo a dicha dirección obtenemos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124323.png' | relative_url }}" text-align="center"/>
</div>

Se trata evidentemente de nuestra contraseña como se puede inferir del comportamiento del algoritmo, con lo que introduciendo dicho string en el ‘input secret’ obtenemos:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124422.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level6 → Level7:**
En este ejercicio, nada más logearnos nos aparece lo siguiente:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124445.png' | relative_url }}" text-align="center"/>
</div>


Explorando un poco más el código fuente obtenemos que desplegando el body tenemos el siguiente mensaje escondido en un comentario.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124629.png' | relative_url }}" text-align="center"/>
</div>
    

De forma que el problema plantea el siguiente obstáculo. Acceder a una zona del servidor distinta de la zona en la que se localiza la página web y sus directorios.

  

Para resolver este problema vamos a utilizar la **vulnerabilidad de inclusión de archivos**. Esta se aprovecha de una mala programación de una página dinámica PHP que permite el enlace de archivos remotos para exponer ficheros que pueden contener información sensible.

En este caso la referencia de inclusión de archivos es: \<a href=“index.php?page=home”>. Cambiando entonces “home”, que se trata de un subdirectorio incluido en el directorio raíz de la página web por el directorio indicado en el comentario y accediendo al enlace obtenemos que:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124642.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124655.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level7 → Level8

En primer lugar en este problema nos vuelven a pedir una contraseña de acceso y nos vuelven a dar un algoritmo de procesamiento del input intrucido.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124719.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124724.png' | relative_url }}" text-align="center"/>
</div>

En el se puede apreciar que se trata de un programa en el que: se coge nuestra cadena de caracteres, la cifra mediante una función resultante de combinar diversas funciones predefinidas de php, lo

compara con un string almacenado en una variable denominada “encodedSecret” (que evidentemente se trata de nuestra contraseña escondida tras un proceso de cifrado) y si coincide nos da la clave para el siguiente nivel.

Así pues, en nuestra kali construimos un script en php que haga justo lo contrario de lo que hace la función de cifrado en el script anterior utilizando las funciones predefinidas adecuadas y le pasamos el contenido de $encodedSecret para obtener el string a introducir por pantalla para obtener la contraseña del siguiente nivel. Seguidamente lo compilamos y ejecutamos con el comando ‘php’ en kali.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124940.png' | relative_url }}" text-align="center"/>
</div>

Obteniendo así (el script se llamó natas7 por equivocación, debería de ser natas8):

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124952.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303124956.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level8 → Level9

En este caso, nos ponen ante ventana que nos permite introducir input y nos devuelve un output basado en el texto que hemos introducido:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125039.png' | relative_url }}" text-align="center"/>
</div>

Y al mismo tiempo nos dan el código fuente del script que utilizan para manejar nuestro input y en función de eso generar output.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125312.png' | relative_url }}" text-align="center"/>
</div>

Podemos observar que lo que hace es primero identificar si la entrada está vacía, si no está vacía mete nuestro input en una función de php (passthru) que ejecuta el comando contenido entre paréntesis sin introducir ninguna condición de saneamiento de código.

Se trata por tanto de una página vulnerable ante un ataque de inyección de comandos.

Además, siguiendo por los derroteros de los ejercicios anteriores, probablemente la contraseña para el siguiente nivel este almacenada en: /etc /natas_webpass /natas10:

Así:

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125328.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level9 → Level10
En este caso tenemos el mismo problema que en el caso anterior salvo por el hecho de que se han filtrado caracteres de concatenación de comandos. Sin embargo, nos podemos aprovechar de la forma en la que está construido el comando. El script utiliza la función passthru y ejecuta el comando:


```bash
grep -i <input> dictionary.txt
```
  

Y lo que hace en principio es buscar nuestro input en dictionary.txt. Sin embargo, grep admite la concatenación de ficheros en los que buscar y por tanto podemos pedirle que busque nuestro ‘input’ en /etc/natas\_webpass/natas11. Así, bastará con ir probando letras y obtendremos la contraseña en el momento en el que introduzcamos un caracter que se encuentre incluido dentro de la cadena de caracteres que constituye de hecho la contraseña. En este caso obtenemos lo que queremos al probar con la ‘u’, es decir, que una frase válida como solución sería: “u /etc /natas_webpass /natas11”

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125607.png' | relative_url }}" text-align="center"/>
</div>

<br />

## Level10 → Level11

## Level11→ Level12

En este nivel nos encontramos con que podemos subir un fichero al servidor web.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125731.png' | relative_url }}" text-align="center"/>
</div>


Inspeccionando además el código fuente de la página web podemos además comprobar que existen dos valores escondidos que se subirán junto con el archivo que nosotros elijamos;

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125840.png' | relative_url }}" text-align="center"/>
</div>


Además, tenemos la opción de echar un vistazo a un fichero que contiene un algoritmo que muestra lo que se supone que son las operaciones que la página web realiza a la hora de subir el fichero.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125853.png' | relative_url }}" text-align="center"/>
</div>


Analizando este código podemos comprobar que, en definitiva y muy resumidamente, este script de php utiliza una función que genera un nombre aleatorio, otra función que utiliza la anterior para generar una ruta sobre el nombre aleatorio y una tercera que comprueba y valida la extensión de la ruta que se le pasa como argumento. Estas tres funciones se utilizan sobre el elemento escondido "filename" para generar un path fijo en un subdirectorio de la web (/upload/) que siempre tiene una terminación .jpg.

Y posteriormente mueve nuestro fichero "uploadedfile" sobre el path anterior al subirlo al http post request, de forma que nuestro fichero queda con un nombre aleatorio y una extensión .jpg.

De esta forma, nuestro archivo queda subido con un nombre aleatorio y una extensión .jpg impidiendo que se puedan subir ficheros con otras extensiones.

Ahora bien, observemos dos cosas:

- En primer lugar, no filtra qué tipo de archivo subimos, sólo se asegura de que dicho archivo se sube con una extensión .jpg en una localización concreta

- En segundo lugar, cuando subimos un archivo, este siempre tendrá la extensión del valor del elemento cuyo nombre es “filename” esto es; .jpg (por eso siempre tiene la misma extensión independientemente de qué archivo sea), si cambiamos la extensión del valor de “filename” antes de subir nuestro archivo, al subirlo el sistema comprobará que “filename” tiene una extensión distinta y añadirá dicha extensión a nuestro archivo subido.

Es decir, que no filtra qué contenido subimos y, aunque se esfuerce por poner una extensión al archivo que se sube, esta extensión se puede manipular a conveniencia, pudiendo así subir un script de php en un archivo que puede terminar con extensión .php dentro de la aplicación web, siendo susceptible de ser ejecutado por el servidor cuando se manda la http post request y mostrando información sensible, como la contraseña al siguiente nivel contenida en /etc/natas_webpass/natas13.

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125951.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125955.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303125959.png' | relative_url }}" text-align="center"/>
</div>

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/Wargames/Pasted image 20220303130004.png' | relative_url }}" text-align="center"/>
</div>
