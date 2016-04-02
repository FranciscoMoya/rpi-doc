# Manejo de errores

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

Lo correcto hoy en día es emplear un mecanismo soportado por todos los
lenguajes de programación modernos, que se denomina *excepciones*.
Pero C no tiene excepciones. Afortunadamente hay un conjunto de
bibliotecas que implementan el mecanismo utilizando macros del
preprocesador y dos funciones de la biblioteca estándar de C que no
suelen ser muy conocidas, `setjmp` y `longjmp`.

No es propósito de este taller explicar el funcionamiento de estas
funciones.  Si tienes curiosidad mira la página de manual para
entender su funcionamiento.  Son esenciales para implementar
corrutinas, o cualquier otra forma de interrumpir el flujo normal de
ejecución de un programa.

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
El tipo de datos de `ex` es definible por el usuario.

En nuestros ejemplos utilizaremos `reactor/exception.h`, una
biblioteca auxiliar en la que definimos la excepción como una
estructura similar a esta:

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
exception ex;
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

## Anidamiento de `Try/Catch`

En algunos casos se sabe cómo manejar algunos errores, pero no todos.
En esos casos puede ser útil capturar la posible excepción y volverla
a lanzar si no se sabe qué hacer con ella.  Por ejemplo:

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

La función `f` realiza el procesamiento complejo y captura la
excepción `ERROR_NO_MAS_DATOS` porque entiende que es una situación
normal, que no necesita ser tratada de ninguna forma especial.  Sin
embargo en cualquier otro caso relanza la excepción para que se trate
más arriba en la cadena de llamadas.

La función `g`, que es de mayor nivel de abstracción, utiliza `f` pero
en caso de fallo lo notifica al usuario por la salida de error
estándar.  Fíjate cómo se reparte la responsabilidad de emitir el
error (e.g. en `procesamiento_complejo`) y de tratarlo en el punto
donde se sabe cómo tratarlo (`f` o `g`).  No es necesario devolver
códigos de error en todas las funciones, ni añadir `if` cuando no se
sabe cómo actuar con el error.

## Prevenir *leaks*

Las excepciones pueden ser un mecanismo muy efectivo para evitar que
algunos recursos se queden sin liberar (memoria dinámica reservada con
`malloc` que no se libera con `free`, archivos abiertos con `fopen`
que no se cierran con `fclose`, archivos abiertos con `open` que no se
cierran con `close`, etc.).  Por ejemplo, considera esta función para
contar el número de líneas de un archivo:

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

Esta función cuenta correctamente el número de líneas de un archivo
contando los caracteres terminador de línea `'\n'` que aparecen y
corrigiendo la cuenta si la última línea no tiene terminador de línea
y no está vacía.  Pero ¿qué pasa si hay error?

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
cierra nunca.  Nadie llama a `fclose`.  La solución es poner la
excepción pero no lanzarla hasta el final.

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
que usan excepciones, o si el flujo en caso de error no está claro, es
mejor meter el código en un `Try/Catch`.  Por ejemplo, en el siguiente
fragmento separamos el código en dos funciones, la función
`contar_lineas_en_file` ni siquiera sabe que se tenga que liberar
ningún recurso.

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

Esta construcción es equivalente a lo que en Java se denomina
*claúsula `finally`*.  No importa lo que haga el programa dentro del
`Try`, en cualquier caso se llamará a `close`.

No te quedes en esta breve explicación, mira tranquilamente los
ejemplos que te proporcionamos en el taller, especialmente la
biblioteca `reactor`.  Puede que las siguientes secciones te ayuden
también a comprender el código.

## Otros contextos de excepción

En `reactor` no nos hemos preocupado demasiado del manejo de
excepciones en los hilos que no son el hilo principal.  En la práctica
esta limitación no es tan importante, porque los hilos auxiliares son
habitualmente muy simples.  Cuando la cosa se complica suele usarse un
*proceso* diferente, por ejemplo con un `process_handler`.

Sin embargo hemos habilitado un proceso para permitir el uso de
excepciones en múltiples hilos de ejecución cambiando el denominado
*contexto de excepción*.  Por ejemplo, imagina que tenemos dos hilos
que realizan un trabajo importante.  Por ejemplo, cada uno con su
reactor.  El hilo principal tiene el contexto de ejecución 0 (cero).
Si queremos tener excepciones en el otro hilo basta asignarle un
contexto de excepción diferente al principio del hilo:

```
void* thread(thread_handler* t)
{
    current_exception_context = 1;
    ...
}
```

Es así de simple, pero hay que tener cuidado de que todos los hilos
con excepciones tengan un `current_exception_context diferente.  El
número de contextos disponible es reducido, pero puede cambiarse
fácilmente en `reactor/exception.c` editando el valor de la constante
`NUM_EXCEPTION_CONTEXTS` y recompilando.