---
layout: post
title: Análisis de Objetivos
subtitle: Introducción a OSINT y análisis de objetivos de ataque.
tags: [redteam]
---

# 1. Introducción OSINT.

## 1.1 Análisis de Objetivos.

### 1.1.1. Índice.

- Qué es.
- Para qué sirve.
- Qué es OSINT.
- Google Hacking.
- Búsqueda de información por buscadores.
- Perfil Social
- Perfil Tecnologico.
- Información adicional a obtener.

<br />

### 1.1.2. Qué es el análisis de objetivos.

Se trata de una fase en la que se busca recopilar toda la información básica acerca del objetivo para poder definir un plan de ataque. Por ejemplo:

- Localización de la compañia / Residencia de la persona.
- Trabajadores de la compañía / Lugar de trabajo de la persona.
- Correos electrónicos.
- Infraestructura informática.
- Metadatos en documentos públicos.

Para buscar esta información empleamos:

- Google.
- Registro Mecantil: eInforma, datosCIF, empresia.es, boe, rmmercantil.es.
- Redes Sociales:
	- Web de la empresa/persona.
	- Twitter/Facebook.
	- Opiniones en foros y compañía.


<br />

### 1.1.3. Qué es OSINT?

Es un término creado y utilizado por militares, fuerzas del orden y personal de inteligencia de las agencias gubernamentales. Se trata de un acrónimo que refiere a "Open Source Intelligence" Inteligencia de Fuentes abiertas.

Las fuentes OSINT se refieren a cualquier información desclasificada y públicamente accesible en internet de forma gratuita.

- Es lo contrario a las fuentes de información clasificadas, reservadas o de pago (bases de datos de los cuerpos y fuerzas de seguridad del estado).

- Son abiertas y gratuitas.

- Son fuentes PASIVAS de información. No estás accediendo a servidores, máquinas de la empresa para tener acceso a dicha información, esta se encuentra pública en la red.

- Las más valiosas suelen estar en la DeepWeb (Internet profunda) que a su vez no está registrada en los motores de búsqueda. Accesible mediante el buscador TOR.

- OSINT incluye tanto a las fuentes de información de la internet superficial como de la internet profunda.

<br />

### 1.1.4. Para qué sirve OSINT?

- OSINT sirve para hacer inteligencia, buscar y obtener información sobre la que tomar decisiones.

- Estas decisiones se toman según los datos recopilados que puede ser sobre una entidad, persona, sistema o empresa. En nuestro caso, Red Team, servirá para realizar el modelado de amenazas y estrategias de ataque.

- También puede ser útil para llevar a cabo otras tareas como:
	- Tener una situación más ventajosa en una empresa de trabajo.
	- Conocer la situación actual de la empresa.
	- Reuniones de investigación.


A nivel de persona, estudiamos dos apartados:

- **Datos tecnológicos**: Direcciones de correo electónico, servidores, IPs, dominos, usuarios, sede social, registro mercantil...
- **Datos personales**: Dirección de vivienda, usuarios de redes sociales, familia, etc.

<br />

### 1.1.5. Proceso de gestion de OSINT.

1) ¿Dónde buscar?
2) Recopilar
3) Procesar
4) Analizar
5) Generar Informe.

<br />

## 1.2. Herramientas a utilizar.

- Herramientas Online (Web)
	- No es necesario instalarlas (pueden usarse desde cualquier lugar desde el que se tenga conexión a internet).
	- Generalmente llevan a cabo búsquedas en motores o BBDD existentes.
	
- Comandos y Aplicaciones Linux / Windows.
	- Comándos estándar de consulta de infraestructura tecnológica.
	- Aplicaciones propias de OSINT.
	- Simplifican la festion de la información adquirida.
	- Utilizan generalmente APIs de terceros.
	- Automatizan informes

<br />

### 1.2.1. Herramientas Online.

#### 1.2.1.1. Whois Domain Tool.

Se trata de una página web que te permite buscar información relativa a un dominio. (Si está libre, a quién pertenece, cuándo expira ...).

