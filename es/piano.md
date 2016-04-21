[//]: # (-*- mode: markdown; coding: utf-8 -*-)
# Piano de juguete

<figure style="float:right;padding:10px">
  <img src="img/piano.png" width="350"/>
  <figcaption style="font-size:smaller;font-style:italic;text-align:center">
	Piano infantil como el que vamos a modificar.
  </figcaption>
</figure>

Vamos a analizar como caso de estudio un piano de juguete.  Supongamos
que tenemos un muchachito travieso que se ha subido en su piano de
juguete y ha estado saltando encima de él hasta que ha dejado de
sonar.  Estas cosas pasan, os lo aseguro, ya no hacen los pianos como
antes.  Supongamos que el muchachito está muy arrepentido y te pide
con su mejor cara de angelito que *le pongas pilas al piano*.  No vale
la pena explicarle que el problema es mayor, hay que hacer algo.
Armados con un destornillador y un poco de paciencia conseguimos
desmontarlo entero e identificamos el teclado de membrana que hay bajo
las teclas del piano.

<figure style="float:right;padding:10px">
  <img src="img/teclas.jpg" width="350"/>
  <figcaption style="font-size:smaller;font-style:italic;text-align:center">
	Teclado de membrana bajo las teclas del piano.
  </figcaption>
</figure>

Las teclas están físicamente dispuestas como una línea pero
lógicamente dispuestas de forma matricial, como muestra la figura
adjunta.  Hay ocho columnas y tres filas.  Cuando se pulsa una de las
teclas se cortocircuita la columna correspondiente con la fila
correspondiente.  Es así de simple.

<figure style="float:right;padding:10px">
  <img src="img/matrix.svg" width="350"/>
  <figcaption style="font-size:smaller;font-style:italic;text-align:center">
	Disposición matricial del teclado de membrana.
  </figcaption>
</figure>

Lo que vamos a hacer es eliminar el controlador actual del piano y
sustituirlo por una Raspberry Pi.  Lo ideal sería poner una Raspberry
Pi Zero, pero eso es un detalle menor, puesto que el diseño y el
software es exactamente igual con cualquier otro modelo actual.  Para
los altavoces podemos usar cualquier mecanismo utilizado en un
ordenador convencional.  A mi me gusta especialmente la idea de un
altavoz Bluetooth, que puede ser externo si queremos.  Se puede
conseguir uno resistente al agua por poco más de seis euros en
[Banggood.com](http://www.banggood.com/Mini-Waterproof-Wireless-Bluetooth-Speaker-For-iPad-iPhone-6-6-p-88071.html).

Bueno, ya tenemos el primer concepto, solo nos falta el software para
tener el prototipo. Hay dos cosas que debemos hacer:

* Detectar las pulsaciones de tecla evitando los rebotes típicos de
los teclados de membrana.

* Tocar notas correspondientes a cada tecla desde el momento que se
  aprieta hasta el momento que se libera.

La biblioteca *Reactor* tal y como la hemos explicado hasta ahora
implementa un tipo de entradas `input_handler` que vale perfectamente
para botones independientes.  Sin embargo con la disposición matricial
no es inmediato que funcione sin modificar nada.  Vamos a explicar
cómo enfrentaríamos este problema suponiendo que la biblioteca no lo
contempla.  Está claro que tenemos que hacer cambios en
`input_handler`, pero ¿cómo?

El otro problema, tocar notas, no es tan simple como parece a primera
vista.  No es cuestión de tocar un *mp3* correspondiente a cada nota y
listos. Podemos apretar varias notas a la vez y deberían sonar todas
simultáneamente.  Eso exige un proceso de síntesis de sonidos y
mezclado digital.

Todo se puede hacer, pero en este taller seguimos la filosofía KISS
(*Keep It Simple, Stupid*).  No vamos a trabajar en algo que ya está
resuelto, bastante trabajo tenemos con lo no resuelto.  Raspbian tiene
preinstalado [*Sonic-Pi*](http://sonic-pi.net), una fantástica
herramienta docente del propio Laboratorio de Computadores de la
Universidad de Cambridge, el mismo que hizo Raspberry Pi.  Nos basta
un simple
[tutorial](https://www.raspberrypi.org/learning/getting-started-with-sonic-pi/)
para ver que hace todo lo que necesitamos y muchísimo más.  Mejor, en
el futuro le añadiremos funcionalidad.

Solo falta ver cómo podemos usar *Sonic-Pi* desde nuestro programa.
Si hacemos una búsqueda rápida para encontrar una *command line
interface to sonic-pi* seguro que encontramos
[sonic-pi-cli](https://github.com/Widdershin/sonic-pi-cli).  Puede
valer, pero si miramos un poquito el código fuente veremos que en
realidad utiliza una conexión TCP al puerto 4557. Mmmm, osea que
Sonic-Pi ya tiene un protocolo para ser controlado de forma remota.
Fantástica idea, vamos a usar el `connector` de la biblioteca
*Reactor* para ésto.  Si exploramos un poco más el código de Sonic-Pi
parece que internamente utiliza otro programa que hace la síntesis de
sonido,
[*SuperCollider audio synthesis server*](http://supercollider.github.io/),
que resulta que tiene un ejecutable independiente llamado `scsynth`
con soporte de conexiones remotas UDP o TCP e incluso un cliente de
línea de órdenes llamado `sclang`.

Todos los elementos están claros, solo falta sentarse a diseñar el
pegamento:

* El servidor `scsynth` debe arrancarse previamente.  Podemos echar
  una ojeada al código de Sonic-Pi para ver qué opciones serían
  razonables.

* Nuestro programa debe explorar el teclado para detectar pulsaciones
  y levantamientos de tecla.

* Cuando se presiona una tecla debe comunicarse con `scsynth` para
  añadir a lo que ya se está tocando una nota más.

* Cuando se libera una tecla debe comunicarse con `scsynth` para
  quitar de lo que se esté tocando la nota correspondiente a esta
  tecla.

## Exploración del teclado

<figure style="float:right;padding:10px">
  <img src="img/button.svg" width="350"/>
  <figcaption style="font-size:smaller;font-style:italic;text-align:center">
	Esquema eléctrico de cada tecla.
  </figcaption>
</figure>


## Control remoto de SuperCollider

Es el momento de aprender algo sobre *SuperCollider* y de su control
remoto.  El control remoto utiliza una versión simplificada del
protocolo
[*Open Sound Control*](http://cnmat.berkeley.edu/user/adrian_freed/blog/2008/10/06/open_sound_control_1_1_specification).
Describiremos la versión TCP, que es la que vamos a usar.

* Todos los datos se codifican en *big-endian* (*network byte order*).

* Cada mensaje va precedido por un entero de 32 bits que contiene la
  longitud del mensaje. A continuación aparece una orden (*command*) o
  un paquete de órdenes (*bundle*).

* Cada orden (*command*) consiste en una cadena que representa la
  orden, seguida de una coma y una lista de letras que representan los
  tipos de los argumentos.  A continuación vendrían los valores de los
  argumentos en *big-endian*.  Las cadenas codifican el `\0`
  terminador y además requieren relleno (*padding*) hasta el siguiente
  múltiplo de cuatro bytes.

* Cada paquete de órdenes (*bundle*) empieza con la cadena `#bundle`
  (incluido el `'\0'` terminador de las cadenas C). A continuación una
  marca temporal de 64 bits y a continuación todas las órdenes
  incluidas en el paquete codificadas igual que en un mensaje normal
  (con la longitud precediendo a cada orden).  Los paquetes de órdenes
  no van a ser necesarios.

La lista completa de las órdenes soportadas está disponible en
[doc.sccode.org](http://doc.sccode.org/Reference/Server-Command-Reference.html).
Algunas órdenes reciben respuestas usando el mismo protocolo.

Básicamente el procedimiento puede extraerse de
[esta página](http://doc.sccode.org/Guides/ClientVsServer.html).  En
el momento en que se pulsa una tecla enviaremos una orden `/s_new` y
en el momento en que se libere la tecla enviaremos una orden
`/n_free`.  Previamente hay que configurar los `SynthDef` ya
compilados en una carpeta accesible por `scsynth`.

> **Warning** En el momento que se escriben estas páginas `sclang`,
> que es el cliente de `scsynth`, no funciona correctamente en
> Raspbian por un error oculto que solo se manifiesta con la nueva
> versión de GCC.  Solo lo necesitamos para compilar los `SynthDef`,
> así que puede usarse temporalmente una versión independiente, como
> [la de *redFrik* en GitHub](https://github.com/redFrik/supercolliderStandaloneRPI2). Así
> nos ahorramos recompilar el paquete.

Antes de nada hemos hecho pruebas para asegurarnos del funcionamiento
de los mensajes.  Primero hemos probado nuestra hipótesis activando de
forma manual dos notas a la vez con `sclang` o con `scide`.

```
s.boot;
SynthDef("seno", { arg freq=600; Out.ar(0, SinOsc.ar(freq)); }).load(s);
s.sendMsg("/s_new", "seno", x = s.nextNodeID, 1, 1);
s.sendMsg("/s_new", "seno", y = s.nextNodeID, 1, 1, "freq", 900);
...
s.sendMsg("/n_free", x);
s.sendMsg("/n_free", y);
```

Probando de forma interactiva con `scide` (*Ctrl-Intro* ejecuta solo
la línea del cursor) podemos ver que efectivamente el mensaje `/s_new`
activa una nota y `/n_free` lo desactiva.  El uso de un único
*SynthDef* con un argumento facilita considerablemente su uso.
Podríamos tener un *SynthDef* más elaborado por cada instrumento.
Pero vayamos poco a poco.

Para asegurarnos del formato del mensaje vamos a capturar los mensajes
usando *netcat*.  Por un lado en un terminal lo ejecutamos en modo
servidor UDP:

```
pi@raspberrypi:~ $ nc -l -u 9999 | tee mensajes.txt
```

Por otro lado ejecutamos el siguiente código con `sclang`:

```
r = Server(\myServer, NetAddr("localhost", 9999));
SynthDef("seno", { arg freq=600; Out.ar(0, SinOsc.ar(freq)); }).load(r);
r.sendMsg("/s_new", "seno", x = s.nextNodeID, 1, 1);
r.sendMsg("/s_new", "seno", y = s.nextNodeID, 1, 1, "freq", 1024);
r.sendMsg("/n_free", x);
r.sendMsg("/n_free", y);
```

Esta vez no arrancamos una nueva instancia de `scsynth` sino que
asumimos que hay uno en el puerto 9999 de *localhost*.  En realidad
está *netcat* capturando todo lo que se genera.  Este es el resultado
poniendo cada mensaje en una línea independiente:

```
/d_load\0,si\0/home/pi/sc/share/user/synthdefs/seno.scsyndef\0\0\0\0\0
/s_new\0\0,siii\0\0\0seno\0\0\0\0\0\0\0\6\0\0\0\1\0\0\0\1
/s_new\0\0,siiisi\0seno\0\0\0\0\0\0\0\7\0\0\0\1\0\0\0\1freq\0\0\0\0\0\0\4\0
/n_free\0,i\0\0\0\0\0\6
/n_free\0,i\0\0\0\0\0\7
```

Nosotros vamos a utilizar TCP por lo que además tendremos que preceder
cada mensaje con la longitud en formato *big-endian*.

Ya tenemos claro los elementos.  Hay que ejecutar `scsynth` y usar un
`connector` para enviar mensajes.  Cuando nuestro programa termine
`scsynth` también debe hacerlo.  Una forma sencilla de conseguirlo es
con un `process_handler`, aunque no necesitemos reaccionar ante lo que
imprima el programa.

Osea para este ejemplo necesitamos un `process_handler` y un
`connector`, ¿cómo lo agrupamos?  ¿Definimos otro manejador de
*Reactor*?  La respuesta está en cómo queremos lidiar con la entrada.
*Reactor* es un patrón para definir comportamientos reactivos, si no
hay que reaccionar no es necesario usarlo.  Los manejadores no
obstante podemos usarlos por mera conveniencia.

¿Y si llegan datos del otro extremo?  En el caso del proceso de
`scsynth` no hay nada que podamos hacer con la salida estándar, así
que redirigiremos toda la salida a `/dev/null`.  Sin embargo el
control remoto de `scsynth` puede generar respuestas y notificaciones
(ver la
[guía de referencia de órdenes](http://doc.sccode.org/Reference/Server-Command-Reference.html)).
Por ejemplo, el mensaje `/d_load` es asíncrono y no termina hasta que
se recibe la respuesta `/done`.

Por tanto ¿qué es nuestro sintetizador? Se trata de un `connector`
especializado.  Un `connector` que automáticamente arranca otro
proceso aprovechando un `process_handler` para ello.  Un conector que
al destruirse envía el mensaje `/quit` a `sc_synth` luego destruye el
`process_handler` correspondiente.  Un `connector` con métodos
especializados para enviar y recibir mensajes OSC.  Ahora ya podemos
sentarnos a escribir código.

> **Info** Es importante sentarse un poco a valorar el proceso de
> diseño que hemos seguido.  Nuestro problema era la síntesis de
> sonido y ha terminado siendo las comunicaciones con un servidor.
> Esto es muy frecuente en la vida real,**lo que vemos como problema no
> siempre es el problema si aplicamos ciertas dosis de imaginación y
> *pensamiento lateral***.

## Juntando todo
