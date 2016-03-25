[//]: # (-*- markdown -*-)

# Arquitectura software (versi�n en C)

Una de las principales diferencias de esta edici�n del taller es que
vamos a dedicar una buena parte del tiempo a hablar de arquitectura
software.

Este cap�tulo tiene dos objetivos:

* Ofrecer unas pinceladas de los patrones de dise�o que consideramos
  m�s importantes para el desarrollo con Raspberry Pi. Documentar
  brevemente los fundamentos y c�mo se aplican.

* Proporcionar plantillas de aplicaci�n totalmente funcionales y
  extensibles para aplicarlas en cualquier proyecto. Esto incluye el
  estudio de diversos casos de estudio, incluyendo la interacci�n a
  trav�s de la red, la interacci�n con programas externos, etc.

## Manejo de errores

Antes de empezar a escribir programas relativamente complejos en C
conviene hacer una peque�a reflexi�n sobre la gesti�n de errores en C.
La forma habitual de manejar errores es devolver un c�digo de error en
las funciones y, dependiendo de su valor actuar de una forma u otra.

Pero �qu� pasa si no sabemos c�mo actuar ante una situaci�n err�nea?
Es muy frecuente que en el momento que se produce el error no tengamos
toda la informaci�n necesaria para tratarlo debidamente.  Es habitual
devolver otro c�digo de error al llamador, pero esto empieza a
oscurecer el c�digo porque cada llamada a funci�n tiene que ir en una
cla�sula `if`.

Lo correcto es emplear un mecanismo soportado por todos los lenguajes
de programaci�n modernos, que se denomina *excepciones*.  Pero C no
tiene excepciones. Afortunadamente hay un conjunto de bibliotecas que
implementan el mecanismo utilizando macros del preprocesador y dos
funciones de la biblioteca est�ndar de C que no suelen ser muy
conocidas, `setjmp` y `longjmp`.  

No es prop�sito de este taller explicar el funcionamiento de estas
funciones, mira la p�gina de manual para entender su funcionamiento.
Son esenciales para implementar corrutinas, o cualquier otra forma de
interrumpir el flujo normal de ejecuci�n de un programa.

En nuestros ejemplos vamos a optar por
[`cexcept`](http://cexcept.sf.net/) por su extrema simplicidad.  Tiene
un �nico archivo de cabecera (`cexcept.h`) y su uso es muy simple.

Cuando el programa tiene suficiente informaci�n para manejar posibles
errores debe utilizar una construcci�n de este estilo:

```
Try {
    /* C�digo de usuario, sin �manejo de errores */ 
}
Catch (ex) {
    /* C�digo para tratar el error notificado en ex */
}
```

En el momento en que se puede producir una situaci�n excepcional o un
error debe notificarse de esta forma:

```
if (funcion_tradicional() < 0)
   Throw ex;
```

Donde `ex` es una excepci�n, una estructura de datos que recoge toda
la informaci�n del error para ser manejada cuando esto sea posible.

El tipo de datos de `ex` es definible por el usuario.  En nuestros
ejemplos utilizaremos una biblioteca auxiliar en la que definimos la
excepci�n como una estructura similar a esta:

```
typedef struct {
    int error_code;
    const char* what;
} exception;

define_exception_type(exception);
```

As� que si necesitas una descripci�n textual del error puedes usar el
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

Para notificar una situaci�n excepcional se dice que se eleva una
excepci�n.  Es decir, se usa una sentencia `Throw` con los valores
apropiados de la excepci�n.  Por ejemplo:

```
f = fopen(fname, "r");
if (f == NULL)
    Throw Exception(errno, "")
```

Cuando se ejecuta la sentencia `Throw` el programa pasa el control a
la cla�sula `Catch` del �ltimo `Try` ejecutado, aunque est� en otra
funci�n.  Es un mecanismo muy poderoso para desacoplar la l�gica del
programa y la l�gica de manejo de errores.

No te quedes en esta breve explicaci�n, mira tranquilamente los
ejemplos que te proporcionamos en el taller, especialmente la
biblioteca `reactor`.  Puede que las siguientes secciones te ayuden
tambi�n a comprender el c�digo.

## Fundamentos de programaci�n orientada a objetos

Puede que te haya extra�ado el t�tulo de esta secci�n. �Programaci�n
orientada a objetos? �En C?  La programaci�n orientada a objetos (POO)
no es m�s que una t�cnica de programaci�n.  Cuando se dice que un
lenguaje *soporta* POO significa que incorpora mecanismos espec�ficos
para hacer m�s f�cil la POO. Pero en cualquier lenguaje actual se
puede programar orientado a objetos.

Un objeto no es m�s que una zona de memoria con sem�ntica asociada.
Es decir, que tienen significado y hay un conjunto de operaciones que
tiene sentido realizar sobre ellos.  Un entero, por ejemplo, es un
objeto en C. Tiene significado matem�tico y se pueden realizar las
operaciones de suma, resta, etc. �C�mo hacemos objetos arbitrarios en
C?  Por ejemplo, �c�mo hacemos un objeto que represente un `empleado`
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

Pero todav�a no hemos hecho objetos, solo hemos definido la forma que
tendr�n esos objetos, es lo que se denomina *clase*.  Para construir
objetos se definen funciones que reciben los par�metros necesarios y
devuelven un puntero a una estructura de �stas, los constructores.
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
un `XXX_init`.  El primero reserva espacio en memoria din�mica,
mientras que el segundo solo rellena los datos.  Esto resulta �til
para poder definir empleados en memoria din�mica o en variables
autom�ticas:

```
empleado* ed = empleado_new("Paco", "Jefe", 1800.);
empleado ea;
empleado_init(&ea, "Pepe", "Currito", 1000.);
```

El empleado `ed` es un objeto en memoria din�mica, que debe ser
liberado llamando a `free`.  Sin embargo el empleado `ea` es un objeto
en la pila, que se libera autom�ticamente cuando termine el �mbito de
declaraci�n.  La segunda forma es tambi�n muy �til para construir
vectores de `empleado`.

Algunos de vosotros os habr�is dado cuenta de que aqu� falla algo.  Si
el objeto `ea` se libera no vamos a poder liberar las cadenas
correspondientes a `nombre` o `puesto`.  La soluci�n es utilizar una
funci�n especial para liberar todo lo reservado en el constructor, el
destructor.  Pero lo que hay que liberar es diferente si se ha
utilizado `XXX_new` o `XXX_init`. �C�mo conseguimos que se comporte de
forma diferente dependiendo de d�nde haya sido construido?

La t�cnica para resolver este problema se denomina *funciones
virtuales*.  Hay varias formas de implementarlo, lo haremos de la
forma m�s simple posible.  En lugar de usar una funci�n para liberar
el objeto vamos a usar un puntero a funci�n y ese puntero cambiar�
dependiendo del modo en que se ha construido el objeto.  El decir, el
propio objeto lleva no solo datos sino tambi�n punteros a funciones
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

Y el destructor puede ser una funci�n tan simple como:

```
void empleado_destroy(empleado* e)
{
    e->destroy(e);
}
```

Evidentemente el trabajo no est� en esa funci�n sino en aquella a la
que apunta `e->destroy`.  Y adem�s es preciso garantizar que siempre
apunta a una funci�n v�lida, porque de otro modo al llamar al
destructor se producir�a un error catastr�fico.  �sta y cualquier otra
garant�a que sea preciso mantener para que el estado sea siempre
consistente se denominan *invariantes de clase*.

El constructor es responsable de *establecer* los invariantes de
clase. Todas las dem�s operaciones son responsables de *mantener* los
invariantes de clase.  Veamos c�mo quedar�a la funci�n
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
con `empleado_new`.  As� que en ese caso habr� que cambiar el
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

La funci�n `empleado_destroy` es un ejemplo de operaci�n que se puede
realizar sobre un `empleado`.  Estas operaciones se llaman de forma
general *m�todos* del objeto, y por tratarse de la operaci�n que
libera los recursos del `empleado` se llamar�a de forma m�s espec�fica
*destructor*.  Adem�s es un m�todo que se comporta de forma diferente
seg�n el tipo de `empleado` sobre el que se llame.  Este tipo de
*m�todos* se llaman en general *m�todos virtuales*, y al tratarse de un
destructor ser�a tambi�n de forma m�s espec�fica *destructor virtual*.

Los *m�todos virtuales* se utilizan en combinaci�n con otra
caracter�stica muy importante de la POO, la herencia. En el ejemplo de
`empleado` podemos considerar el caso de un `gerente` que b�sicamente
funciona igual que un `empleado`, pero tiene otras caracter�sticas,
como por ejemplo bonus por productividad.  Por tanto el c�lculo de las
retribuciones anuales ser� diferente seg�n se trate de un empleado
normal o de un gerente.  Eso se puede conseguir a�adiendo otra funci�n
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
similar a `destroy` utilizando la siguiente funci�n:

```
static double empleado_retribuciones_14pagas(empleado* e)
{
    return 14.*e->sueldo;
}
```

Con lo que el m�todo quedar�a tan simple como:

```
double empleado_retribuciones(empleado* e)
{
    return e->retribuciones(e);
}
```

Sin embargo, en el caso del gerente tendr�amos un objeto con m�s
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
a un `empleado`.  Por tanto los m�todos de `empleado` siguen
funcionando sobre un objeto de la clase `gerente`, porque solo acceden
a esa primera parte.

Por otro lado, la clase `gerente` debe *redefinir* el m�todo de
c�lculo de retribuciones:

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

Observa c�mo la nueva implementaci�n del m�todo retribuciones
convierte su argumento en un `gerente`.  Si se ha llamado a este
m�todo es seguro que se trata de un gerente.  Ahora podemos calcular
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
homog�nea a objetos, pero introduce much�simo acoplamiento.  Intenta
minimizarla lo m�s posible.

Esta implementaci�n de POO en C es primitiva pero para los fines del
taller nos bastar�.  En proyectos reales convendr�a echar un vistazo a
las bibliotecas que ya existen.  En particular merece la pena destacar
[GObject](https://developer.gnome.org/gobject/stable/) y
[COS](https://sourceforge.net/projects/cos/). Cada una tiene sus
ventajas y sus inconvenientes que habr� que valorar.

## Reactor