Pueden existir registro También podemos utilizar NIC.es (cada país tiene el suyo propio). Whois puede no tener acceso a determindas bases de datos, pero sin embargo te dirá a qué sitio ir en caso de no tener acceso a dicho dato.

Aunque también se trata de un comando de kali, es decir, también se puede utilizar como comando.

Nota: Si el apartao DNSSEC no está firmado : unsigned, es una capa de seguridad menos.

Proporciona información como:

- Si el registro está o no cogido.
- Dominio de registro ID.
- Fecha de creación.
- Fecha de de update.

<br />

#### 1.2.1.2. IPv4info.
Página web. Da información acerca de la IP proporcionada, también le podemos pasar con dominios, ASN (numero de asignación que se le da a ciertos autónomos) o el nombre de una compañía. Ofrece información en forma de tabla de:

- Rango de IPs asocidas al dominio.
- Tamaño del rango.
- Dominios asociado al mismo AS (identificador internacional).
- Identificador.
- Organización a la que está asociada todos esos dominios.
- País.
- Status del dominio.

<br />

#### 1.2.1.3. DNS Dumpster.
Ofrecen información tanto gráfica como textual acerca de elementos relacionados con el DNS de un dominio:

- **Hosting**: El hosting es un servicio que ofrece alojamiento para contenidos de páginas web o correos. En este caso se ofrece como gráfica que muestra una barra respecto del tanto porciento.

- **GeoIP Host Location**: Localización de los host a los que el dominio ofrece servicio.

- **DNS servers**: El DNS es un protocolo que se traduce en el "sistema de nombres de domino" y se encarga de asociar una IP a un dominio y viceversa, de forma que un servidor DNS es un servidor que asocia la IP al domino para el dominio o rango de dominios que estamos buscando en concreto.

- **MX Records**: Estos son los registros de los servicios MX (mail exchanger), que se trata de una máquina que recoge todos los emails que gente que tiene cuenta en dicho dominio.

- **TXT Records**: Registros de ficheros que contienen alguna referencia al dominio.

- **Host Records**:Registros de operaciones hechos con dicho dominio por parte de máquinas. 

- **Mapeado del dominio**
	
<br />

#### 1.2.1.4. The wayback machine (archive.org).

Ofrece información acerca de páginas web asociados a un dominio archivadas a lo largo del tiempo junto con:

-	Sus distintas versiones guardadas en snapshots. Además, si se paga está antigua versión puede ser restaurada.
-	El número de veces que ha sido modificada.
-	Cuándo han sido modificadas. Ordenado en un calendario.
	
<br />

#### 1.2.1.5. robtext.com 
Robtext es una página web utilizada para recoger información general acerca de números IP, nombres de dominio, etc.

Utilitza fuentes públicas o directamente hace uso de otras páginas web para recopilar información que almacena en una fuente de datos y finalmente te da acceso a dichos datos.

Muy útil, tiene el mismo mecanismo que VirusTotal, recopila informaicón de sitios web y ordena y muestra la inforamación recopilada.

<br />

### 1.2.2. Herramientas y aplicaciones de Linux/Windows.


#### 1.2.2.1. Whois. 

Comando que opera igual que la herramienta online.

<br />

#### 1.2.2.2. Host.
Es una herramienta para hacer búsquedas DNS, se utiliza normalmente para identificar la IP que pertenece a un dominio y viceversa. Dónde lo único obligatorio es indicar el nombre de la máquina (dominio o subdomino) a investigar. 

Se le puede añadir información adicional, si se sabe, por ejemplo modificadores o distintos dominios para poder contrastar si la información que tenemos es la misma.

Puede proporcionar la siguiente información:

- Gestor de emails mx (mail xchanger **mirar**).
- Los alias que se utilizan como redirecciones para poder abarcar más dominios.
- IP asociado al dominio.
- IPs redundadas.

y más.

<br />

#### 1.2.2.3. Nslookup.

Supuestamente obsoleto. Aunque sigue teniendo algunas utiliades.

- Servidor que nos conectamos.
- IP y puerto de conexión.
- Si se trata de un DNS auténtico o réplica (DNS falso que tiene la información cacheada) y en el caso de que se trata de una réplica indicarlo, también podemos saber el DNS original aplicando 'whois' o nic.es.

