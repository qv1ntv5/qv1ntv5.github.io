---
layout: post
title: Gramáticas en Dharma
subtitle: Introducción a la construcción de gramáticas para Dharma.
tags: [DHA]
---
### Dharma

Dharma es un mutational-grammar-based que genera programas con inputs (llamadas a webAPIs) sintáctica y semánticamente correctos pero incorporando cambios (mutaciones) en cada input con el objetivo de provocar un fallo, un bug, en el engine de procesamiento de los programas que crea. 

La variación de un input a otro y la diferencia entre un programa y otro depende de la introducción en Dharma de un fichero concreto llamado 'Gramatica'. Las gramaticas de Dharma son ficheros con sintaxis propia que definen la estructura del programa a construir en Dharma.

<br>

### Gramáticas en Dharma

Básicamente, Dharma es un programa que adapta un modelo de plantillas para generar programas en texto plano que luego pueden procesarse a través de un engine (como por ejemplo, un intérprete de JavaScript o un compilador de C). Podemos utilizar este modelo de plantillas para construir muchos scripts que muten en cada iteración que posteriormente son procesados por el engine/intérprete en cuestión en busca de recopilación de bugs o recopilación de información.

<br>

### Aprender a utilizar Dharma.

**Antecedentes**

Antes de continuar debemos recapitular que Dharma es un programa que construye plantillas sobre scripts de distintos lenguajes de programación y que, como herramienta, tiene su propia sintaxis diferenciada de la sintaxis del lenguaje de programación sobre el que plantea una plantilla, es decir, que hay que distinguir las variables, referencias y funciones del lenguaje de Dharma de las variables, referencias y funciones de un lenguaje de programación sobre el que construimos una plantilla en Dharma para formar scripts.

Es decir, que en todo lo que viene a continuación, se habla de variables o funciones de Dharma y variables o funciones de un lenguaje de programación (en este caso; JavaScript) y no hay que confundirlas.

<br>

#### Secciones.

Una parte fundamental de la utilización de Dharma son las secciones, que definen los datos y como estos son tratados. Se definen las siguientes secciones (value, variable, variance) y sus propósitos.

<br>

**Value Section**

En esta sección se definen las variables (contenedores de datos) de Dharma.

Un ejemplo de la sección 'value' puede ser la siguiente:

```
%section% := value

name :=
	Tom
	Jerry
	Bunny

message :=
	I want to play with +name+!
```

La variable 'name' puede ser referenciada en partes ulteriores del código entre signos '+', como se ve en la variable 'message'. Cuando se creen los programas y se procesen las plantillas, el contenido de 'message' será en parte rellenado con los valores de 'name'.

<br>

**Variable Section**

En esta sección se definen la forma de las variables del lenguaje de programación sobre el que se está planteando la plantilla. Por ejemplo;

```
%section% := variable

string :=
	var @string@ = +message+
```

Observemos que estamos declarando variables en JavaScript, a la hora de ejecutar Dharma con esta gramática, primero se declararán las variables construidas en esta sección y luego se construirá el resto del código JavaScript con el objetivo de hacer un código sintácticamente coherente. Así, podríamos decir que en la sección 'variable' se describen aquellos objetos cuya funcionalidad se describe con las variables de Dharma construidas en la sección 'value'. Esto se apreciará mejor con la gramática construida para fuzzear el objeto Thermometer.

<br>

**Variance Section**

Por último vamos a presentar la sección 'variance'; en ella se define la estructura principal de la plantilla que se repetirá una y otra vez mientras que se varían cada una de las partes que la componen de acuerdo a las variaciones de la sección 'value'.

La idea es que esta estructura que se define sea una amalgama de las partes que se han definido en las secciones anteriores, de forma que Dharma crea la misma estructura una y otra vez variando las partes constituyentes de la misma, obteniendo así iteraciones ligeramente distintas del mismo modelo.

```
%section% := variance

main :=
	try {console.log(@string@)}catch(e){}
```

<br>

### Recapitulando...

Hemos definido una plantilla que se basa en tres secciones:

- Variable: Donde se definen las variables del lenguaje de programación que van a formar parte de los scripts.
- Value: Donde se define la funcionalidad y la acción del script en base a los objetos definidos en la sección 'variable'. Es decir, en 'variable' definimos un objeto y en 'value' definimos su funcionalidad (propiedades, métodos, etc) orientado al fuzzeo.
- Variance: Se plantea la esctructura principal que se itera mutando cada parte constituyente de la misma en base a la sección 'value'.

