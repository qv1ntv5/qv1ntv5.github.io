---
layout: post
title: Mi experiencia con el OSCP
subtitle: Client Side attacks.
tags: [pen]
---
En este blog resumo las experiencias e ideas que he ido recogiendo a lo largo del curso PEN200.


1. Puntos claves sobre el curso.

	- Pasa rápidamente a los laboratorios. 
	
	Al ser esta una certificación eminentemente práctica, el autentico aprendizaje comienza cuando empiezas los laboratorios. Todo el tiempo de más que dedicas a la teoría es tiempo perdido.

	- No te sobreecompliques comprobando exploits o vectores de ataque. 
	
	
	Esta certificación no es exigente técnicamente, nadie espera que sepas hacer algo muy complicado o que consuma mucho tiempo. La mayoría de los imprevistos se superan prestando un poco de atención o, como mucho, realizando una búsqueda en google. Si algo es demasiado complicado desechalo.

	- Ten paciencia. 
	
	La parte más importante de la certificación consiste en aprender a saber qué hacer cuando te quedas estancado, y eso sólo se aprende enfrentandote a ella. No caigas en la tentación de mirar la solución rápidamente y date un momento para organizar tus ideas.

	<br />
	
2. Puntos claves a tener en cuenta acerca de la metodología.

	Esta certificación, pese a ser práctica, no requiere el desarrollo de unos conocimientos técnicos exigentes si no de una metodología de trabajo adecuada. En pocas palabras; las máquinas del curso, así como las del examen, tienen una lógica a la que hay que adaptarse que, creo, queda explayada en los siguientes puntos:


	- Hacer una enumeración metódica, organizada y exhaustiva.

	Es muy tentador explorar una máquina sin seguir un orden o una metodología concreta; acudir a aquellas partes que nos llamen la atención y testear si estos son vulnerables. Esto con frecuencia sólo conduce a la frustación ya que (excepto para las mentes muy experimentadas), la ruta más sugerente no suele ser el camino a seguir. 

	Yo recomiendo no dejarse llevar por el entusiasmo y, en primer lugar, recopilar información sobre la máquina para seguidamente repasar dicha información en busca de vectores de ataque y una vez enumerados todos los posibles vectores de ataque probarlos uno a uno. Si nada de esto funciona, continuar enumerando.



	- Aprender a pensar sobre la información que se recopila.

	Rara vez encontramos una máquina que se resuelva de manera directa, esto es; que encontremos todos los elementos necesarios para poder resolver la máquina en una primera barrida de enumeración. 

	Generalmente, las máquinas del curso PEN200 suelen ofrecer, tras un proceso de recopilación exhaustiva, una cantidad de información insuficiente para su resolución: puede que tengamos credenciales pero no tengamos un portal de logeo, o puede que tengamos un exploit RCE pero no tengamos credenciales para usarlo, o puede que no tengamos nada en absoluto. 

	En cualquier caso, tras una primera barrida es muy probable que no encontremos nada que podamos usar inmediatamente y lo convienente en estos casos es realizar una segunda barrida teniendo en cuenta la información ya recopilada, esto es; pensar qué se nos puede estar pasando, qué nos falta en base a lo que ya tenemos o incluso qué dice la información que hemos recopilado hasta ahora acerca de la propia máquina (puede que haga referencia a una tecnología que todavía no hemos encontrado o incluso que mencione la invalidez de unas credenciales o nombre de usuario). De forma que esta segunda barrida tiene que ir dirigida a encontrar esa pieza de información clave que nos falta para poder entrar en la máquina.


	- Nunca hay nada demasiado complicado.

	Que una operación sea demasiado complicada suele ser un indicador de que no estás yendo por el camino adecuado, estás en una 'rabbit hole'. De nuevo, en esta certificación nadie espera que poseas conocimientos técnicos muy complicados, la mayoría de los vectores de ataque son muy simples y pueden abordarse en cuestión de minutos como mucho con la ayuda de una búsqueda en google. 

	Por otra parte, el tiempo para resolver el examen es limitado (23:45 horas seguidas, teniendo en cuenta, comidas, sueño, tiempos de descanso) con lo que tampoco se espera que pases la mitad del tiempo recopilando información, de forma que con la enumeración sucede exactamente lo mismo; nada esta especialmente escondido. Los problemas con la enumeración suelen residir en que no hemos utilizado la herramienta adecuada (a veces es necesario probar con varias herramientas que hagan la misma tarea) o con que hemos pasado por alto información relevante tomandola por banal.


	- Aprender a saltar de máquina a máquina. 
	
	Es importante evitar la visión de túnel, y es una buena práctica cambiar cada cierto tiempo de máquina o de tarea (si estamos estancados) para evadirla y en la medida de lo posible, evitar perder tiempo. 

	<br />

3. Comentarios acerca de la preparación del examen y la metodología de trabajo.


	El exámen está basado en dos pilares; la enumeración post-explotación y pivoting, reflejado sobre el set de Active Directory y los 'foothold' reflejados en las 'standalone machines'. 

	La primera habilidad reside en tener un conocimiento razonable del sistema operativo Windows, esto es; de las cosas que son normales o de las que no tendrían que estar ahí y de dónde podemos encontrar cosas interesantes (como logs, archivos de configuración, etc) y un conocimiento básico de técnicas de conexión, creación y estabilización de shells en Windows, importante la herramienta PowerCat así como el kit de 'impacket'. 

	La segunda esta relativamente descrita en los puntos anteriores, pero a modo de resúmen, consiste en saber orientar la enumeración hacia un dato clave en un contexto de información limitada para poder obtener RCE. 

	Si conseguimos construir una metodología de trabajo basado en ambos pilares obtendremos el OSCP.