<br/>
	
#### 1.2.2.4. dig
El comando **dig** se utiliza de la siguiente forma:

```bash
dig [server] [name] [type]
```

Donde:

- **server**: El nombre del host o la dirección ip sobre la que está diriga la búsqueda.
- **name**: El dominio del servidor a buscar.
- **type**: El tipo de registro DNS. Por defecto dig utiliza un tipo A.

Se trata de un comando superior a los anteriores, traslada el mismo tipo que los anteriores pero además admite muchos tipos de funcionalidades. 

Se pueden con modificadores los tipos de consultas que se quieren hacer. Por ejemplo, axfr (transferencia total de zona) es un modificador que te da acceso a todo, sin embargo esto no está permitido y muchas configuraciones de seguridad no lo permite. 

<br />


#### 1.2.2.5. urlcrazy.

Te dice todo los nombres asociados a un dominio y te dice si dicho nombre está o no registrado basado . Cambia TLDs, SLD, caracter repetidos, quita caracteres...

<br />

#### 1.2.2.6. dnsenum.

Automáticamente identifica registros básicos de DNS como MX, NS.

<br />

#### 1.2.2.7. sublist3r -d.

Para utilizarlo con un dominio. Busca subdominios.

<br />

#### 1.2.2.8. theHarvester. 

Tiene muchos modificadores, un ejemplo sería:

```bash
theHarvester -d thebridgeschool.es -l 1000 -b linkedin
```

theHarvester es una herramienta poco fiable que hay que meter varias veecs en distintos momentos al día.

Ofrece información de email, personas, máquinas, IP, casi de todo.
Con este comando le estamos diciendo que busque el domino thebridgeschool.es, una búsqueda en linkedin, que busque 1000 búsquedas.

Si queremos obtener ayuda sobre un comando metemos -h. 

<br />

# 2. Buscadores y Metadatos.

## 2.1 Buscadores.

### 2.1.1. Libreborme.
Una págian librebor.me es para buscar información mercantil de empresas con el CIF o el nombre con el que dicha empresa está registrada en el registro mercantil.

Ofrece información como:

- NIF.
- Situación Mercantil.
- Fecha de constitución.
- Domicilio.
- Objeto social.
- Capital social.

- Cargos directivos tantos activos como no activos.

Tenemos un número de búsquedas limitadas.

<br />

### 2.1.2. Infocif.es.

De nuevo, se trata de una página en la que buscamos una empresa por Nombre, Razón Social (Nombre del registro mercantil) o NIF.

<br />

### 2.1.3. boe.es.

Con Ctrl+f puedes buscar un nombre, y de esa forma puedes acceder a los informes del boe en busca de una empresa o una persona. 

El boe es propio de españa, en ese sentido, hay que buscar el análogo en otros países.

<br />

### 2.1.4. Explotid-db.com.

Exploit Database (ExploitDB) es un archivo de exploits para propósito de la seguridad pública.

Se trata de un recurso que te puede ayudar a identificar posibles debilidades en tu red y para estar al día sobre posibles ataques ocurridos en otras redes.

Funciona enseñandote distintas archivos sobre vulnerabilidades en relación a la palabra que tu metas en tu búsqueda.

<br />

### 2.1.5. Shodan.io.

Shodan es un motor de búsqueda para encontrar dispostivos específicos y tipos de dispositivos. Las búsquedas más populars son cosas como "webcam", "linksys", "cisco", etc.

Funciona escaneando internet entero y parseando los banners que devuelven diversos dispositivos. De esta forma, Shodan puede darte información acerca de los dispositivos relacionados con tu búsqueda, más concretamente, te devuelve la red que está creada por distintos dispostivos que están conectados a internet, recopilan entonces dicha información y acto seguido te la enseña, pudiendo tener acceso al sistema operativo que se está utilizando, los puertos que se emplean, etc.

<br />

#### 2.1.5.2. Guía de Shodan.io.

##### 2.1.5.2.1. Webcams.

Lo primero que hay que precisar es que Shodan.io es un buscador técnico, requiere de búsquedas de dispositivos específicos haciendo uso de filtros. En este caso, se buscaría un modelo de cámara de alguna marca conocida y se buscaría

