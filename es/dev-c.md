[//]: # (-*- markdown; coding: utf-8 -*-)

# Desarrollo en C

Por primera vez el taller se realizará en versión multi-lenguaje (C y
Python).  Esperamos de esta forma acomodarnos a un número mayor de
alumnos.  En la actualidad usamos Python como primer lenguaje de la
titulación (primer semestre de primer curso, asignatura *Informática*)
pero hasta este curso hemos utilizado C.  Por tanto hay un grupo de
estudiantes que se sienten más cómodos con C y otro que se siente más
cómodo con Python.

Este manual trata de dividir el contenido de forma que no tengas que
leer todo si lo que te interesa es uno solo de estos lenguajes.  En
este capítulo describiremos el conjunto de herramientas que vamos a
usar para hacer programas en C y un ejemplo sencillo de su uso.

## Herramientas

Para la realización de estas prácticas utilizaremos un
conjunto muy reducido de herramientas: un editor de textos, la
consola, el compilador `gcc`, la herramienta `make`, y el
depurador `gdb`.

La Raspberry Pi cuenta con un editor de textos simple, `nano` que
puede ser lanzado desde la consola del sistema.  Aunque no
vengan instalados con la distribución, posteriormente se podrán
instalar otros como vi, vim, o joe.  En otro orden de magnitud, el
editor `emacs` está orientado a la edición de código, pero su uso
requiere un estudio más pormenorizado que el resto de herramientas.
Se deja a decisión del alumno la herramienta de edición que más se
ajuste a sus necesidades.

Este documento no detallará el uso del depurador, ya que éste se
estudiará y documentará en mayor profundidad en la sección
correspondiente.  Por lo tanto, en este documento presentaremos los
detalles esenciales del compilador `gcc` y de la herramienta
`make`.  Al final del documento se proporciona una lista de las
referencias donde se podrá obtener información más profunda y
detallada de los mismos.  Aquí, sólo se pretende apuntar de manera
sucinta lo esencial para que el alumno pueda practicar con los
ejemplos de código que se le proporcionan en las primeras sesiones,
hasta que se familiarice con la plataforma utilizada en los
laboratorios.

### Compilador: GCC

Las siglas GCC provienen de *GNU Compiler
  Collection*, y como su propio nombre indica, se trata de un conjunto
de compiladores para diversos lenguajes de programación: C, C++,
Objective C, Chill, Fortran, y Java, distribuido bajo licencia GPL.

Se trata de una herramienta muy completa y compleja, y no es objeto de
este documento presentarla en profundidad.  Las opciones más comunes
son:


**Opción**&nbsp;&nbsp;&nbsp;&nbsp; | **Significado**
-----------|----------------
`-c` | Compila o ensambla, pero no enlaza, obteniéndose un archivo en código objeto con extensión `.o`
`-E` |  Realiza únicamente el preprocesamiento, enviando el resultado a la salida estándar
`-g` | Incluye información estándar para el depurado
`-Iruta` | Especifica la *ruta* a la carpeta donde encontrar los archivos de cabecera
`-Lruta` | Especifica la *ruta* a la carpeta donde encontrar bibliotecas de funciones
`-lXXX` |  Enlaza con la biblioteca de nombre `libXXX.a`
`-o` *nombre* | Indica el nombre del archivo ejecutable
`-S` | Preprocesa y compila, pero no ensambla ni enlaza
`-v` | Muestra las fases por las que va pasando el compilador
`-w` | Suprime los mensajes de aviso (*warnings*)
`-Wall` | Emite todos los avisos que el compilador pueda generar


### La herramienta de construcción `make`

`Make` es una herramienta que permite automatizar la
construcción de archivos a partir de otros archivos.  En particular es
especialmente útil para automatizar la compilación del código C para
generar archivos objeto y el montaje de los archivos objeto para
generar un ejecutable.

El archivo `Makefile` debe contener el conjunto de dependencias
y reglas que construcción que determinan cómo generar el código.
Veamos un ejemplo sencillo para compilar un programa `ejecutable`
a partir de dos archivos fuente `archivo1.c` y
`archivo2.c`, y un archivo de cabecera `archivo1.h`
incluido por ambos:

```
ejecutable: archivo1.o archivo2.o
        gcc -o ejecutable archivo1.o archivo2.o

archivo1.o: archivo1.c archivo1.h
        gcc -c archivo1.c

archivo2.o: archivo2.c archivo1.h
        gcc -c archivo2.c
```

Con este `Makefile` estamos indicando a `make` que para
construir `ejecutable` necesita primero disponer de los archivos
`archivo1.o` y `archivo2.o` (los archivos objeto).  Y una
vez que disponga de ellos puede construirlo empleando la orden que
aparece en la segunda línea (uso de `gcc` como montador o linker).

En la línea 4 se indica que para construir `archivo1.o` se
precisa primero de los archivos `archivo1.c` y
`archivo1.h`.  Y una vez que se disponga de ellos puede
construirlo con la orden de la línea 5 (uso de `gcc` como
compilador).  Las líneas 7 y 8 son similares a las líneas 4 y 5 pero
indican como construir `archivo2.o`.

Un aspecto interesante del uso de `make` es que solo se genera lo
necesario, y se aplican las reglas afectadas por archivos que han
cambiado.  Es decir, si se dispone de un buen `Makefile` basta
con ejecutar `make` para que se compile todo lo necesario y solo
lo necesario en cualquier momento.

Cae fuera del ámbito de este curso explicar con más detalle la
sintaxis que utiliza el archivo `Makefile`.  En su lugar,
utilizaremos `make` con archivos `Makefile` proporcionados
en plantillas de aplicaciones.

### GNU Debugger

Se suele asociar el uso de herramientas de depuración a la
resolución de aquellos problemas o fallos que aparecen en tiempo de
ejecución, causantes de que programas aparentemente bien escritos
terminen su ejecución inesperadamente con un error del tipo
*segmentation fault*.

Las herramientas de depuración permiten detener la ejecución del
programa en puntos específicos, por ejemplo, justo antes de la
ocurrencia de un error en el que se sabe que la ejecución termina
inesperadamente.  La detención del programa permite explorar en ese
punto la memoria, incluso conocer el valor de las variables y
registros del procesador e ir avanzando por la ejecución hasta
encontrar el punto exacto en el que ocurre la disfunción del programa.
En cualquier caso, el manejo de herramientas de depuración es
fundamental para que el programador pueda llevar a cabo una
programación confiable.

Sin embargo, el uso de los depuradores no se limita únicamente al campo
de la localización y resolución de errores, sino que también se
emplean para explorar la arquitectura sobre la que está ejecutando un
programa.  Con esta finalidad esta sesión expondrá someramente una guía
de uso de una herramienta de depuración.  La siguiente sesión,
utilizará conjuntamente el emulador de la consola y el depurador para
explorar el código, la organización de la memoria y los registros del
procesador.

Existe un amplio abanico de herramientas de depuración.  Este
documento presentará el depurador de GNU, abreviado `gdb`, como un
depurador simbólico avanzado con soporte de infinidad de
arquitecturas, formatos de ejecutable y lenguajes de programación.
Aunque existen numerosos interfaces de usuario para gdb, comenzaremos
su estudio con la interfaz de línea de órdenes, ya que ésta será
común para todos los demás, aunque también se comentarán algunas
herramientas gráficas.

#### Uso de `gdb`

Para que `gdb` pueda ser aprovechado al máximo en la
depuración de un programa, será necesario que éste haya sido compilado
con soporte para depuración, lo cual se especifica al compilador
`gcc` mediante la opción `-g`.

Utilizando interfaz de línea de órdenes, el primer paso consiste en
invocar `gdb` y para ello, la forma más usual de realizar la
invocación es especificando el ejecutable a depurar:

```
$ gdb programa
```

Una vez que se ha cargado el ejecutable, se podrán realizar las
siguientes operaciones:

#### list (l)

Lista el código del programa o de una función.  Se utiliza como:

```
(gdb) l
(gdb) l funcion
(gdb) l 1,40
```

La línea 1 lista las 10 próximas líneas.  La línea 2 lista la función
\src{funcion} y la línea 3, lista de las líneas 1 a 40.

#### break (b)

Coloca un punto de ruptura en la función, línea o fichero
indicado.  Los puntos de ruptura indican al depurador aquellos puntos
del código en los que se debe detener la ejecución del programa.

```
(gdb) b funcion
(gdb) b 30
(gdb) b codigo.c:30
```

La línea 1 pone un punto de ruptura al comienzo de la función
`funcion`.  La línea 2 lo pone en la línea número 30, mientras
que la línea 3 pone un punto de ruptura en el archivo
`codigo.c` en la línea 30.

#### run

Ejecuta el programa, con los argumentos de entrada que se
especifiquen.  Se ejecutará desde el comienzo, deteniéndose en aquellos puntos
en los que se hayan colocado algún punto de ruptura o watchpoint.

```
(gdb) run
(gdb) run arg1 arg2
```

La línea 1 lanza la ejecución del programa sin argumentos de entrada.
La línea 2 pasa los argumentos arg1 y arg2.

#### print (p)

Se utiliza para imprimir el valor de las variables.  Sólo se
pueden visualizar aquellas expresiones que aún formen parte del
contexto actual de ejecución, por lo que no será posible visualizar el
valor de una variable local a una función cuando se ha salido del
contexto de la misma.

```
(gdb) print i
$1 = 3
(gdb) print i+2
$2 = 5
```

#### backtrace (bt)

Muestra la traza de llamadas del programa, es decir, las
funciones que han sido invocadas y desde dónde.  Las funciones
aparecen apiladas, siendo la base la primera función invocada.

#### continue (c)

Cuando la ejecución del programa está detenida, bien por un
breakpoint o porque se haya lanzado una ejecución paso a paso, esta
orden continúa la ejecución.

#### next (n)

Ejecuta la siguiente línea del programa, sin entrar en las
funciones, es decir, las funciones se ejecutan en un sólo paso.

#### step (s)

Como la anterior, esta instrucción ejecuta la siguiente línea del
programa, con la diferencia de que si es una función, accede al código
de la misma.

#### jump (j)

Salta al punto del programa especificado, sin ejecutar el
código intermedio.  Ese punto puede ser una línea del programa o una
función.

```
(gdb) j 50
(gdb) j funcion
```

La línea 1 indica el salto de la ejecución del código a la línea
número 50 del programa.  La línea 2 salta a la función de nombre
\src{funcion}.

#### until

Lleva la ejecución del programa hasta llegar a la siguiente
línea.  Esta orden será útil para ejecutar bucles en un paso.

#### watch

Los watchpoints se utilizan para vigilar el valor de una
variable y cuando ésta cambie detener la ejecución del programa.  Se
puede utilizar de las siguiente maneras.

```
(gdb) watch expr
(gdb) rwatch expr
(gdb) awatch expr
(gdb) info watchpoints
```

La línea 1 para la ejecución cada vez que la expresión \src{expr}
cambia.  La línea 2 detiene la ejecución cuando se lee la expresión
\src{expr}, la línea 3 cuando \src{expr} es leída o escrita.  La
última línea muestra un listado de todos los watchpoint establecidos.

#### info (i)

Muestra información sobre los breakpoints o watchpoints
definidos.

```
(gdb) info breakpoints
(gdb) info b
(gdb) info watchpoints
```

#### clear

Elimina breakpoints o watchpoints.

```
(gdb) clear 10
```

Elimina el breakpoint o watchpoint establecido en la línea
10.

#### set

Set establece el contenido de una variable al valor dado.

```
(gdb) set variable=1
```

#### return

Termina la función en el punto en el que se encuentra el
depurador y devuelve el valor que se especifica.

```
(gdb) return 3
```

#### whatis

Muestra el tipo de la variable.

```
(gdb) whatis i
type = int
(gdb)
```

#### quit

Con esta orden se sale del depurador.

```
(gdb) quit
$
```

### Front-ends para `gdb`

Un frontend no es más que una interfaz gráfica entre la
aplicación y los usuarios.  Por lo tanto, los front-ends para gdb
realmente no son depuradores en sí mismos, sino sólo interfaces
gráficas que redirigen las peticiones en forma de órdenes al
`gdb`.

Dentro del amplio abanico de front-ends que existe para `gdb` aquí
sólo presentamos las propuestas más relevantes de entre aquéllos que
da soporte a la depuración remota, esencial cuando se desarrolla para
plataformas mediante compilación cruzada, como es el caso de la
compilación y depuración de código para la Raspberry Pi. Excluiremos
Eclipse, que será cubierto con más detalle en sesiones posteriores.

#### `KDbg`

`KDbg` es un front-end desarrollado utilizando la arquitectura
de componentes KDE.  La invocación de `kdbg` sobre la consola del
sistema se realiza de la siguiente manera:

```
$ apt-get install kdbg
$ kdgb ejecutable
```

La figura \ref{fig:kdbg} presenta la ventana principal de la
aplicación.  El ejecutable a depurar también se puede indicar después
de la invocación de la aplicación, en el menu `File |
  Executable`.

La **ventana de código** muestra el código fuente del ejecutable, sobre
el que se podrán insertar directamente los breakpoints, haciendo
doble click sobre las líneas en las que se deberá detener la ejecución
del programa (en el margen izquierdo de la línea aparecerá entonces un
punto rojo indicando la existencia de un breakpoint).  En todo momento
el punto actual de ejecución viene determinado por un puntero verde
que se va desplazando por las sucesivas líneas del código, saltando
incluso a otros archivos, cuando la ejecución del programa pasa al
codigo presente en otro archivo fuente.  También a la izquierda del
código, el símbolo `+' expande el equivalente en ensamblador de la
línea actual.

\imagenhere{img/kdbg-main.png}{13cm}{Pantalla principal de
  KDbg.}{fig:kdbg}

El código fuente se muestra cuando se abre el ejecutable, pero para
abrir cualquier otro archivo de código se accede al menú *File
  $\rightarrow$ Open Source*.


La **ventana de las variables locales** se abre desde el menú
*View $\rightarrow$ Locals*.  El conjunto de variables que se
muestra dependerá del stack frame seleccionado.  Para ver la
**ventana del stack** seleccionar en el menú *View
  $\rightarrow$ Stack*.

Cuando la ejecución se detiene, `KDbg` muestra el contenido de
los registros del procesador en la **ventana de registros**
(*View $\rightarrow$ Registers*).  Los registros se muestran
agrupados según su tipo y en un formato de tres columnas.  La columna
`Register` muestra el nombre del registro.  La columna
`Value` muestra el valor hexadecimal del registro mientras que
la column `Decoded Value` muestra el contenido decodificado del
registro.

La **ventana del volcado de memoria** muestra el contenido de la memoria
de programa (*View $\rightarrow$ Memory*).  Para ver el valor contenido en
una posición concreta basta con introducir la dirección en el campo
editable de la misma ventana.

#### `DDD`

`DDD` son las siglas de *Data Display
  Debugger*.  Lo más característico de este front-end es que permite
explorar los datos mediante interacción con la representación gráfica
de los mismos, pues las estructuras de datos se representan como
grafos.

\imagenhere{img/ddd.png}{13cm}{Pantalla principal de DDD.}{fig:ddd}

Para la invocación desde la consola del sistema:

```
$ apt-get install ddd
$ ddd ejecutable
```

La ventana principal de `DDD` se compone de otras
tres ventanas principales, la ventana de datos, la de código y la
consola de depuración, como se puede ver en la figura~\ref{fig:ddd}.
Además de estas ventanas hay otras más que son opcionales como la de
órdenes, la ventana de código máquina y la de ejecución.

Para lanzar la depuración del ejecutable se selecciona la opción del
menu *Program $\rightarrow$ Run*, seguidamente aparece un cuadro de dialogo
donde se pueden introducir los argumentos de entrada al programa, si
los tuviera.

Este front-end propone varias maneras para examinar las variables:

* La forma más rápida de examinar las variables es acceder al texto
  del código fuente.  Situando el puntero sobre la variable, seguidamente
  aparecerá una ventana con el valor.  Esto es muy útil para un examen
  rápido de variables simples.
* Para estructuras de datos más grandes se puede utilizar la
  consola del depurador para imprimir su valor.
* La ventana de datos permite la representación gráfica de
  estructuras de datos más complejas, muy útil cuando las estructuras
  de datos son dinámicas, ya que la representación se actualiza cada
  vez que se detiene la ejecución del programa.
* Para examinar arrays de valores numéricos éstos se puede
  representar gráficamente como una función en una ventana separada.
* También se podrá acceder al contenido de la memoria y darle el
  formato deseado.



%### Emacs
### Instalación

En esta sección se describen los pasos necesarios para poder
visualizar el ejemplo de ``Hola, mundo'' en la pantalla de la
Raspberry Pi.  Además, se detallará todo lo necesario para poder
desarrollar, compilar y ejecutar una aplicación en la Raspberry Pi.

### Ejemplos de prueba

De nuevo, como ejemplo de prueba eligiremos el ``Hola,
mundo'' que implementamos en C al principio de este documento.
Utiliza un cliente SSH (por ejemplo putty) para conectarte a tu
Raspberry Pi. Edita el archivo `hello.c` con el editor
*nano*:

```
$ mkdir /tmp/hello
$ nano /tmp/hello/hello.c
```

Con estos dos pasos hemos creado el directorio
`/tmp/hello` y posteriormente hemos editado el archivo
`hello.c` dentro de él. Debería quedar más o menos así:

\begin{lstlisting}[language=C]
#include <stdio.h>
void main() {
    puts("Hola, mundo");
}
\end{lstlisting}

Para compilarlo podemos usar directamente la herramienta *make*:

```
$ cd /tmp/hello
$ make hello
```

Con la primera orden nos ubicamos en el directorio donde se
encuentra el programa.  La orden `make` llevará a cabo todo el
proceso de compilación y generará el `hello` que ya podremos
ejecutar.

```
$ ./hello
Hola, mundo
```

La herramienta *make* ha podido construirlo porque solo había un
archivo fuente con el mismo nombre que el ejecutable. Normalmente
necesitará más instrucciones que se le proporcionan en un archivo
llamado `Makefile`.  Por ejemplo, prueba a editar un
*Makefile* para nuestro caso:

```
$ nano Makefile
```

Puede tener este contenido:

\begin{lstlisting}[language=]
CXXFLAGS= -Wall -O3
hello: hello.o
\end{lstlisting}

En este caso le decimos a *make* que por defecto debe hacer
*hello*, que para poder hacerlo necesita hacer primero
*hello.o*. Además hemos definido una variable que contiene
opciones del compilador.  Éstas permiten tener todas las advertencias
posibles (`-Wall`) y optimizar al máximo (`-O3`).

TODO: procesos, redirecciones

depuracion, compilacion
