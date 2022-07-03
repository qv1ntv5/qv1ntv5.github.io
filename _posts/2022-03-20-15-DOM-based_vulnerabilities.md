---
layout: post
title: 15.DOM-based XSS vulnerabilities.
subtitle: Detección y explotación de vulnerabilidades XSS basados en DOM través de la herramienta BurpSuite.
tags: [burp]
---
## 0. Índice.
- 1 Introducción al DOM.
- 2 Taint-flow vulnerabilities.
	- 2.1. Web message source.
	- 2.2. DOM-based open redirection.
	- 2.3. DOM-Based cookie manipulation.

	<br />

### 1. Introducción al DOM.
El modelo de objeto de documento (DOM) es la representación jerárquica de los elementos de la página renderizada por el navegador web. Los sitios web pueden usar JavaScript para manipular los nodos y objetos del DOM, así como sus propiedades. La manipulación de DOM en sí misma no es un problema. De hecho, es una parte integral de cómo funcionan los sitios web modernos. Sin embargo, JavaScript que maneja datos de manera insegura puede permitir varios ataques. Las vulnerabilidades basadas en DOM surgen cuando un sitio web contiene JavaScript que toma un valor controlable por el atacante, conocido como fuente, y lo pasa a una función peligrosa, conocida como sumidero.

<br />

### 2. Taint-flow vulnerabilities.

Para explotar o mitigar estas vulnerabilidades, es importante que primero se familiarice con los conceptos básicos del flujo contaminado entre fuentes y sumideros.

Fundamentalmente, las vulnerabilidades basadas en DOM surgen cuando un sitio web pasa datos de una fuente a un sumidero, que luego maneja los datos de manera insegura en el contexto de la sesión del cliente:

<br />

- Una **fuente** es una propiedad de JavaScript que acepta datos que están potencialmente controlados por un usuario. Un ejemplo de una fuente es *location.search* porque lee la entrada de la cadena de consulta, que es relativamente fácil de controlar para un atacante. En última instancia, cualquier propiedad que pueda ser controlada por el atacante es una fuente potencial. Esto incluye la URL de referencia (expuesta por la document.referrer cadena), las cookies del usuario (expuestas por la document.cookie cadena) y los mensajes web.

<br/>

- Un **sumidero** es una función de JavaScript potencialmente peligrosa o un objeto DOM que puede causar efectos no deseados si se le pasan datos controlados por un atacante. Por ejemplo, **la  eval() función es un sumidero porque procesa el argumento que se le pasa como JavaScript**. Un ejemplo de un sumidero HTML es  document.body.innerHTML porque potencialmente permite que un atacante inyecte HTML malicioso y ejecute JavaScript arbitrario.

<br />

### 2.1. Web message source.

Si una página maneja mensajes web de forma insegura se pueden provocar sumideros potenciales.

Veámos cómo provocar un ataque basado en "web messages".

**Laboratorio 1: DOM XSS using web messages**

En primer lugar, acudimos al laboratorio e inspeccionamos el script contenido en el código fuente. Podemos comprobar que posee un *addEventListener*:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220624110856.png' | relative_url }}" text-align="center"/>
</div>

Este escucha web messages. De esta forma abrimos el servidor del exploit e introducimos el siguiente contenido en el apartado body:

```HTML
<iframe src="https://your-lab-id.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
```

De esta forma, al cargar el *iframe* el postMessage() envía un mensaje a la página *home*. El EventListener está implementando sobre un código vulnerable que carga el contenido del mensaje sobre un *div*. En este caso, lo que carga es una imagen con una fuente no valida que desencadena un error con código JavaScript malicioso.

<br />

**Laboratorio 2: DOM XSS using web messages and a JavaScript URL**

En este caso, al inspeccionar el código fuente encontramos de nuevo un addEventListener sin embargo, además tiene el componente IndexOf() que comprueba si hay "http:" o "https:"

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220624113932.png' | relative_url }}" text-align="center"/>
</div>

Esto nos obliga a introducir un payload que contenga dicho string. Así, nos valdrá con el siguiente código "iframe":

```HTML
<iframe src="https://your-lab-id.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">
```

Este script envía un mensaje web que contiene una carga útil de JavaScript arbitraria, junto con la cadena "http:". El segundo argumento especifica que se permite cualquier targetOrigin para el mensaje web.

Cuando se carga el iframe, el método postMessage() envía la carga útil de JavaScript a la página principal. El detector de eventos detecta la cadena "http:" y procede a enviar la carga útil al receptor de location.href, donde se llama a la función print().

<br />

Una medida de seguridad adicional consiste en un mecanismo de verificación del origen:

**Laboratorio 3: # DOM XSS using web messages and JSON.parse**

Accedemos al laboratorio, inspeccionamos el código fuente y:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220624122353.png' | relative_url }}" text-align="center"/>
</div>

Podemos obsrervar que contiene un JSON.parse(), de esta forma introducimos el siguiente payload en el servidor del exploit:

```HTML
<iframe src="https://your-lab-id.web-security-academy.net/" onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

<br />

#### 2.2. DOM-based open redirection.

Las vulnerabilidades de redirección abierta basadas en DOM surgen cuando un script escribe datos controlables por un atacante en un receptor que puede desencadenar la navegación entre dominios.

<br />

**Laboratorio 4: DOM-Based XSS open redirect**

Acudimos al blog de los post e inspeccionamos el código fuente y observamos:

<div style="text-align:center">
	<img src="{{ 'assets/img/Burp/Pasted image 20220624140744.png' | relative_url }}" text-align="center"/>
</div>

Es decir, que existe un botón que lleva a cabo una redirección vulnerable permitiendo cambiar la dirección a la que se le redirige al usuario que pulsa el botón:

Así, podemos construir el siguiente enlace:

```default
https://your-lab-id.web-security-academy.net/post?postId=4&url=https://your-exploit-server-id.web-security-academy.net/
```

Con esto modificaremos el parámetro de la redirección y cualquier que pulse el botón en cuestión será redirigido a un sitio web de conveniencia.

<br />

### 2.3. DOM-Based cookie manipulatio.

Algunas vulnerabilidades basadas en DOM permiten a los atacantes manipular datos que normalmente no controlan. Esto transforma los tipos de datos normalmente seguros, como las cookies, en fuentes potenciales. Las vulnerabilidades de manipulación de cookies basadas en DOM surgen cuando un script escribe datos controlables por un atacante en el valor de una cookie.

<br />

**Laboratorio 5: DOM-based cookie manipulation**

Accedemos al laboratorios, acudimos a un post y luego a otro e interceptamos la request observando que existe una cookie indicando la dirección del producto que hemos visitado anteriormente. Esta dirección se adhiere a una botón de redirección de forma que aquel que pulse dicho botón será direccionado al sitio que contenga la dirección del valor de la cookie.

Así, en el servidor del exploit colocamos el siguiente código iframe:

```HTML
<iframe src="https://your-lab-id.web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://your-lab-id.web-security-academy.net';window.x=1;">
```