Se lleva a cabo una búsqueda y tiene aparecen los dispositivos encontrados relacionados con esta búsqueda junto con su IP el protcolo y el puerto y si está abierto o no.

<br />

### 2.1.6. censys.io

Página que permite buscar vulnerabilidades. Y ampliar la superficie de ataque. De pago.

<br />

### 2.1.7. Yandex.com

Búsqueda fotos relacionadas con una palabra.

<br />


## 2.2 Google's Dork.

Cómo hacer uso eficiente de las búsquedas de google. Estudiar google dork. Se pueden ver en https://securitytrails.com/blog/google-hacking-techniques 

<br />

### 2.2.1. Qué es Google Dork?

Un google dork (también conocido como Google Dorking o Google Hacking), es una valiosa herramienta de búsqueda para investigadores de seguridad.

Google tiene una capacidad enorme para el rastreo web y puede indexar una gran cantidad de información que se encuentre dentro de tu sitio web, incluyendo información sensible. Esto significa que un sitio web puede estar exponiendo más información de la cuenta. Es decir, Google Dorking consiste en la práctica de buscar servidores y sitios web vulnerables mediante el **uso de las capacidades naticas del motor de búsqueda de google**.

A menos que bloquee recursos específicos de su sitio web mediante un archivo; **robots.txt**, google indexa toda la informacíon que este presente en un archivo web. Así, cualquier persona puede tener acceso a esa información si sabe qué buscar. También puede tener acceso a la base de datos de piratería de Google que contiene todos los comandos de dorking de google. Esta información está disponible públicamente en Internet, y Google la proporciona y alienta a que se use de manera legal. A parte del hecho de que Google tiene acceso a su identidad mientras realiza estas búsquedas y por tanto no es tán anonimo como piensa.

<br />

## 2.3. Filtraciones.

Filtraciones, filtración de datos debido a una vulnerabilidad. 

Para saber si tu cuenta ha sido vulnerada:

- **haveibeenpwned.com**: Metes tu correo electrónico y te dice si ha habido alguna filtración en la que has estado involucrado.

- **pastebin.com**. Muestra filtraciones.

- **StalkFace.com**: Estalkea perfil de facebook de una víctima. Existen páginas web que también estalquean otros perfiles.

<br />

## 2.4. API key

### 2.3.1. Qué son las APIs.

API es el acrónimo de "Application Programing Interface" (Interfaz de programación de aplicaciones). A su vez, una interfaz es un lazo compartido entre o más componentes separados de un sistema informático que permite a estos intercambiar información.

Se trata de algo así como un intermedario que permite que dos aplicaciones puedan intercambiar datos entre sí. En otras palabras es el protocolo de comunicación que utilizan dos aplicaciones para intercambiar datos.

Las APIs son interfaces que ayudan a construir software y definen cómo las piezas de software interactuan entre sí. Se utilizan comunmente en dispositivos IoT, ya sean aplicaciones o servidores web con la finalidad de recopilar o procesar datos, (token functionality).

<br />

### 2.3.2. API Keys.

Las API Key son un identificador, un código utilizado para identificar y autenticar a una aplicación o a un usuario que pretende utilizar una API para modificar el código de una aplicación externa. Estas son accesibles mediante plataformas. También actuan como identificadores y proveen token secretos para propósitos de autenticación. 

Una aplicación que utiliza APIs, lo que hace es llamar a la API para pedir información sobre el usuario. De esta forma se llega a controlar qué usuarios tienen acceso a la aplicación, cómo el usuario está utilizando los datos de la aplicación, etc.  

<br />

## 2.4. Metadatos: Datos acercas de los datos.

La definición más concreta de los metadatos es qué son "datos acerca de los datos" y sirven para suministrar información sobre los datos producidos, por ejemplo una foto o un fichero. Es decir, consisten en información que caracteriza datos, describen el contenido, calidad, condiciones, historia, disponibilidad y otras características de los datos. 

