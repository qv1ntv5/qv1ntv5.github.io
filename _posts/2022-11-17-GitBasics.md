---
layout: post
title: Git Basics
subtitle: Introducción a Git y comandos de la cli git.
---

### Git Intro.

Git es un VCS (Systema de control de versiones) empleado fundamentalmente para operaciones de intercambio de código, seguimiento de los cambios del código compartido y seguimiento de aquellos individuos que modifican el código. Sin embargo, el procedimiento de Git es novedoso porque, a diferencia de otros VCS, que al realizar cambios a modo de conjuntos de diferencias que cambian en la copia online del código,

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221116102951.png' | relative_url }}" text-align="center"/>
</div>

Git cambia toda la copia en conjunto por otra copia modificada (snapshot) introduciendo así los cambios como una modificación total de la copia original existente.

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221116103022.png' | relative_url }}" text-align="center"/>
</div>

Esto tiene la ventaja de que todas las modificaciones que se hacen del projecto son locales y sólo se produce un cambio online cuando el usuario ha terminado con todas las modificaciones y ejecuta la subida al servidor. Sin embargo, todas las pequeñas modificaciones del proyecto quedan almacenadas y registradas en la copia offline.

Además, en cada clonación (copia local) de un repositorio, git añade una pequeña base de datos en la que introduce toda la información relativa a ese repositorio incluidos cambios anteriores de forma que vuelve esta información rápidamente accesible y sin necesidad de estar conectado a internet.

Así, git tiene tres pasos fundamentales a la hora de realizar cambios: modified (se han producido cambios pero no se han registrado en la base de datos), staged (Se marcan los ficheros modificados del repositorio para su posterior registro) and committed (El commit es el registro de los datos en la base de datos).

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221116104400.png' | relative_url }}" text-align="center"/>
</div>

<br />

### Git Basics.

**.git and cloning.**

Un repositorio como ya hemos indicado es la copia de un proyecto en un servidor, de forma que se puede decir que es una copia que puede ser local u online. En una máquina local se puede obtener un repositorio git de dos formas:

Puedes transformar un directorio local que no está actualmente bajo Git en un repositorio Git:

```bash
mkdir /home/user/myproject
cd /home/user/myproject
git init
```

Esto creará un subdirectorio en la carpeta llamado '.git' que contiene lo que se denomina como el 'Git repository skeleton'; un conjunto de ficheros y carpetas que contienen todo lo necesario para hacer del directorio un repositorio git.

O también puedes clonar un repositorio en la nube en un directorio local.

```bash
git clone https://github.com/libgit2/libgit2
```

Esto crea un directorio llamado 'libgit2', inicializa dentro de él el directorio '.git' y descarga el repositorio.

<br />

**Recording changes to the repository**.

Una vez tenemos un repositorio que contiene un trabajo iremos poco a poco provocando cambios en él. Hay que tener en cuenta que cada fichero que contenemos en el directorio puede tener dos estados: 'tracked' o 'untracked'. Los ficheros 'tracked' son ficheros de los que el repositorio .git del directorio tiene conocimiento; estos pueden ser directorios que fueron añadidos en anteriores snapshots (commits) o que fueron añadidos (add) mientras que los 'untracked' son el resto de ficheros.

Así por ejemplo cuando clonas un repositorio, todos los ficheros son 'tracked' en la medida en la que empiezas a modificar el repositorio y a añadir ficheros, todos esos ficheros recientes son untracket hasta que ejecutas 'git add' y posteriormente un 'git commit'. En ese momento git los registra y adquiere conocimiento de ellos con lo que pasan a ser 'tracked'

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221116125454.png' | relative_url }}" text-align="center"/>
</div>


Podemos emplear el comando 'git status' para determinar qué ficheros están en qué status. Si ejecutáramos el comando nada más realizar un clone recibiríamos el siguiente mensaje:

```bash
git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working tree clean
```

Pero en la medida en la que comenzáramos a trabajar obtendríamos:

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221116130742.png' | relative_url }}" text-align="center"/>
</div>

<br />

Debemos observar que además de los untracked files, este comando te notifica en que 'branch' estas. La rama principal siempre será origin/master o main.

Así podemos concluir que 'untracked' significa que el comando git a notado la existencia de dos ficheros nuevos que no estaban en un snapshot previo. Para incluir los ficheros nuevos debemos usar el comando:

```bash
git add <filename>
```

Aunque por costumbre también se utiliza 'git add .' que significa añadir todo el directorio.

También podemos hacer que 'git status' ignore ciertos ficheros a través del fichero '.gitignore' en el que se pueden incluir nombres, patrones, directorios, etc.

Por último, si el output que despliega el comando 'git status' es un poco vago, podemos ampliar los contenidos de las variaciones entre los snapshots mediante el comando 'git diff'.

Examinamos el output más detenidamente.

<br />

**Git Diff Internals**

Diff es una función que toma dos datos del usuario a modo de input y devuelve las diferencias o los cambios del uno al otro. Concretamente:

```bash
git diff <commit1> <commit2>
```

Por defecto, el comando asumirá que el commit2 es el más antiguo y el commit1 el más reciente y siguiendo esta lógica desplegará el output. Así por ejemplo supongámos un caso sencillo de un repositorio en el que existe un fichero cuyo contenido a cambiado de "this is a git diff test example" a "this is a git example":

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221117102304.png' | relative_url }}" text-align="center"/>
</div>

