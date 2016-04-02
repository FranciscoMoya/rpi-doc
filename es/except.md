# Manejo de errores

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

Lo correcto hoy en d�a es emplear un mecanismo soportado por todos los
lenguajes de programaci�n modernos, que se denomina *excepciones*.
Pero C no tiene excepciones. Afortunadamente hay un conjunto de
bibliotecas que implementan el mecanismo utilizando macros del
preprocesador y dos funciones de la biblioteca est�ndar de C que no
suelen ser muy conocidas, `setjmp` y `longjmp`.

No es prop�sito de este taller explicar el funcionamiento de estas
funciones.  Si tienes curiosidad mira la p�gina de manual para
entender su funcionamiento.  Son esenciales para implementar
corrutinas, o cualquier otra forma de interrumpir el flujo normal de
ejecuci�n de un programa.

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
El tipo de datos de `ex` es definible por el usuario.

En nuestros ejemplos utilizaremos `reactor/exception.h`, una
biblioteca auxiliar en la que definimos la excepci�n como una
estructura similar a esta:

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
exception ex;
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

## Anidamiento de `Try/Catch`

En algunos casos se sabe c�mo manejar algunos errores, pero no todos.
En esos casos puede ser �til capturar la posible excepci�n y volverla
a lanzar si no se sabe qu� hacer con ella.  Por ejemplo:

```
void f() {
    exception e;
    Try {
        procesamiento_complejo();
        mas_procesamiento_complejo();
        todavia_mas_procesamiento_complejo();
    }
    Catch(e) {
        if (e.error_code != ERROR_NO_MAS_DATOS)
            Throw e;
    }
}

void g() {
    exception e;
    Try {
        f();
    Catch(e) {
        fprintf(stderr, "Error: %s\n", e.what);
    }
}
```

La funci�n `f` realiza el procesamiento complejo y captura la
excepci�n `ERROR_NO_MAS_DATOS` porque entiende que es una situaci�n
normal, que no necesita ser tratada de ninguna forma especial.  Sin
embargo en cualquier otro caso relanza la excepci�n para que se trate
m�s arriba en la cadena de llamadas.

La funci�n `g`, que es de mayor nivel de abstracci�n, utiliza `f` pero
en caso de fallo lo notifica al usuario por la salida de error
est�ndar.  F�jate c�mo se reparte la responsabilidad de emitir el
error (e.g. en `procesamiento_complejo`) y de tratarlo en el punto
donde se sabe c�mo tratarlo (`f` o `g`).  No es necesario devolver
c�digos de error en todas las funciones, ni a�adir `if` cuando no se
sabe c�mo actuar con el error.

## Prevenir *leaks*

Las excepciones pueden ser un mecanismo muy efectivo para evitar que
algunos recursos se queden sin liberar (memoria din�mica reservada con
`malloc` que no se libera con `free`, archivos abiertos con `fopen`
que no se cierran con `fclose`, archivos abiertos con `open` que no se
cierran con `close`, etc.).  Por ejemplo, considera esta funci�n para
contar el n�mero de l�neas de un archivo:

```
int contar_lineas(const char* nombre)
{
    FILE* f = fopen(nombre, "r");
    int lineas = 0;
    char buf[128];
    while (NULL != fgets(buf, sizeof(buf), f))
        if (buf[strlen(buf)-1] == '\n')
            ++lineas;
    if (buf[0] != '\0' && buf[0] != '\n')
        ++lineas;
    fclose(f);
    return lineas;
}
```

Esta funci�n cuenta correctamente el n�mero de l�neas de un archivo
contando los caracteres terminador de l�nea `'\n'` que aparecen y
corrigiendo la cuenta si la �ltima l�nea no tiene terminador de l�nea
y no est� vac�a.  Pero �qu� pasa si hay error?

Podemos aprovechar lo que ahora sabemos para notificar errores:

```
int contar_lineas(const char* nombre)
{
    FILE* f = fopen(nombre, "r");
    if (f == NULL)
        Throw Exception(errno, "Error al abrir archivo");
    int lineas = 0;
    char buf[128];
    while (NULL != fgets(buf, sizeof(buf), f))
        if (buf[strlen(buf)-1] == '\n')
            ++lineas;
    if (ferror(f))
        Throw Exception(errno, "Error de lectura");
    if (buf[0] != '\0' && buf[0] != '\n')
        ++lineas;
    fclose(f);
    return lineas;
}
```

El problema es que en caso de error de lectura el archivo `f` no se
cierra nunca.  Nadie llama a `fclose`.  La soluci�n es poner la
excepci�n pero no lanzarla hasta el final.

```
int contar_lineas(const char* nombre)
{
    FILE* f = fopen(nombre, "r");
    if (f == NULL)
        Throw Exception(errno, "Error al abrir archivo");
    int lineas = 0;
    char buf[128];
    while (NULL != fgets(buf, sizeof(buf), f))
        if (buf[strlen(buf)-1] == '\n')
            ++lineas;
    exception e = { -1, NULL };
    if (ferror(f))
        e = Exception(errno, "Error de lectura");
    else if (buf[0] != '\0' && buf[0] != '\n')
        ++lineas;
    fclose(f);
    if (e.error_code >= 0)
        Throw e;
    return lineas;
}
```

Para mayor generalidad, para el caso de que haya funciones anidadas
que usan excepciones, o si el flujo en caso de error no est� claro, es
mejor meter el c�digo en un `Try/Catch`.  Por ejemplo, en el siguiente
fragmento separamos el c�digo en dos funciones, la funci�n
`contar_lineas_en_file` ni siquiera sabe que se tenga que liberar
ning�n recurso.

```
int contar_lineas_en_file(FILE* f)
{
    int lineas = 0;
    char buf[128];
    while (NULL != fgets(buf, sizeof(buf), f))
        if (buf[strlen(buf)-1] == '\n')
            ++lineas;
    if (ferror(f))
        Throw Exception(errno, "Error de lectura");
    if (buf[0] != '\0' && buf[0] != '\n')
        ++lineas;
    return lineas;
}

int contar_lineas(const char* nombre)
{
    int ret = 0;
    exception e = { -1, NULL };
    FILE* f = fopen(nombre, "r");
    if (f == NULL)
        Throw Exception(errno, "Error al abrir archivo");
    Try {
        ret = contar_lineas_en_file(f);
    }
    Catch(e) {
    }
    fclose(f);
    if (e.error_code >= 0)
        Throw e;
    return lineas;
}
```

Esta construcci�n es equivalente a lo que en Java se denomina
*cla�sula `finally`*.  No importa lo que haga el programa dentro del
`Try`, en cualquier caso se llamar� a `close`.

No te quedes en esta breve explicaci�n, mira tranquilamente los
ejemplos que te proporcionamos en el taller, especialmente la
biblioteca `reactor`.  Puede que las siguientes secciones te ayuden
tambi�n a comprender el c�digo.

## Otros contextos de excepci�n

En `reactor` no nos hemos preocupado demasiado del manejo de
excepciones en los hilos que no son el hilo principal.  En la pr�ctica
esta limitaci�n no es tan importante, porque los hilos auxiliares son
habitualmente muy simples.  Cuando la cosa se complica suele usarse un
*proceso* diferente, por ejemplo con un `process_handler`.

Sin embargo hemos habilitado un proceso para permitir el uso de
excepciones en m�ltiples hilos de ejecuci�n cambiando el denominado
*contexto de excepci�n*.  Por ejemplo, imagina que tenemos dos hilos
que realizan un trabajo importante.  Por ejemplo, cada uno con su
reactor.  El hilo principal tiene el contexto de ejecuci�n 0 (cero).
Si queremos tener excepciones en el otro hilo basta asignarle un
contexto de excepci�n diferente al principio del hilo:

```
void* thread(thread_handler* t)
{
    current_exception_context = 1;
    ...
}
```

Es as� de simple, pero hay que tener cuidado de que todos los hilos
con excepciones tengan un `current_exception_context diferente.  El
n�mero de contextos disponible es reducido, pero puede cambiarse
f�cilmente en `reactor/exception.c` editando el valor de la constante
`NUM_EXCEPTION_CONTEXTS` y recompilando.