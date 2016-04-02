[//]: # (-*- markdown -*-)

# Arquitectura software (versión en C)

Una de las principales diferencias de esta edición del taller es que
vamos a dedicar una buena parte del tiempo a hablar de arquitectura
software.

Este capítulo tiene dos objetivos:

* Ofrecer unas pinceladas de los patrones de diseño que consideramos
  más importantes para el desarrollo con Raspberry Pi. Documentar
  brevemente los fundamentos y cómo se aplican.

* Proporcionar plantillas de aplicación totalmente funcionales y
  extensibles para aplicarlas en cualquier proyecto. Esto incluye el
  estudio de diversos casos de estudio, incluyendo la interacción a
  través de la red, la interacción con programas externos, etc.



## Fundamentos de programación orientada a objetos

Puede que te haya extrañado el título de esta sección. ¿Programación
orientada a objetos? ¿En C?  La programación orientada a objetos (POO)
no es más que una técnica de programación.  Cuando se dice que un
lenguaje *soporta* POO significa que incorpora mecanismos específicos
para hacer más fácil la POO. Pero en cualquier lenguaje actual se
puede programar orientado a objetos.

Un objeto no es más que una zona de memoria con semántica asociada.
Es decir, que tienen significado y hay un conjunto de operaciones que
tiene sentido realizar sobre ellos.  Un entero, por ejemplo, es un
objeto en C. Tiene significado matemático y se pueden realizar las
operaciones de suma, resta, etc. ¿Cómo hacemos objetos arbitrarios en
C?  Por ejemplo, ¿cómo hacemos un objeto que represente un `empleado`
en un registro de personal?

La respuesta es con ayuda de una estructura, por ejemplo:

```
typedef struct empleado_ empleado;
struct empleado_ {
    const char* nombre;
    const char* puesto;
    double sueldo;
};
```

Los distintos elementos de la estructura se denominan *atributos del
objeto*.  Es todo aquello que le da significado a cada uno de los
objetos.

Pero todavía no hemos hecho objetos, solo hemos definido la forma que
tendrán esos objetos, es lo que se denomina *clase*.  Para construir
objetos se definen funciones que reciben los parámetros necesarios y
devuelven un puntero a una estructura de éstas, los constructores.
Por ejemplo:

```
empleado* empleado_new(const char* nombre, 
                       const char* puesto,
                       double sueldo)
{
    empleado* this = (empleado*)malloc(sizeof(empleado));
    empleado_init(this, nombre, puesto, sueldo);
    return this;
}

void empleado_init(empleado* this,
                   const char* nombre, 
                   const char* puesto,
                   double sueldo)
{
    this->nombre = strdup(nombre);
    this->puesto = strdup(puesto);
    this->sueldo = sueldo;
}
```

Los constructores en C suelen dividirse en dos partes, un `XXX_new` y
un `XXX_init`.  El primero reserva espacio en memoria dinámica,
mientras que el segundo solo rellena los datos.  Esto resulta útil
para poder definir empleados en memoria dinámica o en variables
automáticas:

```
empleado* ed = empleado_new("Paco", "Jefe", 1800.);
empleado ea;
empleado_init(&ea, "Pepe", "Currito", 1000.);
```

El empleado `ed` es un objeto en memoria dinámica, que debe ser
liberado llamando a `free`.  Sin embargo el empleado `ea` es un objeto
en la pila, que se libera automáticamente cuando termine el ámbito de
declaración.  La segunda forma es también muy útil para construir
vectores de `empleado`.

Algunos de vosotros os habréis dado cuenta de que aquí falla algo.  Si
el objeto `ea` se libera no vamos a poder liberar las cadenas
correspondientes a `nombre` o `puesto`.  La solución es utilizar una
función especial para liberar todo lo reservado en el constructor, el
destructor.  Pero lo que hay que liberar es diferente si se ha
utilizado `XXX_new` o `XXX_init`. ¿Cómo conseguimos que se comporte de
forma diferente dependiendo de dónde haya sido construido?

La técnica para resolver este problema se denomina *funciones
virtuales*.  Hay varias formas de implementarlo, lo haremos de la
forma más simple posible.  En lugar de usar una función para liberar
el objeto vamos a usar un puntero a función y ese puntero cambiará
dependiendo del modo en que se ha construido el objeto.  El decir, el
propio objeto lleva no solo datos sino también punteros a funciones
que pueden cambiar dependiendo del caso.

```
typedef struct empleado_ empleado;
typedef void (*empleado_func)(empleado*);
struct empleado_ {
    const char* nombre;
    const char* puesto;
    double sueldo;
    empleado_func destroy;
};
```

Y el destructor puede ser una función tan simple como:

```
void empleado_destroy(empleado* this)
{
    this->destroy(this);
}
```

Evidentemente el trabajo no está en esa función sino en aquella a la
que apunta `e->destroy`.  Y además es preciso garantizar que siempre
apunta a una función válida, porque de otro modo al llamar al
destructor se produciría un error catastrófico.  Ésta y cualquier otra
garantía que sea preciso mantener para que el estado sea siempre
consistente se denominan *invariantes de clase*.

El constructor es responsable de *establecer* los invariantes de
clase. Todas las demás operaciones son responsables de *mantener* los
invariantes de clase.  Veamos cómo quedaría la función
`empleado_init`.

```
static void empleado_free_members(empleado*);

void empleado_init(empleado* this,
                   const char* nombre, 
                   const char* puesto,
                   double sueldo)
{
    this->nombre = strdup(nombre);
    this->puesto = strdup(puesto);
    this->sueldo = sueldo;
    this->destroy = empleado_free_members;
}

static void empleado_free_members(empleado* this) 
{ 
    free(this->nombre);
    free(this->puesto);
}
```

Pero esto no libera la estructura de `empleado` cuando se construye
con `empleado_new`.  Así que en ese caso habrá que cambiar el
destructor:

```
static void empleado_free(empleado*);

empleado* empleado_new(const char* nombre, 
                       const char* puesto,
                       double sueldo)
{
    empleado* this = (empleado*)malloc(sizeof(empleado));
    empleado_init(this, nombre, puesto, sueldo);
    this->destroy = empleado_free;
    return this;
}

static void empleado_free(empleado* this)
{
    empleado_free_members(this);
    free(this);
}
```

La función `empleado_destroy` es un ejemplo de operación que se puede
realizar sobre un `empleado`.  Estas operaciones se llaman de forma
general *métodos* del objeto, y por tratarse de la operación que
libera los recursos del `empleado` se llamaría de forma más específica
*destructor*.  Además es un método que se comporta de forma diferente
según el tipo de `empleado` sobre el que se llame.  Este tipo de
*métodos* se llaman en general *métodos virtuales*, y al tratarse de un
destructor sería también de forma más específica *destructor virtual*.

Los *métodos virtuales* se utilizan en combinación con otra
característica muy importante de la POO, la herencia. En el ejemplo de
`empleado` podemos considerar el caso de un `gerente` que básicamente
funciona igual que un `empleado`, pero tiene otras características,
como por ejemplo bonus por productividad.  Por tanto el cálculo de las
retribuciones anuales será diferente según se trate de un empleado
normal o de un gerente.  Eso se puede conseguir añadiendo otra función
virtual para ello.

```
typedef struct empleado_ empleado;
typedef void (*empleado_func)(empleado*);
typedef double (*empleado_func_double)(empleado*);
struct empleado_ {
    const char* nombre;
    const char* puesto;
    double sueldo;
    empleado_func destroy;
    empleado_func_double retribuciones;
};
```

El campo `retribuciones` se inicializa en el constructor, de forma
similar a `destroy` utilizando la siguiente función:

```
static double empleado_retribuciones_14pagas(empleado* this)
{
    return 14.*this->sueldo;
}
```

Con lo que el método quedaría tan simple como:

```
double empleado_retribuciones(empleado* this)
{
    return this->retribuciones(this);
}
```

Sin embargo, en el caso del gerente tendríamos un objeto con más
atributos:

```
typedef struct gerente_ gerente;
typedef void (*gerente_func)(gerente*);
struct gerente_ {
    empleado parent;
    double bonus;
};
```

Hemos colocado como primer elemento de la estructura un `empleado`.
Esto hace que la primera parte de un `gerente` sea la que corresponde
a un `empleado`.  Por tanto los métodos de `empleado` siguen
funcionando sobre un objeto de la clase `gerente`, porque solo acceden
a esa primera parte.

Por otro lado, la clase `gerente` debe *redefinir* el método de
cálculo de retribuciones:

```
static double gerente_retribuciones(empleado* this)
{
    gerente* g = (gerente*) this;
    double r = 14. * this->sueldo;
    if (gerente_objetivos_conseguidos(g))
        r += g->bonus;
    return r;
}

void gerente_init(gerente* this,
                  const char* nombre, 
                  const char* puesto,
                  double sueldo,
                  double bonus)
{
    empleado* parent = &this->parent;
    empleado_init(parent, nombre, puesto, sueldo);
    parent->retribuciones = gerente_retribuciones;
}
```

Observa cómo la nueva implementación del método retribuciones
convierte su argumento en un `gerente`.  Si se ha llamado a este
método es seguro que se trata de un gerente.  Ahora podemos calcular
las retribuciones de cualquier empleado sea o no gerente:

```
empleado* equipo[] = {
    (empleado*) gerente_new("Paco", "Jefe", 1800., 4000.),
    empleado_new("Pepe", "Currito", 1000.),
    empleado_new("Juan", "Currito", 1000.),
};

#define NELEMS(a) (sizeof(a)/sizeof(a[0]))
double coste_personal = 0.;
for (int i=0; i<NELEMS(equipo); ++i)
    coste_personal += empleado_retribuciones(equipo[i]);
```

La herencia es un mecanismo muy efectivo para tratar de forma
homogénea a objetos, pero introduce muchísimo acoplamiento.  Intenta
minimizarla lo más posible.

Esta implementación de POO en C es primitiva pero para los fines del
taller nos bastará.  En proyectos reales convendría echar un vistazo a
las bibliotecas que ya existen.  En particular merece la pena destacar
[GObject](https://developer.gnome.org/gobject/stable/) y
[COS](https://sourceforge.net/projects/cos/). Cada una tiene sus
ventajas y sus inconvenientes que habrá que valorar.

## El patrón *reactor*

Uno de los mayores beneficios de utilizar POO es que nos abre la
posibilidad de utilizar directamente la mayor parte del catálogo de
*patrones de diseño* disponibles en la actualidad.  Un *patrón de
diseño* es una solución de diseño bien probada a un problema o
conjuntos de problemas y suele describirse de una manera semiformal en
términos de objetos y relaciones entre ellos.

Para este taller es especialmente importante el patrón
[*reactor*](https://es.wikipedia.org/wiki/Reactor_(patr%C3%B3n_de_dise%C3%B1o)).
Se trata de un patrón arquitectural, en el sentido de que determina la
estructura de la aplicación en términos de componentes y cómo se
relacionan.  Nos ayuda a organizar el programa cuando se trata de un
*sistema reactivo*.  Es decir, cuando el sistema debe responder lo más
rápidamente posible a *eventos* externos o internos.

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
  consciente de la necesidad de *exception safety* se empiezan a
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

El `reactor` tiene un método `reactor_run` que implementa el
denominado *bucle de eventos*.  Es el bucle en el que se detecta si
hay eventos disponibles y en caso de que los hubiera se *despachan*
usando su manejador correspondiente.

En el taller utilizaremos una implementación propia de este patrón que
se incluye en la carpeta `reactor`.  Incluye numerosas
implementaciones de diferentes `event_handler` para eventos de
interés.  Tómate tu tiempo para analizarlos en detalle.

Vamos a hacer un breve recorrido por los manejadores de eventos
incluídos en la biblioteca `reactor` del taller.

### Eventos de teclado

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
  micrófono, etc.)

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
typedef struct {
    event_handler ev;
    reactor* r;
} keyboard_handler;

static void keyboard(event_handler* ev);

event_handler* keyboard_handler_new(reactor* r)
{
    keyboard_handler* kb = malloc(sizeof(keyboard_handler));
    event_handler_init(&kb->ev, 0, keyboard);
    kb->ev.destroy = (event_handler_func) free;
    kb->r = r;
    return &kb->ev;
}

static void keyboard(event_handler* ev)
{
    keyboard_handler* kb = (keyboard_handler*)ev;

    char buf[1];
    if (read(ev->fd, buf, 1) < 0 || buf[0] == 'q')
        reactor_quit(kb->r);

    printf("Pulsado %c\n", buf[0]);
}
```

Este manejador de ejemplo utiliza al `reactor` porque invoca a su
método `reactor_quit` (salir del bucle de eventos del *reactor*)
cuando detecta una condición de terminación.  Por tanto para hacerlo
visible tenemos que añadirlo a los atributos del *event handler*.
Osea, hacemos un *event handler* ampliado, heredamos de él.  Fíjate en
la llamada a `event_handler_init`.  Delegamos en `event_handler` todo
lo que podemos (la inicialización de la primera parte) y le pasamos el
descriptor de archivo 0 y la función de manejo de eventos.  El
descriptor 0 corresponde a la entrada estándar.

El programa principal es sencillo:

```
int main()
{
    void* state = console_set_raw_mode(0);
    reactor* r = reactor_new();
    event_handler* kb = keyboard_handler_new(r);
    reactor_add(r, kb);
    reactor_run(r);
    console_restore(0, state);
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

### Eventos disparados por tiempo

Además de los eventos que ocurren en el entorno, hay otra magnitud de
importancia capital en los sistemas reactivos, el tiempo.  Hay
multitud de características que dependen del tiempo.  Por ejemplo, un
simple LED puede parpadear durante unos segundos.  Esto exige contar
el tiempo en que el LED se enciende o se apaga y también el tiempo
total de parpadeo.

Nuestra implementación del *reactor* procesa este tipo de eventos de
manera especial a través de los *timeout handlers*.

## Programación en red (patrón *acceptor-connector*)

Ya hemos visto la interfaz de programación *socket* que ofrece
GNU/Linux para programar comunicaciones en red.  Solo nos falta
organizarlo de manera razonable.  Hemos visto que en las
comunicaciones se pueden identificar dos roles, el rol del *servidor*
y el rol del *cliente*.  El rol del *servidor* es pasivo, espera hasta
que ocurra un evento (la conexión) y entonces reacciona.

Acceptor-Connector es un patrón de diseño propuesto por Douglas
C. Schmidt~\cite{schmidt97:_accep} y utilizado extensivamente en ACE
(*Adaptive Communications Engine*), su biblioteca de comunicaciones.
Se ocupa de la primera parte de la comunicación, desacopla el
establecimiento de conexión y la inicialización del servicio del
procesamiento que se realiza una vez que el servicio está
inicializado.  Para ello intervienen tres componentes: *acceptors*,
*connectors* y manejadores de servicio *service handlers*.  Un
*connector* representa el rol activo, y solicita una conexión a un
*acceptor*, que representa el rol pasivo. Cuando la conexión se
establece ambos crean un manejador de servicio que procesa los datos
intercambiados en la conexión.

Veamos un ejemplo sencillo, un servidor de *echo*.  Se trata de un
programa que devuelve lo mismo que se le envía por cualquier conexión.

```
#include "reactor.h"

void handle_echo(service_handler* h)
{
    char buf[1024];
    int n = service_handler_recv (h, buf, sizeof(buf));
    if (n <= 0) {
        service_handler_close (h);
        Throw Exception(n, "Connection closed");
    }
    buf[n] =  '\0';
    printf("server: [%s]\n", buf);
    service_handler_send (h, buf, n);
}

int main()
{
    reactor* r = reactor_new();
    reactor_add(r, acceptor_new ("10000", "tcp", handle_echo));
    reactor_run(r);
    return 0;
}
```

El `acceptor` se encargará de llamar a *listen* y *accept* cuando sea
preciso.  En el momento en que se establezca una nueva conexión se
creará un `service_handler` con el *socket* esclavo y con la función
de procesamiento que se le indica.  Cuando se llama a
`service_handler_close` se elimina automáticamente ese manejador del
`reactor`.

El conector es similar, salvo por el hecho de que el complementario
del `acceptor`, el `connector`, no es un `event_handler` en nuestra
implementación.

```
#include "reactor.h"

void handle_echo(service_handler* h)
{
    char buf[1024];
    int n = service_handler_recv (h, buf, sizeof(buf));
    if (n <= 0) {
        service_handler_close (h);
        Throw Exception(n, "Connection closed");
    }
    buf[n] =  '\0';
    printf("client: [%s]\n", buf);
}

int main()
{
    reactor* r = reactor_new();
    connector* c = connector_new ("localhost", "10000", "tcp");
    service_handler* h = connector_connect(c, echo_handler);
    service_handler_send(h, "Hello, World!", 13);
    reactor_add(r, &h->parent);
    reactor_run(r);
    connector_destroy(c);
    return 0;
}
```

El `connector` crea el *service handler* para atender la conexión.
Podemos usarlo directamente con `service_handler_send` y
`service_handler_recv` o bien añadirlo a un `reactor` para responder
de forma automática.  En este ejemplo lo usamos primero directamente y
posteriormente lo añadimos a un `reactor` para que escriba las
respuestas cuando lleguen.

Los sistemas en los que un componente actúa fundamentalmente con el
rol de servidor, mientras que los otros componentes actúan como
clientes siguen la *arquitectura cliente-servidor*.  Es la típica en
la *World-Wide-Web*.

Otras aplicaciones no tienen una división tan marcada.  En un momento
dado una aplicación que normalmente se comporta como servidora puede
comportarse como cliente y al revés.  Este tipo de sistemas en que
todos los componentes toman rol de cliente o de servidor
indistintamente se denominan arquitecturas *peer-to-peer*.

## Casos de estudio

Vamos a explicar someramente algunos de los ejemplos incluídos en el
software que acompaña al taller.  Se trata de ejemplos ligeramente más
complejos que los ejemplos triviales, de forma que pueda apreciarse la
ventaja de una arquitectura software sólida.

### Dispositivo MP3

El primer ejemplo consiste en el desarrollo de un player MP3 con una
Raspberry Pi.  

### Control de accesos

### Control de aparcamiento

### Vehículo autónomo