Los Metadatos permiten a una persona ubicar y entender los datos, incluyen información requerida para determinar qué conjuntos de datos existen para una localización geográfica particular, la información necesaria para determinar si un conjunto de datos es apropiado para fines específicos, la información requerida para recuperar o conseguir un conjunto ya identificado de datos y la información requerida para procesarlos y utilizarlos.

**La generación de Metadatos no solo es aplicable a la información digital, también debe aplicarse a cualquier conjunto de datos independientemente del soporte en el cual se encuentren**, ya que ello puede facilitar su localización, y así, agregarle un valor añadido a la información histórica con la que cuenta una entidad.

Los **metadatos de información georreferenciada** con que cuenta una organización o un País, constituyen elementos centrales de sus Infraestructuras de Datos Espaciales, pues solo a través de éstos es posible conocer las características de los datos georreferenciados existentes, buscarlos y seleccionar los datos que se necesitan.

A través de los metadatos, se pueden responder las siguientes preguntas para saber si un conjunto determinado de datos se ajusta a nuestras necesidades:

-   ¿Dónde se originó?
-   ¿Qué pasos se siguieron para crearlo?
-   ¿Qué atributos contiene?
-   ¿Cómo están proyectados los datos?
-   ¿Qué área geográfica cubre?
-   ¿Cómo obtener la información completa?
-   ¿Cuánto cuesta?
-   ¿Con que persona se puede contactar para obtener una copia?, etc.

<br />

### 2.4.2. Metagoofil.

Se trata de un comando que permite la extración de metadatos de documentos públicos de una página web, se le pasa como argumento un dominio.

```bash
metagoofil -d nmap.org -t pdf -l 200 -n 10 -o /tmp -f 
```

<br />

### 2.4.3. FOCA (Windows).

Es una herramienta de windows, que te permite extraer metadatos de documentos. A esto si le puedes pasar un fichero manuelamente.

<br />

### 2.4.4. Exiftool.

Lee o manipula datos de archivos.  Se puede descargar como herramienta para kali y también le puedes pasar ficheros. 

<br />

# 3. Personalidad ficticia y Spiderfoot.

## 3.1. Personalidad Ficticia.

Para poder crear una personalidad fictica utilizamos las siguientes herramientas: 

<br />

### 3.1.1. This person does not exist.

https://thispersondoesnotexist.com/

Esta página web genera una cara aleatoria combinando múltiples caras de personas de imágenes públicas. De forma que aunque pueda ser realista en ningún caso se trata de una suplantación de identidad.

<br />

### 3.1.2. fake name generator

https://www.fakenamegenerator.com/

Esta página construye una identidad falsa pero con ciertos márgenes de realismo de la siguietne forma:

- **Nombre**: los nombres se generan al extraer aleatoriamente un nombre y un apellido de una base de datos. La base de datos se compiló a partir de fuentes de dominio público. Lo más probable es que alguien de entre los miles de millones de personas que han vivido en este planeta haya tenido el mismo nombre que el que se generó. Sin embargo, te aseguramos que estos nombres se generan aleatoriamente.

- **Dirección postal**: el número de la casa es un número generado aleatoriamente. El nombre de la calle se extrae de una base de datos de nombres de calles plausibles para el estado / país que se está generando. Lo más probable es que la dirección postal generada no sea válida.

- **Ciudad, estado y código postal**: Hemos compilado una base de datos que contiene cientos de miles de combinaciones válidas de ciudad, estado y código postal. Una de estas combinaciones se extrae aleatoriamente de la base de datos para cada identidad.

- **Dirección de correo electrónico**: la dirección de correo electrónico generada tiene el formato Nombre generado @ Servicio de correo electrónico anónimo.com. Los servicios de correo electrónico anónimo son proporcionados de forma gratuita por Fake Mail Generator, **nuestro servicio de correo electrónico temporal**. Este puede ser utilizado desde la página web.

- **Número de teléfono**: Hemos compilado una base de datos de prefijos y códigos de área válidos. Una de estas combinaciones se extrae aleatoriamente de la base de datos y luego se agrega un número aleatorio de la longitud adecuada al final para que el número de teléfono tenga la longitud correcta.

