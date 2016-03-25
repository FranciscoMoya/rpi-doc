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

## Manejo de errores

Antes de empezar a escribir programas relativamente complejos en C
conviene hacer una pequeña reflexión sobre la gestión de errores en C.
La forma habitual de manejar errores es devolver un código de error en
las funciones y, dependiendo de su valor actuar de una forma u otra.

Pero ¿qué pasa si no sabemos cómo actuar ante una situación errónea?
Es muy frecuente que en el momento que se produce el error no tengamos
toda la información necesaria para tratarlo debidamente.  Es habitual
devolver otro código de error al llamador, pero esto empieza a
oscurecer el código porque cada llamada a función tiene que ir en una
claúsula `if`.

Lo correcto es emplear un mecanismo soportado por todos los lenguajes
de programación modernos, que se denomina *excepciones*.  Pero C no
tiene excepciones. Afortunadamente hay un conjunto de bibliotecas que
implementan el mecanismo utilizando macros del preprocesador y dos
funciones de la biblioteca estándar de C que no suelen ser muy
conocidas, `setjmp` y `longjmp`.  

No es propósito de este taller explicar el funcionamiento de estas
funciones, mira la página de manual para entender su funcionamiento.
Son esenciales para implementar corrutinas, o cualquier otra forma de
interrumpir el flujo normal de ejecución de un programa.

En nuestros ejemplos vamos a optar por
[`cexcept`](http://cexcept.sf.net/) por su extrema simplicidad.  Tiene
un único archivo de cabecera (`cexcept.h`) y su uso es muy simple.

Cuando el programa tiene suficiente información para manejar posibles
errores debe utilizar una construcción de este estilo:

```
Try {
    /* Código de usuario, sin ´manejo de errores */ 
}
Catch (ex) {
    /* Código para tratar el error notificado en ex */
}
```

En el momento en que se puede producir una situación excepcional o un
error debe notificarse de esta forma:

```
if (funcion_tradicional() < 0)
   Throw ex;
```

Donde `ex` es una excepción, una estructura de datos que recoge toda
la información del error para ser manejada cuando esto sea posible.

El tipo de datos de `ex` es definible por el usuario.  En nuestros
ejemplos utilizaremos una biblioteca auxiliar en la que definimos la
excepción como una estructura similar a esta:

```
typedef struct {
    int error_code;
    const char* what;
} exception;

define_exception_type(exception);
```

Así que si necesitas una descripción textual del error puedes usar el
campo `what` de la estructura.  Por ejemplo:

```
Try {
    guardar_datos(db);
}
Catch (ex) {
    printf("No pudo grabar los datos.\n"
           "Motivo: %s\n", ex.what);
}
```

Para notificar una situación excepcional se dice que se eleva una
excepción.  Es decir, se usa una sentencia `Throw` con los valores
apropiados de la excepción.  Por ejemplo:

```
f = fopen(fname, "r");
if (f == NULL)
    Throw Exception(errno, "")
```

Cuando se ejecuta la sentencia `Throw` el programa pasa el control a
la claúsula `Catch` del último `Try` ejecutado, aunque esté en otra
función.  Es un mecanismo muy poderoso para desacoplar la lógica del
programa y la lógica de manejo de errores.

No te quedes en esta breve explicación, mira tranquilamente los
ejemplos que te proporcionamos en el taller, especialmente la
biblioteca `reactor`.  Puede que las siguientes secciones te ayuden
también a comprender el código.

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
    empleado* e = (empleado*)malloc(sizeof(empleado));
    empleado_init(e, nombre, puesto, sueldo);
    return e;
}

void empleado_init(empleado* e,
                   const char* nombre, 
                   const char* puesto,
                   double sueldo)
{
    e->nombre = strdup(nombre);
    e->puesto = strdup(puesto);
    e->sueldo = sueldo;
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
void empleado_destroy(empleado* e)
{
    e->destroy(e);
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
static void empleado_free_members(empleado* e);

void empleado_init(empleado* e,
                   const char* nombre, 
                   const char* puesto,
                   double sueldo)
{
    e->nombre = strdup(nombre);
    e->puesto = strdup(puesto);
    e->sueldo = sueldo;
    e->destroy = empleado_free_members;
}

static void empleado_free_members(empleado* e) 
{ 
    free(e->nombre);
    free(e->puesto);
}
```

Pero esto no libera la estructura de `empleado` cuando se construye
con `empleado_new`.  Así que en ese caso habrá que cambiar el
destructor:

```
static void empleado_free(empleado* e);

empleado* empleado_new(const char* nombre, 
                       const char* puesto,
                       double sueldo)
{
    empleado* e = (empleado*)malloc(sizeof(empleado));
    empleado_init(e, nombre, puesto, sueldo);
    e->destroy = empleado_free;
    return e;
}

static void empleado_free(empleado* e)
{
    empleado_free_members(e);
    free(e);
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
static double empleado_retribuciones_14pagas(empleado* e)
{
    return 14.*e->sueldo;
}
```

Con lo que el método quedaría tan simple como:

```
double empleado_retribuciones(empleado* e)
{
    return e->retribuciones(e);
}
```

Sin embargo, en el caso del gerente tendríamos un objeto con más
atributos:

```
typedef struct gerente_ gerente;
typedef void (*gerente_func)(gerente*);
struct gerente_ {
    empleado e;
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
static double gerente_retribuciones(empleado* e)
{
    gerente* g = (gerente*) e;
    double r = 14.*e->sueldo;
    if (gerente_objetivos_conseguidos(g))
        r += g->bonus;
    return r;
}

void gerente_init(gerente* g,
                  const char* nombre, 
                  const char* puesto,
                  double sueldo,
                  double bonus)
{
    empleado* e = &g->e;
    empleado_init(e, nombre, puesto, sueldo);
    e->retribuciones = gerente_retribuciones;
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

## Reactor