Poniéndolo todo junto la gramática queda como sigue:

```
%section% := value

name :=
	Tom
	Jerry
	Bunny

message :=
	I want to play with +name+!

%section% := variable

string :=
	var @string@ = "+message+"

%section% := variance

main :=
    try {console.log(@string@)}catch(e){}
```

Y si procesamos el código obtenemos el siguiente output en alguno de los ficheros:

```
gsanmillan@Ubuntu:~/Documents/Grammars$ dharma -grammars test4.dg -count 2 
[Dharma] 2024-03-12 17:37:34,417 INFO: Machine random seed: 3274314972506318399
[Dharma] 2024-03-12 17:37:34,417 DEBUG: Using configuration from: /home/gsanmillan/.local/lib/python3.10/site-packages/dharma/settings.py
[Dharma] 2024-03-12 17:37:34,418 DEBUG: Processing grammar content of ../../.local/lib/python3.10/site-packages/dharma/grammars/common.dg
[Dharma] 2024-03-12 17:37:34,421 DEBUG: Processing grammar content of test4.dg
var string1 = "I want to play with Bunny!"
var string2 = "I want to play with Jerry!"
var string3 = "I want to play with Bunny!"

try {console.log(string1)}catch(e){}
try {console.log(string2)}catch(e){}
try {console.log(string3)}catch(e){}
try {console.log(string2)}catch(e){}
try {console.log(string2)}catch(e){}
try {console.log(string1)}catch(e){}

var string1 = "I want to play with Jerry!"
var string2 = "I want to play with Jerry!"
var string3 = "I want to play with Bunny!"
var string4 = "I want to play with Tom!"

try {console.log(string1)}catch(e){}
try {console.log(string2)}catch(e){}
try {console.log(string3)}catch(e){}
try {console.log(string4)}catch(e){}
try {console.log(string1)}catch(e){}
```

Podemos comprobar que se ha generado código javascript sintácticamente correto. Se han definido por un lado las variables que son las estructuras definidas en la sección 'variable' y por el otro la estructura definida en la sección 'variance' donde se ha repetido una y otra vez el mismo modelo mutando las partes declaradas en la sección 'value'.

Si procesamos este código con un engine de javascript obtenemos:

```
gsanmillan@Ubuntu:~/Documents/Grammars$ dharma -grammars test4.dg -count 2  > output.js; node output.js
[Dharma] 2024-03-12 17:40:10,715 INFO: Machine random seed: 2848579258711801962
[Dharma] 2024-03-12 17:40:10,715 DEBUG: Using configuration from: /home/gsanmillan/.local/lib/python3.10/site-packages/dharma/settings.py
[Dharma] 2024-03-12 17:40:10,716 DEBUG: Processing grammar content of ../../.local/lib/python3.10/site-packages/dharma/grammars/common.dg
[Dharma] 2024-03-12 17:40:10,719 DEBUG: Processing grammar content of test4.dg
I want to play with Bunny!
I want to play with Tom!
I want to play with Tom!
I want to play with Tom!
I want to play with Tom!
```


<br>

### Thermometer.

**Definición del objeto**

Para ilustrar mejor cómo podemos manejar Dharma, vamos a realizar una serie de ejemplos de como ciertos usuarios han implementado en Dharma la sintaxis de un objeto de JavaScript.

