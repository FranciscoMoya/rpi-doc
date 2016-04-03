# Fundamentos de POO

POO es la abreviatura de programación orientada a objetos. Puede que
te haya extrañado el título de esta sección. ¿Programación orientada a
objetos? ¿En C?  La programación orientada a objetos no es más que una
técnica de programación.  Cuando se dice que un lenguaje *soporta* POO
significa que incorpora mecanismos específicos para hacer más fácil la
POO. Pero en cualquier lenguaje actual se puede programar orientado a
objetos.

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
minimizarla todo lo posible.

Esta implementación de POO en C es primitiva pero para los fines del
taller nos bastará.  En proyectos reales convendría echar un vistazo a
las bibliotecas que ya existen.  En particular merece la pena destacar
[GObject](https://developer.gnome.org/gobject/stable/) y
[COS](https://sourceforge.net/projects/cos/). Cada una tiene sus
ventajas y sus inconvenientes que habrá que valorar.