Examinando el output más detenidamente encontramos las siguientes partes:

La primera línea:

```bash
diff --git a/diff_test.txt b/diff_test.txt
```

Indica que ha encontrado diferencias en un fichero existente en ambos snapshots, a/diff_test.txt pertenecería al antiguo, al commit1 y b/diff_test.txt pertenecería al commit2.

```default
--- a/diff_test.txt
+++ b/diff_test.txt
```

Esto se trata de simología que actúa a modo de leyenda, los cambios de a/diff_test.txt vendrán marcados con un '-' y los de b/diff_test.txt con '+'.

Por otra parte, diff sólo despliega las secciones del fichero en los que comprueba que hay cambios. Concretamente estos cambios se leen en términos de líneas:

```bash
@@ -1 +1 @@
```

Esto significa que por un lado, la línea -1 (esto es, la línea 1 del fichero a/diff_test.txt) ha sufrido cambios así como la línea +1 (esto es, la línea 1 del fichero b/diff_test.txt). Esto es un caso muy simple, pero en un contexto más real tendríamos:

```default
@@ -34,6 +34,8 @@
```

Esto es que en el fichero del commit1, a partir de la línea 34, 6 líneas han sufrido cambios y en el fichero del commit2 a partir de la línea 34, 8 líneas han sufrido cambios (esto sería el caso en el que, en un mismo fichero, se han borrado 6 líneas y se han escrito 8)

Por último, tenemos el contenido que ha variado:

```default
-this is a git diff test example
+this is a git example
```

La línea -1 tenía el contenido "this is a git diff test example" y la línea +1 tiene ahora el contenido "+this is a git example" recordemos que git asume que commit2 es el más reciente, entonces la simbología del output alude a que se ha eliminado una línea del commit1 y se ha incorporado otra en el commit2.

Veámos otro ejemplo. En este caso, encontramos que se ha incorporado un fichero nuevo al repositorio y el resto de elementos se han mantenido intactos:

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221117103731.png' | relative_url }}" text-align="center"/>
</div>

Lo primero en lo que debemos fijarnos es que git diff encuentra diferencias en un fichero y además nos dice que se trata de un fichero nuevo, que antes no estaba. Por ello, podemos observar que la simbología atribuida al fichero del commit1 corresponde a /dev/null porque no existe. Además se nos informa de que en el primer fichero desde la línea 0 se han modificado 0 líneas y en el segundo fichero se ha añadido una que es "Holadios".

<br />

**Viewing the Commit History**

Si has clonado un repositorio o has estado trabajando en uno generando varios commits puedes ver el historia de commits para ver qué ha ocurrido con el comando 'git log'.

Por defecto, 'git log' extrae los commits en orden inverso; del último al primero, junto con el SHA-1 checksum, autor y la fecha:

```bash
git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date: Mon Mar 17 21:52:11 2008 -0700

Change version number

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date: Sat Mar 15 16:40:33 2008 -0700

Remove unnecessary test

commit a11bef06a3f659402fe7563abf99ad00de2209e6
Author: Scott Chacon <schacon@gee-mail.com>
Date: Sat Mar 15 10:31:28 2008 -0700

Initial commit
```

Tenemos un flag importante '-p' que además muestra las diferencias entre los commits:

```bash
git log -p -2
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date: Mon Mar 17 21:52:11 2008 -0700

Change version number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
--- a/Rakefile
+++ b/Rakefile
@@ -5,7 +5,7 @@ require 'rake/gempackagetask'
spec = Gem::Specification.new do |s|
s.platform = Gem::Platform::RUBY
s.name = "simplegit"
- s.version = "0.1.0"
+ s.version = "0.1.1"
s.author = "Scott Chacon"
s.email = "schacon@gee-mail.com"
s.summary = "A simple gem for using Git in Ruby code."

commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7
Author: Scott Chacon <schacon@gee-mail.com>
Date: Sat Mar 15 16:40:33 2008 -0700

Remove unnecessary test

diff --git a/lib/simplegit.rb b/lib/simplegit.rb
index a0a60ae..47c6340 100644
--- a/lib/simplegit.rb
+++ b/lib/simplegit.rb
@@ -18,8 +18,3 @@ class SimpleGit
end

end
-
-if $0 == __FILE__
- git = SimpleGit.new
- puts git.show
-end
```

<br />


**Git Branches**

El funcionamiento de Git se asienta sobre los repositorios que son colecciones ordenadas de proyectos (código). De forma que los individuos que desean tener en local una copia en su dispositivo para trabajar realizan una clonación del repositorio (git clone) online en su máquina local.

Conjuntamente, existe el Git Branching. El Branching consiste en la separación de la línea de desarrollo de un proyecto en diferentes líneas que esencialmente son variantes del mismo proyecto que no convergen entre sí:

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221116101534.png' | relative_url }}" text-align="center"/>
</div>

Podemos mostrar las ramas de un proyecto con el comando 'git branch' o 'git show-branch':

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221117165133.png' | relative_url }}" text-align="center"/>
</div>

Para intercambiar entre ramas utilizamos el comando 'git checkout':

```bash
git checkout dev
```

<br />

<div style="text-align:center">
	<img src="{{ 'assets/img/gitimg/Pasted image 20221117165303.png' | relative_url }}" text-align="center"/>
</div>