- **Apellido de soltera de la madre**: Se extrae un nombre aleatorio de nuestra base de datos de apellidos y se enumera como "apellido de soltera de la madre".

- **Cumpleaños**: el cumpleaños es una fecha generada aleatoriamente.

- **Tarjeta de crédito**: utilizamos el generador de tarjetas de crédito PHP de Graham King. **Este script crea un número de tarjeta de crédito falso, pero con sintaxis válida**. La fecha de vencimiento se genera aleatoriamente para que sea una fecha próxima. **Usamos prefijos inactivos para asegurarnos de que estas tarjetas no se utilicen con fines fraudulentos. Estos números de tarjetas de crédito no reflejan el número de tarjeta de crédito real de nadie**.

- **Número de identificación nacional**: los números de seguro social se generan utilizando el patrón descrito por la Administración del Seguro Social. Los números de seguro social se generan utilizando el patrón descrito en Wikipedia. Estos números son completamente aleatorios y **es muy poco probable que coincidan con el nombre generado**.

<br />

## 3.2. Spiderfoot.

Spiderfoot es una aplicación que se puede encontrar en Github. Se trata de una herramienta de reconocimiento que automáticamente buscar sobre 100 fuentes de datos públicos (OSINT) para recopilar información de inteligencia sobre:

- Direcciones IP.
- Dominios.
- Direcciones e-mail.
- nombres y más.

Se utiliza pasándole una IP y un puerto, separándo la ip del puerto con dos puntos.

El puerto debe de estar por encima de 1024 y no se puede estar usando previamente.

Con esta herramienta montamos una especie de servidor web al que podemos acceder metiendo la IP proporcionada al crearse la página web. Por defecto, introduciremos nuestra IP loopback.


Al realizar un scaneo e introducir la dirección IP de nuestro objetivo, esta misma web recopilará toda la información de dicho sitio que se conecta y la guarda y la muestra por dicho servidor web. Se trata de una herramienta espía. Especialmente eficaz contra un router.

De esta forma, la herramienta se utiliza dándo click en nuevo escaneo y añadiendo el nombre del escaneo y la IP del nuestro objetivo.

<br />

# 4. Sherlock, Epieos, Maltego
## 4.1. Sherlock. 



Busca nombres en cuentas de redes sociales. Hay que descargarla desde github aunque también puede descargarse como un paquete de python desde la apt.

**El manual contiene un fallo, si se descarga como un script de python en github primero hay que trasladarse a la carpeta ./sherlock para instalar la aplicación y después hay que trasladarse de nuevo a la carpeta /sherlock/sherlock para utilizar sherlock.**

Existe un archivo, (una vez instalado la aplicación) que se llama README.txt dentro de la carpeta sherlock que actúa a modo de instrucciones.

En ella podemos encontrar los siguientes elementos interesantes:

- Las cuentas encontradas tras una búsqueda se almacenarán en un fichero de texto cuyo nombre se corresponde con el nombre de la búsqueda.

Suplantación de identidad.

<br />

## 4.2. Epieos.

Se trata de una página web a la que le puedes pasar una dirección de correo o un teléfono y hace OSINT sobre dicha dirección de correo electrónico o teléfono.

<br />

## 4.3. Maltego.
 
 Existen distintas versiones de maltego, nosotros utilizaremos la versión que obtienes nada más logearte. Necesitas estas logeado para utilizar la versión CE, comunity edition.
 
 Maltego es una aplicación que ofrece un servicio con el potencial de encontrar información sobre personas y empresas en internet, permitiendo cruzar datos para obtener perfiles en redes sociales, servidores de correo, etc.
 
 Maltego, como ya hemos dicho contiene dos versiones, la básica y la profesional, la diferencia entre ambas estriba en el número de módulos disponibles que tiene cada uno. Un módulo no es más que una funcionalidad extra que se añade al servicio de rastreo que ofrece Maltego.
 
### 4.3.1. Realizando un rastreo.
 
 Una vez realizada la isntalación e iniciada la sesión, deberemos crear una hoja de búsqueda nueva y arrastrar la entidad (este es el tipo de búsqueda que queremos hacer) para luego ver la búsqueda y comprobar qué resultados arroja.
























