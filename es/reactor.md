[//]: # (-*- markdown; coding: utf-8 -*-)

# La biblioteca `reactor`

Este capítulo tiene dos objetivos:

* Ofrecer unas pinceladas de los patrones de diseño que consideramos
  más importantes para el desarrollo con Raspberry Pi. Documentar
  brevemente los fundamentos y cómo se aplican.

* Proporcionar plantillas de aplicación totalmente funcionales y
  extensibles para aplicarlas en cualquier proyecto. En un capítulo
  posterior se desarrollarán en diversos casos de estudio, incluyendo
  la interacción a través de la red, la interacción con programas
  externos, etc.


## El patrón *reactor*

Uno de los mayores beneficios de utilizar programación orientada a
objetos es que nos abre la posibilidad de utilizar directamente la
mayor parte del catálogo de *patrones de diseño* disponibles en la
actualidad.  Un *patrón de diseño* es una solución de diseño bien
probada a un problema o conjuntos de problemas y suele describirse de
una manera semiformal en términos de objetos y relaciones entre ellos.
El primer libro publicado sobre este tema (se conoce popularmente como
*Gang of Four*, GOF) sigue siendo una referencia fundamental {{
"gamma95:_desig_patter" | cite }} pero ya no es la única.  Cabe
destacar la serie de volúmenes de *Pattern-Oriented Software
Architecture* (POSA)
{{ "buschmann96:_patter_orien_softw_archit_vol1" | cite }}
{{ "schmidt00:_patter_orien_softw_archit_vol2" | cite }}
{{ "kircher04:_patter_orien_softw_archit_vol3" | cite }}
{{ "buschmann07:_patter_orien_softw_archit_vol4" | cite }}
{{ "buschmann07:_patter_orien_softw_archit_vol5" | cite }}.

Para este taller es especialmente importante el patrón
[*reactor*](https://es.wikipedia.org/wiki/Reactor_(patr%C3%B3n_de_dise%C3%B1o))
que se encuentra descrito en el volumen 2 de POSA.  Se trata de un
patrón arquitectural, en el sentido de que determina la estructura de
la aplicación en términos de componentes y cómo se relacionan.  Nos
ayuda a organizar el programa cuando se trata de un *sistema
reactivo*.  Es decir, cuando el sistema responde lo más rápidamente
posible a *eventos* externos o internos.

Un sistema electrónico es muy frecuentemente un sistema reactivo.
Reacciona cuando se pulsan botones, o se detecta luz, o se detecta
proximidad, o se detecta humedad, o se recibe un flanco de la señal de
un *encoder*, o se activa un *final de carrera*, o se pulsa una tecla
del teclado, o se pulsa un botón del ratón, o se recibe un dato por la
red, ... Prácticamente todo lo que proporciona datos al sistema
(entradas) puede modelarse como fuentes de eventos.

Cualquier desarrollo progresará de forma incremental, primero con una
fuente de eventos, y luego otra, y luego otra.  No es infrecuente ver
programas que se convierten en un auténtico galimatías que es
imposible hacer funcionar.  La gama de aberraciones es muy amplia:

* Algunos utilizan un bucle infinito enorme, en el que se leen todas
  las entradas y se van haciendo cálculos parciales mezclados con el
  proceso de lectura de eventos.  Frecuentemente no es posible tener
  un resultado hasta que vengan otros eventos, así que se guardan
  valores en variables temporales para tratarlo en otras iteraciones
  del bucle.  La maraña de `if` encadenados se hace inmanejable.  El
  resultado es extremadamente difícil de seguir hasta para un experto.
  Solo pensar en añadir una nueva entrada produce escalofríos.

* Otros hacen todo en manejadores de interrupción. Al pulsar un botón
  se dispara un manejador de interrupcción que se utiliza directamente
  para cambiar el color de un LED.  Cuando el sistema es diminuto
  parece que funciona de maravilla.  Luego va creciendo y empiezan los
  problemas.  Primero fallos catastróficos inexplicables, que terminan
  en maldiciones contra los punteros de C.  Luego, cuando se es
  consciente de la necesidad de *async-safety* se empiezan a
  utilizar primitivas de sincronización y aparecen los interbloqueos.

No hay nada que podamos hacer para arreglar esto, la única solución
rentable a largo plazo es analizar el programa para ver cada requisito
funcional y volverlo a construir desde cero con una arquitectura
razonable.  Hay gente que no cree esto e invierte meses en intentar
depurar el software.

Incluso he llegado a ver casos en los que aparentemente se ha
conseguido que funcione, pero es una falacia, no es real. ¿Por qué?
Pues simplemente porque se equivoca el propósito del programa.  El
programa no está solo ni principalmente para ser ejecutado.  Está
fundamentalmente para ser leído y modificado.  Por eso usamos un
lenguaje de alto nivel.  La vida del programa no acaba cuando se
ejecuta.

Una arquitectura incorrecta pone una losa en la espalda de todos los
que tengan que modificar el programa en el futuro.  Es peor que no
hacer nada, es comprometer gasto en el futuro.  La productividad de
estos programadores habría que contabilizarla como un número negativo.

El patrón *reactor* no es la única arquitectura posible, pero es
probablemente la más apropiada para el tipo de sistemas de este
taller.  Consiste en asociar cada tipo de eventos con un objeto
manejador (`event_handler`).  El manejador tiene un método para
consumir los eventos pendientes de ese tipo (`handle`).  Por otro lado
hay un objeto `reactor` que encapsula todo lo que tiene que ver con la
demultiplexación de eventos (*dispatching*).  Cuando ocurre cierto
evento busca al manejador correspondiente para invocar su método
`handle_events`.

Cada vez que incorporemos una nueva fuente de eventos es necesario
crear un manejador de eventos asociado y registrarlo en el `reactor`.
También puede considerarse el tiempo como una fuente de eventos, pero
se tratan de forma especial, porque hay que indicar un tiempo de
disparo o frecuencia de invocación.

En el taller utilizaremos una implementación propia de este patrón que
se incluye en la carpeta `reactor`.  Incluye numerosas
implementaciones de diferentes `event_handler` para eventos de
interés.  Tómate tu tiempo para analizarlos en detalle.

El `reactor` tiene un método `reactor_run` que implementa el
denominado *bucle de eventos*.  Es el bucle en el que se detecta si
hay eventos disponibles y en caso de que los hubiera se *despachan*
usando su manejador correspondiente.

Vamos a hacer un breve recorrido por los manejadores de eventos
incluídos en la biblioteca `reactor` del taller.

## Eventos de teclado

El teclado en C suele leerse con funciones como `getchar`.  Pero ésta,
como todas las demás funciones de `stdio`, no produce un valor
inmediatemente cuando se pulsa una tecla. Se puede pulsar toda una
secuencia de teclas pero hasta que no se presione la tecla *Intro* no
se recibirá ni una sola pulsación.

Este modo de funcionamiento está pensado para aliviar la carga de
trabajo en el modo habitual de uso, para editar textos u órdenes.  Los
programas solo reciben información cuando tienen algo significativo
que hacer.

Las funciones de `stdio` utilizan siempre objetos de tipo `FILE`.
Incluso cuando no se indica (`printf`, `scanf`) trabajan con variables
`FILE` globales (`stdout`, `stdin`).  Estos objetos incorporan
*arrays* que actúan como *buffers* de la entrada.  Retienen los
caracteres leídos hasta que se disponga de suficiente información.
Evidentemente con `FILE` nunca vamos a conseguir una respuesta
inmediata al pulsar una tecla.

Tenemos que mirar por tanto a la interfaz del *sistema operativo*, lo
que usa `stdio` en su implementación.  Y aquí estamos de enhorabuena,
porque GNU replica el modelo de Unix que se caracteriza por su
simplicidad.

* En Unix todo son archivos o dispositivos que se comportan como
  archivos (teclado, ratón, terminales, puertos serie, discos,
  micrófono, red, gráficos, etc.)

* Todos los archivos y dispositivos se manejan con cuatro operaciones
  básicas: `open`, `read`, `write`, `close` y excepcionalmente con una
  quinta `ioctl`.

* Para usar un archivo o dispositivo debe abrirse con `open`.  A
  partir de entonces se obtiene un entero (*descriptor de archivo*)
  que representa el archivo en todas las demás llamadas.

* En todos los programas la entrada estándar está disponible como un
  archivo ya abierto con el *descriptor de archivo* 0.  Análogamente
  la salida estándar y la salida de error estándar están disponibles
  en los descriptores 1 y 2 respectivamente.

Esta uniformidad de Unix es lo que explota nuestra implementación del
*reactor* para que la demultiplexión de eventos sea siempre de
archivos Unix.  Ya veremos qué hacer cuando no hay tales archivos
(e.g. GPIO).

Visto esto parece que lo único que tenemos que hacer es leer del
descriptor 0 usando `read`.  Pero si lo intentamos veremos que sigue
sin recibirse las teclas hasta que se pulse *Intro*.  El motivo es que
el dispositivo está configurado para que solo interrumpa al procesador
si tiene pendientes cierto número de caracteres, o si ha pasado cierto
tiempo desde la última interrupción, o si se ha pulsado *Intro*.

En los archivos `console.h` y `console.c` de la biblioteca `reactor`
te proporcionamos un par de funciones `console_set_raw_mode` y
`console_restore`.  La primera permite la realimentación inmediata del
teclado, configurando los parámetros adecuados y devuelve un *puntero
opaco* con información de la configuración previa.  La segunda permite
restaurar la configuración previa. Veamos un ejemplo de uso en
combinación con el `reactor`.

Siguiendo el patrón *reactor* encapsulamos la interacción con
cualquier fuente de eventos como un *event handler*:

```
#include <reactor/reactor.h>

static void keyboard(event_handler* ev);

event_handler* keyboard_handler_new()
{
    return event_handler_new(0, keyboard);
}

static void keyboard(event_handler* ev)
{
    char buf[1];
    if (read(ev->fd, buf, 1) < 0 || buf[0] == 'q')
        reactor_quit(ev->r);

    printf("Pulsado %c\n", buf[0]);
}
```

Este manejador de ejemplo utiliza al `reactor` porque invoca a su
método `reactor_quit` (salir del bucle de eventos del *reactor*)
cuando detecta una condición de terminación.  Para ello utiliza uno de
los atributos del *event handler*.  Fíjate en la llamada a
`event_handler_new`.  Delegamos en `event_handler` todo lo que podemos
y le pasamos el descriptor de archivo 0 y la función de manejo de
eventos.  El descriptor 0 corresponde a la entrada estándar.

El programa principal es sencillo:

```
#include <reactor/reactor.h>
#include <reactor/console.h>

int main()
{
    void* state = console_set_raw_mode(0);
    reactor* r = reactor_new();
    reactor_add(r, keyboard_handler_new());
    reactor_run(r);
    console_restore(0, state);
	return 0;
}
```

Primero ponemos la entrada estándar (descriptor 0) en modo *raw* para
tener respuesta inmediata de las pulsaciones de tecla. Luego
construimos el *reactor* y todos sus manejadores de eventos (en este
caso solo el del teclado).  Registramos los manejadores en el
*reactor* y entramos en el bucle de eventos (`reactor_run`).  Una vez
terminado el programa dejamos la consola en el estado normal.

Algunos de vosotros estaréis pensando que esto es matar moscas a
cañonazos, que es mucho más simple un bucle sin más, sin reactor ni
manejadores.  Algo de este estilo:

```
int main()
{
    void* state = console_set_raw_mode(0);
    for(;;) {
        char buf[1];
        if (read(0, buf, 1) < 0 || buf[0] == 'q')
            break;
        printf("Pulsado %c\n", buf[0]);
    }
    console_restore(0, state);
}
```

Funciona exactamente igual, es mucho más corto y hasta se entiende
mejor. ¿Por qué complicarlo?  Pues porque la vida es compleja, no es
así de simple.  No hay una fuente de eventos, hay decenas.  No hay que
procesar un evento, sino varios.  No son eventos independientes, sino
relacionados, hay eventos periódicos, hay fuentes que aparecen y
desaparecen, etc.  Te proponemos un ejercicio, empieza con este
ejemplo y sigue con él haciendo en paralelo todos los ejemplos que te
mostramos.  Después si lo consigues comparamos.

Si te frustra escribir tanto cambia de lenguaje.  El problema no está
en el programa, sino en el lenguaje, que es demasiado primitivo.  Usa
C++ o Python.


## Eventos en una entrada digital

Las patas de GPIO configuradas como entradas son una fuente frecuente
de eventos para el sistema.  De hecho en un sistema empotrado será
mucho más frecuente que eventos de un teclado USB.  Los teclados de
los sistemas empotrados se implementan frecuentemente con patas de
GPIO.

Para tratar estos eventos en cualquier conjunto de entradas incluimos
el `input_handler`.  Se configuran automáticamente como entradas con
*pull down* y se detectan tanto las transiciones de nivel alto a bajo
como de nivel bajo a alto.

```
#include <reactor/reactor.h>
#include <reactor/input_handler.h>
#include <wiringPi.h>
#include <stdio.h>

static void press(input_handler* ev, int key) { printf("Press %d\n", key); }
static void release(input_handler* ev, int key) { printf("Release %d\n", key); }

int main()
{
    int buttons[] = { 18, 23, 24, 25 };

    wiringPiSetupGpio();
    reactor* r = reactor_new();
    reactor_add(r, (event_handler*) input_handler_new(buttons, 4,
						      press, release));
    reactor_run(r);
}
```

Este ejemplo muestra cómo configuraríamos un conjunto de cuatro
botones en la Raspberry Pi. Se detectarían tanto las pulsaciones como
las liberaciones de cada botón.


## Eventos disparados por tiempo

Además de los eventos que ocurren en el entorno, hay otra magnitud de
importancia capital en los sistemas reactivos, el tiempo.  Hay
multitud de características que dependen del tiempo.  Por ejemplo, un
simple LED puede parpadear durante unos segundos.  Esto exige contar
el tiempo en que el LED se enciende o se apaga y también el tiempo
total de parpadeo.

### Eventos periódicos

Nuestra implementación del *reactor* implementa *periodic handlers*.
Un *periodic handler* crea un nuevo hilo que va generando eventos cada
cierto número de milisegundos en un descriptor especial denominado
*pipe* (tubería).  Realmente los detalles no es necesario conocerlos,
pero conviene saber que aunque aparentemente el tiempo no está
relacionado con un archivo nuestra implementación lo traduce a eventos
en un archivo especial.

Usar *periodic handlers* es igual de sencillo que cualquier otro:

```
#include <reactor/reactor.h>
#include <reactor/periodic_handler.h>
#include <stdio.h>

void handler(event_handler* ev)
{
    puts("Tick");
}

int main()
{
    reactor* r = reactor_new();
    reactor_add(r, (event_handler*)periodic_handler_new(1000, handler));
    reactor_run(r);
    return 0;
}
```

Este ejemplo imprime un mensaje cada 1000 milisegundos, es decir, cada
segundo.

Los manejadores de eventos se desinstalan automáticamente cuando
ocurre una excepción, así que si queremos ejecutar un evento cierto
número de veces podemos hacerlo contando el número de disparos.

```
void handler(event_handler* ev)
{
    static int i = 0;
    if (++i > 5)
	    Throw Exception(0,"");
	puts("Tick");
}

int main()
{
    reactor* r = reactor_new();
    reactor_add(r, (event_handler*)periodic_handler_new(100, handler));
    reactor_run(r);
    return 0;
}
```

> **Warning**
> Este ejemplo dispara el periodic cinco veces. ¿Qué pasaría si pasado
> cierto tiempo añadimos otra vez al reactor otro *periodic handler*
> igual?  ¿Funcionaría?  Propón una solución y discútela con tus
> compañeros.

### Eventos retardados

Otro tipo de eventos disparados por tiempo es el caso de los eventos
retardados.  Por ejemplo, pasado cierto tiempo si no se cumple cierta
condición salir del programa.

```
#include <reactor/reactor.h>
#include <reactor/delayed_handler.h>

void timeout(event_handler* ev)
{
    if (condicion_de_salida(ev))
        reactor_quit(ev->r);
}

int main()
{
    reactor* r = reactor_new();
    ...
    reactor_add(r, (event_handler*) delayed_handler_new(1500, timeout));
    reactor_run(r);
}
```

Un `delayed_handler` precisamente hace esto.  Espera cierto tiempo de
milisegundos antes de invocar el manejador.  Una vez invocado se
desinstala automáticamente.

### Parpadeo de un LED

Un caso de evento temporal que es muy frecuente en un sistema
empotrado es el de hacer parpadear un LED cierto número de veces.
Esto se consigue con un manejador especial denominado `blink_handler`.
Se indica el pin donde se conecta, el número de milisegundos que debe
permanecer en cada estado (encendido o apagado) y el número de
parpadeos (ciclos encendido/apagado) que debe realizar.  Se encarga
automáticamente de configurar como salida el pin y de destruir el
manejador una vez terminada la secuencia.

```
#include <reactor/reactor.h>
#include <reactor/blink_handler.h>
#include <wiringPi.h>

int main()
{
    wiringPiSetupGpio();
    reactor* r = reactor_new();
    reactor_add(r, (event_handler*) blink_handler_new(18, 200, 5));
    reactor_run(r);
}
```

En este caso el LED conectado a la pata GPIO18 parpadeará 5 veces con
un periodo de 400ms (200ms encendido y 200ms apagado).


## Programación en red (patrón *acceptor-connector*)

Ya hemos visto la interfaz de programación *socket* que ofrece
GNU/Linux para programar comunicaciones en red.  Solo nos falta
organizarlo de manera razonable.  Hemos visto que en las
comunicaciones se pueden identificar dos roles, el rol del *servidor*
y el rol del *cliente*.  El rol del *servidor* es pasivo, espera hasta
que ocurra un evento (la conexión) y entonces reacciona.

Acceptor-Connector es un patrón de diseño propuesto por Douglas
C. Schmidt {{ "schmidt97:_accep" | cite }} y utilizado extensivamente
en ACE (*Adaptive Communications Engine*), su biblioteca de
comunicaciones.  Se ocupa de la primera parte de la comunicación,
desacopla el establecimiento de conexión y la inicialización del
servicio del procesamiento que se realiza una vez que el servicio está
inicializado.  Para ello intervienen tres componentes: *acceptors*,
*connectors* y manejadores de servicio *service handlers*.  Un
*connector* representa el rol activo, y solicita una conexión a un
*acceptor*, que representa el rol pasivo. Cuando la conexión se
establece ambos crean un manejador de servicio que procesa los datos
intercambiados en la conexión.

Veamos un ejemplo sencillo, un servidor de *echo*.  Se trata de un
programa que devuelve lo mismo que se le envía por cualquier conexión.

```
#include <reactor/reactor.h>
#include <reactor/socket_handler.h>

void handle_echo(event_handler* ev)
{
    char buf[128];
    int n = event_handler_recv(ev, buf, sizeof(buf));
    event_handler_send(ev, buf, n);
}

int main()
{
    reactor* r = reactor_new();
    reactor_add(r, (event_handler*)acceptor_new ("10000", handle_echo));
    reactor_run(r);
    return 0;
}
```


El `acceptor` se encargará de llamar a *listen* y *accept* cuando sea
preciso.  En el momento en que se establezca una nueva conexión se
creará un `event_handler` que actúa de *service handler* con el
*socket* esclavo y con la función de procesamiento que se le indica.
En este caso escucha en el puerto TCP 10000.

El `conector` es similar, salvo por el hecho de que en el constructor
establece la conexión con el otro extremo.  Si éste no está disponible
se elevará una excepción.  Si solo se pretende usarlo para enviar
datos al servidor ni siquiera es necesario añadirlo al *reactor*.

```
#include <reactor/reactor.h>
#include <reactor/socket_handler.h>

void handler(event_handler* ev)
{
    char buf[128];
    int n = event_handler_recv(ev, buf, sizeof(buf));
    buf[n] = '\0';
    printf("Got: %s\n", buf);
}

int main()
{
    reactor* r = reactor_new();
    connector* c = connector_new("localhost", "10000", handler);
    reactor_add(r, (event_handler*)c);
    connector c_aux = *c;
    
    void producer(event_handler* ev)
    {
	    static int i;
		char buf[128];
		snprintf(buf, 128, "Prueba %d", i++);
		connector_send(&c_aux, buf, strlen(buf));
    }

    reactor_add(r, (event_handler*)periodic_handler_new(1000, producer));
    reactor_run(r);
    return 0;
}
```

El `connector` crea el *event handler* para atender la conexión.
Podemos usarlo directamente con `connector_send` y `connector_recv` o
bien añadirlo a un `reactor` para responder de forma automática.  En
este ejemplo creamos un evento periódico que envía por el conector un
mensaje distinto cada vez y añadimos el `connector` al *reactor* para
recibir el mensaje de eco del servidor anterior.

Nota que el evento periódico no usa el puntero al conector
directamente.  En su lugar hace una copia de todo el objeto y usa esa
copia.  El motivo es simple, la memoria del objeto conector puede ser
liberada en cualquier momento (por ejemplo, si se pierde la conexión).
En ese caso ese espacio de memoria será ocupado por otros objetos y el
campo correspondiente al descriptor de archivo (el socket en este
caso) podría ser alterado.  El evento periódico necesita enviar al
descriptor correcto, aunque haya sido cerrado por un problema de
comunicaciones.  Si está cerrado se elevará una excepción y el evento
periódico también se desinstalará automáticamente, como cabría
esperar.

Los sistemas en los que un componente actúa fundamentalmente con el
rol de servidor, mientras que los otros componentes actúan como
clientes siguen la *arquitectura cliente-servidor*.  Es la típica en
la *World-Wide-Web*.

Otras aplicaciones no tienen una división tan marcada.  En un momento
dado una aplicación que normalmente se comporta como servidora puede
comportarse como cliente y al revés.  Este tipo de sistemas en que
todos los componentes toman rol de cliente o de servidor
indistintamente se denominan arquitecturas *peer-to-peer*.

## Un *media player* como manejador

## Otros manejadores

### *Pipe handler*

Cuando queremos traducir eventos de otro tipo para que sean
demultiplexados por el *reactor*.

### *Thread handler*

Si además se necesita un hilo.

### *Process handler*

Para manejar programas externos de forma bidireccional.
