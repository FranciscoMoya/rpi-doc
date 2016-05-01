[//]: # (-*- mode: markdown; coding: utf-8 -*-)
# Casos de estudio

Vamos a explicar someramente algunos de los ejemplos incluídos en el
software que acompaña al taller.  Se trata de ejemplos ligeramente más
complejos que los ejemplos triviales, de forma que pueda apreciarse la
ventaja de una arquitectura software sólida.

Estos casos de estudio tienen dos propósitos:

* Que el alumno sea consciente de la complejidad real de los sistemas
  basados en Raspberry Pi.  Los ejemplos más evolucionados apenas
  ocupan unos cientos de líneas de código con módulos de muy bajo
  acoplamiento y funciones que no superan las 20 líneas.

* Que le sirvan al alumno como fuente de inspiración para que sus
  proyectos personales sean más creativos y evolucionados.

Antes de nada vamos a ver un mini-caso de estudio en el que
realizaremos una de las operaciones más frecuentes cuando se diseña
una nueva aplicación, la creación de un nuevo manejador.  Esto se va a
repetir multitud de veces en nuestros casos de estudio y en la vida
real.  Es la base de la programación de sistemas: la abstracción.
Abstraer es eliminar detalle.  Abstraemos para simplificar tanto el
uso como la programación de los módulos.

* Cada manejador trata un problema pequeño y nada mas.  Por tanto su
  programación es relativamente sencilla y tiene poca relación con el
  resto del programa.

* Cuando se usa el manejador no es preciso conocer los detalles de
  cómo se implementa.  Solo lo estrictamente necesario.

## Sensor analógico

Hemos visto cómo puede medirse una magnitud analógica utilizando un
montaje con la pata MISO de la interfaz SPI y una entrada digital.  Es
hora de convertir esa técnica en un nuevo manejador de eventos para la
automatización.

El reto es definir el manejador `analog_handler` para detectar cuando
se pasan ciertos umbrales previamente configurados.  Debe cumplir los
siguientes requisitos:

* Debe poder seleccionarse la pata digital, los umbrales de activación
  y desactivación, el periodo de muestreo y la frecuencia del reloj.
  
* Debe medir periódicamente usando el método descrito en el capítulo
  sobre SPI.
  
* Debe notificar cuando se supera el umbral de activación y cuando el
  valor queda por debajo del umbral de desactivación.
  
* Una vez notificada una activación o una desactivación ya no vuelve a
  notificar nada hasta que cambie el estado (pasa de activado a
  desactivado o a la inversa).

Bien, el problema está claro, vamos a resolverlo paso a paso.

### Empezando por el final

No me cansaré de repetir esto: *si quieres que funcione las pruebas
primero*.  Antes de empezar a hacer nada del nuevo manejador tenemos
que hacer un caso de prueba típico.  Esto nos ayuda a entender
completamente la interfaz de programación que se pretende crear y nos
permitirá en el futuro tener cierta confianza en el correcto
funcionamiento del manejador.

Es el primer ejemplo de *Reactor*, así que vamos a ayudar un poco. En
la carpeta `src/c/ejemplos/analog` tienes un esqueleto de lo que
queremos construir.  Echa un vistazo a `test_analog_handler.c`.  Como
ves sigue el convenio de todas las pruebas de la biblioteca *Reactor*,
un único archivo que comienza por `test_` junto al nombre del módulo
que queremos probar.  Procura respetar este convenio, hace que
cualquier usuario de tu código encuentre las cosas más fácilmente.

```C
#include "analog_handler.h"
#include <reactor/reactor.h>
#include <wiringPi.h>
#include <stdio.h>

static void bajo(analog_handler* this) {
    puts("Bajo limite inferior");
}
		 
static void alto(analog_handler* this) {
    puts("Sobre limite superior");
}

int main(int argc, char* argv[])
{
    wiringPiSetupGpio();
    reactor* r = reactor_new();
    analog_handler* in = analog_handler_new(25, 100, 200,
					    bajo, alto);
    reactor_add(r, (event_handler*)in);
    reactor_run(r);
    reactor_destroy(r);
    return 0;
}
```

Construimos un `analog_handler` con funciones que simplemente imprimen
un mensaje cada vez que detecta el paso de umbral.  Solo queda
compilarlo con un `makefile`.  Como verás en la carpeta
`src/c/ejemplos/analog` ya tienes uno:

```
CFLAGS=-pthread -std=c11 -Wall -D_POSIX_C_SOURCE=200809L -I../../reactor -ggdb \
	-fasynchronous-unwind-tables 
LDFLAGS=-L../../reactor/reactor -pthread -rdynamic
LDLIBS=-lreactor -lwiringPi -lpthread
CC=gcc

all: test_analog_handler

test_analog_handler: test_analog_handler.o analog_handler.o

clean:
	$(RM) *~ *.o test_analog_handler
```

## Esqueleto del manejador

Es ahora y no antes cuando tenemos que plantearnos escribir los
archivos `analog_handler.h` y `analog_handler.c`.  Se parte de
cualquiera de los que ya tienes en *Reactor* y se modifican de acuerdo
a las necesidades.  Lo primero es decidir qué tipo de `event_handler`
**es** y para ello hay que fijarse en cómo va a notificar los eventos
al programa.  Como ya sabemos *Reactor* utiliza descriptores de
archivo para recibir eventos, por lo que las notificaciones tienen que
llegar por este medio.  En `analog_handler` no hay descriptores de
archivo, solo hay el pin MISO y una salida GPIO.  Hay que realizar
periódicamente un proceso ciertamente tedioso, en el que hay que
contar bits.  Todo esto nos da pistas.

Por un lado tenemos que sintetizar eventos en un descriptor de
archivo, porque el problema como tal no tiene descriptores.  Eso
implica usar un `pipe_handler`.  Por otro lado hay que realizar un
proceso periódico al margen de la tarea principal.  Eso nos lleva a un
`thread_handler` o un `process_handler`.  Dado que no hay que ejecutar
nada externo nos decantaremos por un `thread_handler`, que es más
eficiente.  En *Reactor* tenemos un ejemplo de manejador similar, el
de las entradas digitales, solo hay que copiarlo, sustituir
`input_handler` por `analog_handler` y quitar todo el código
específico.

Empezamos por la cabecera que quedaría así:

``` C
#ifndef ANALOG_HANDLER_H
#define ANALOG_HANDLER_H
#include <reactor/thread_handler.h>

typedef void (*analog_handler_function)(analog_handler* this);

typedef struct analog_handler_ analog_handler;
struct analog_handler_ {
	thread_handler parent;
	event_handler_function destroy_parent_members;

	// FIXME: Añade todos los atributos que necesites
};

analog_handler* analog_handler_new (int pin, int low, int high,
                                    analog_handler_function low_handler,
	                                analog_handler_function high_handler);
void analog_handler_init (analog_handler* this,
	                      int pin, int low, int high,
	                      analog_handler_function low_handler,
	                      analog_handler_function high_handler);
void analog_handler_destroy (analog_handler* ev);

#endif
```

Esto es lo mínimo: el constructor con y sin reserva de memoria
dinámica y el destructor.  El nuevo tipo `analog_handler` declara que
es una especialización de `thread_handler` porque su primer atributo
(*parent*) es un `thread_handler`.  Hemos dejado el atributo
`destroy_parent_members` porque asumimos que alguna operación de
destrucción puede ser necesaria.  Ya veremos.

A continuación hacemos lo mismo con `analog_handler.c`:

``` C
#include "analog_handler.h"

// FIXME: declaraciones de funciones estáticas específicas

analog_handler* analog_handler_new (int pin, int low, int high,
	                                analog_handler_function low_handler,
				                    analog_handler_function high_handler)
{
    analog_handler* h = malloc(sizeof(analog_handler));
    analog_handler_init_members(h, pin, low, high, low_handler, high_handler);
    event_handler* ev = (event_handler*) h;
    ev->destroy_self = (event_handler_function) free;
    thread_handler_start (&h->parent, analog_handler_thread);
    return h;
}


void analog_handler_init (analog_handler* this,
	                      int pin, int low, int high,
	                      analog_handler_function low_handler,
	                      analog_handler_function high_handler)
{
    analog_handler_init_members(this, pin, low, high, low_handler, high_handler);
    thread_handler_start (&this->parent, analog_handler_thread);
}


void analog_handler_destroy (analog_handler* this)
{
    event_handler_destroy((event_handler*)this);
}


// FIXME: Definiciones de funciones estáticas específicas
```


Un `thread_handler` es algo relativamente complejo, porque el orden en
que se realizan las operaciones puede afectar al correcto
funcionamiento en determinados casos.  Procura respetar esta
estructura tanto como sea posible para evitarte disgustos.

Los dos constructores (con y sin reserva de memoria) tienen que
retrasar la ejecución del hilo hasta el último momento.  Por eso
extraemos la inicialización de los atributos a otra función diferente
(`analog_handler_init_members`).

Hasta este punto lo tienes hecho en la carpeta
`src/c/ejemplos/analog`.  Sin embargo a partir de aquí te toca
trabajar.  ¡Manos a la obra!


## Funciones específicas

Ya solo queda declarar e implementar las funciones específicas de este
manejador.  Primero declara las funciones que hemos utilizado en el
esqueleto: `analog_handler_thread` y `analog_handler_init_members`.
Si tienes dudas consulta `input_handler.c` en la biblioteca *Reactor*.
Dado que son funciones que no se exportan al usuario decláralas como
`static`.

A continuacion haz un esqueleto de ambas funciones justo debajo.
Déjalas vacías de momento, solo queremos compilar para detectar
problemas lo antes posible.  Una vez definidas ejecuta `make`.  Verás
errores, seguro.  Corrige los errores tipográficos, añade las
directivas *include* necesarias y consigue que compile sin errores ni
advertencias de ningún tipo.  Usa `man` si no sabes en qué cabecera
está declarada una función del sistema.

Una vez que el programa compila rellena las funciones vacías.  ¿Que
debe hacer `analog_handler_init_members`?  Lo dice la propia función,
inicializar los miembros.  Añade a la declaración de la estructura en
`analog_handler.h` los miembros que consideres necesarios e inicializa
sus valores con los argumentos en `analog_handler_init_members`.  No
te olvides de configurar adecuadamente el pin MISO y el pin GPIO.

A continuación solo queda el hilo de exploración.  El código ya lo
tienes en el [capítulo de SPI](spi.html) pero tienes que asegurarte de
repetir periódicamente la medida.  Usa tantas funciones como sea
necesario, no hagas funciones muy largas.  Si ves que tienes que usar
un `for` y dentro un `if` y dentro otro `if` eso es ya señal de que
tienes que dividir en funciones más pequeñas.

Mi sugerencia es copiar de `input_handler.c` tanto como sea posible.
Por ejemplo, el hilo puede ser casi igual pero quitando todo lo que
tiene que ver con el periodo de muestreo.  No hace falta esperar entre
muestra y muestra porque el método que vimos en el capítulo de SPI ya
necesita esperar a la descarga completa del condensador.

``` C
static void* analog_handler_thread(thread_handler* h)
{
    analog_handler* in = (analog_handler*) h;
    while(!h->cancel)
	    analog_handler_poll(in);
    return NULL;
}
```

Esto es evidentemente incompleto.  Hay que definir
`analog_handler_poll`.  Pero su contenido es directamente lo que vimos
en el [capítulo de SPI](spi.html).

## Generalizando

Una vez que el programa de prueba esté funcionando llega el momento de
hacer algo de autocrítica.  ¿Es posible mejorar la interfaz de
programación? ¿Es posible con poco trabajo hacer que sea más útil? Por
ejemplo:

* Los constructores tienen demasiados argumentos.  Es engorroso y
  propenso a error.  Una posible alternativa es pasar una estructura
  con los datos de configuración.

* El tiempo necesario para estar seguros de que el condensador se ha
  descargado depende del valor del condensador, habría que ponerlo en
  un parámetro.

* El reloj SPI a utilizar depende de la magnitud a medir, del tamaño
  máximo de los buffers pasados a `wiringPiSPIDataRW` y de la
  precisión necesaria.  Habría que dejar también como parámetro de
  configuración tanto la frecuencia como el tamaño de los buffers.

* Si el condensador no ha tenido tiempo de descargarse es posible que
  no haya ningún cero en el buffer.  Esa condición debe notificarse
  como un error (elevar una excepción).  Lo mismo ocurre si el
  condensador no ha tenido tiempo de cargarse hasta leer algún uno.
  Si el último byte leido no es distinto de cero debe notificarse como
  error (elevar excepción).

Valora tú cuáles de estas críticas deberían arreglarse y propón una
solución.

## Recapitulando

¿Lo conseguiste? No desistas al primer intento.  Solo hay una forma de
convertir esto en una rutina, repitiéndolo muchas veces.

De todas formas si te cansas tenemos el ejemplo resuelto.  No es buena
política leer directamente la solución pero si es tu deseo aquí tienes
cómo hacerlo.  La solución está en una rama del repositorio que se
llama `analog`.  Podemos sacarla directamente así:

```
pi@raspberrypi:~/src/c/ejemplos/analog $ git checkout analog -- analog_handler.c
pi@raspberrypi:~/src/c/ejemplos/analog $ git checkout analog -- analog_handler.h
pi@raspberrypi:~/src/c/ejemplos/analog $ ▂
```

Una vez generalizado el manejador es posible que resulte útil para un
uso general.  Es el momento de plantearse añadirlo a la propia
biblioteca *Reactor*.