En este caso, vamos a intentar implementar a la API de Dharma el objeto: [Thermometer](https://opensource.adobe.com/dc-acrobat-sdk-docs/library/jsapiref/JS_API_AcroJS.html#id1055)

De acuerdo a la documentación, el objeto Thermometer es un objeto que sirve para indicar el progreso de una operación en ejecución a través de una barra de progreso y una ventana. Además, se nos ofrece un script de ejemplo acerca de cómo implementar este objeto:

```javascript
var t = app.thermometer;         // Acquire a thermometer object
t.duration = this.numPages;
t.begin();
for ( var i = 0; i < this.numPages; i++)
{
    t.value = i;
    t.text = "Processing page " + (i + 1);
    if (t.cancelled) break;       // Break if the operation is cancelled
    .
.. process the page ...
}
t.end();
```

Esta información es valiosísima pues nos ayudará a implementar en Dharma la sintaxis de este objeto de manera precisa.

<br>

Queda claro por tanto que de lo priemero que tenenmos que ocuparnos es de definir en Dharma la definción de un objeto "thermometer". Esto se realiza a modo de variable en la sección 'variable', de forma que iniciaríamos nuestra gramática definiendo primero la sección variable y un objeto thermometer como se ilustra en el script anterior:

```
%section% := variable

obj :=
    var @obj@ = app.termometer
```

<br>

**Funcionalidad del objeto**

Ahora que hemos definido el objeto, lo que buscamos es construir una plantilla que, a través de la lógica de iteración de Dharma, haga un planteamiento exhaustivo de la funcionalidad del objeto. Queremos crear una plantilla que haga que Dharma sea capaz de construir scripts que realizen cualquier acción que pueda realizar el objeto.

<br>

**Propiedades del objeto**

En la programación orientada a objetos llamamos a objeto a un conjunto de definiciones de entre las cuales diferenciamos propiedades; que son valores asociados al objeto en un instante del programa y métodos; que son funcionalidades del objeto. 

En este caso nos ocupamos de las propiedades, atendiendo a la documentación observamos que son cuatro:

- Cancelled - Boolean
- Duration - Integer
- Text - String
- Value - Integer

Cada propiedad es un valor que adaptaremos en la sección 'value'. Además, estos valores por la propia funcionalidad del código, pueden ser asignados (setteados):

```
%section% := value

thermometer_setter :=
	!thermometer_obj!.value = +common:number+
	!thermometer_obj!.text = +common:character+
	!thermometer_obj!.duration = %range%(0-10000)
	!thermometer_obj!.cancelled = +common:bool+
```

Observemos que en el paso anterior estamos realizando los siguientes pasos:

- En primer lugar, generamos la sección 'value'.
- Seguidamente, definimos una variable en Dharma que contiene los nombres de las propiedades del objeto thermometer.
- Por último, generamos otra variable en Dharma que contiene a modo de valores un conjunto de plantillas en JavaScript que se fabrican de la siguiente manera: Se trata de generar una asignación de una variable en JavaScript, para ello se realiza una operación de asignación que consta de una referencia al objeto definido en la sección 'variable' y una importancia del fichero common.dg del tipo de dato de cada asignación.

O extraidos del propio objeto (getteado):

```
thermometer_prob :=
	value
	text
	duration
	cancelled

thermometer_getter :=
	var x+common:digit+ = !thermometer_obj!.+thermometer_prob+
```

Así, hemos construido plantillas que intentan realizar operaciones sobre dicho objeto asignando y extrayendo propiedades del mismo.

<br>

**Métodos del objeto**

Por último, prestamos atención a los métodos, que son la abstracción de acciones que hace el objeto en cuestión. Estos no tienen un valor asociado si no una acción con ciertos valores u otros objetos. 

Atendiendo a la documentación, observamos que existen 2 métodos;

- begin()
- end()

De esta forma, implementamos una plantilla de forma muy simple: 

```
thermometer_functions :=
	begin()
	end()
```

De esta forma, aunamos las estructuras definidas en una variable que denominaremos wrapper, que esencialmente resume la funcionalidad del objeto a fuzzear:

```
wrapper :=
	try{+thermometer_setter+;}catch(e)
	try{+thermometer_getter+;}catch(e)
	try{!thermometer_obj!.+thermometer_functions+;}catch(e)
```

Observemos que estamos implantando todas las funcionalidades relacionadas con el objeto, esta variable será referenciada en la sección 'variance' para ser iterada sucesivamente cambiando pequeñas cosas:

```
%section% := variance
main :=
	+wrapper+
```

El script al completo quedaría de la siguiente forma:

```
%section% := value

thermometer_prob :=
	value
	text
	duration
	cancelled

thermometer_getter :=
	var x+common:digit+ = !thermometer_obj!.+thermometer_prob+

thermometer_setter :=
	!thermometer_obj!.value = +common:number+
	!thermometer_obj!.text = +common:character+
	!thermometer_obj!.duration = %range%(0-10000)
	!thermometer_obj!.cancelled = +common:bool+

%section% := variable

thermometer_obj :=
	var @thermometer_obj@ = app.thermometer;

%section% := variance

main :=
	+wrapper+
```
