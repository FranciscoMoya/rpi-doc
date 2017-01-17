[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Tratamiento de errores

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

``` C
Try {
    /* Código de usuario, sin ´manejo de errores */ 
}
Catch (ex) {
    /* Código para tratar el error notificado en ex */
}
```

En el momento en que se puede producir una situación excepcional o un
error debe notificarse de esta forma:

``` C
if (funcion_tradicional() < 0)
   Throw ex;
```

Donde `ex` es una excepción, una estructura de datos que recoge toda
la información del error para ser manejada cuando esto sea posible.
El tipo de datos de `ex` es definible por el usuario.

En nuestros ejemplos utilizaremos `reactor/exception.h`, una
biblioteca auxiliar en la que definimos la excepción como una
estructura similar a esta:

``` C
typedef struct {
    int error_code;
    const char* what;
} exception;

define_exception_type(exception);
```

Así que si necesitas una descripción textual del error puedes usar el
campo `what` de la estructura.  Por ejemplo:

``` C
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

``` C
f = fopen(fname, "r");
if (f == NULL)
    Throw Exception(errno, "")
```

Cuando se ejecuta la sentencia `Throw` el programa pasa el control a
la claúsula `Catch` del último `Try` abierto, aunque esté en otra
función.  Es un mecanismo muy poderoso para desacoplar la lógica del
programa y la lógica de manejo de errores.

## Anidamiento de `Try/Catch`

En algunos casos se sabe cómo manejar algunos errores, pero no todos.
En esos casos puede ser útil capturar la posible excepción y volverla
a lanzar si no se sabe qué hacer con ella.  Por ejemplo:

``` C
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

``` C
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

``` C
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

``` C
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

``` C
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

## Excepciones no capturadas

En una situación normal si una excepción de *cexcept* no es capturada
con una sentencia `Try/Catch` se produciría un fallo de segmentación.
En `reactor` hemos proporcionado un mecanismo para que las excepciones
no capturadas desemboquen en un mensaje de información sobre el error
y un volcado de la traza de llamada.  Por ejemplo:

``` C
Uncaught exception (error_code 111) at socket_handler.c:167
Error en connect: Connection refused
Current call trace (last 5):
./rpi-src/c/reactor/test/test_connector(connector_init+0x28)[0x12d1c]
./rpi-src/c/reactor/test/test_connector(connector_new+0x3c)[0x12cd4]
./rpi-src/c/reactor/test/test_connector(main__+0x5c)[0x11b0c]
./rpi-src/c/reactor/test/test_connector(main+0xa0)[0x13870]
/lib/arm-linux-gnueabihf/libc.so.6(__libc_start_main+0x114)[0x76d4b294]
```

Informa de que la excepción se ha producido en la línea 167 del
archivo `socket_handler.c`.  Para mayor información se notifica la
lista de llamadas no terminadas en el momento de la excepción.  En
este ejemplo la excepción se produjo en la función `connector_init`
que fue llamada desde `connector_new` y ésta desde `main__`.  Nota que
el nombre de `main` aparece algo extraño.  Eso es por el mecanismo
incorporado para tratar las excepciones no capturadas.

## Uso en tus programas

Para usar excepciones en tus programas solo tienes que tener en cuenta
algunos detalles.

* Incluye el archivo de cabecera `reactor/exception.h`.
* Compila con las opciones `-fasynchronous-unwind-tables` y `-pthread`.
* Añade al montador las opciones `-pthread` y `-rdynamic`.

Realmente solo el primer punto es estrictamente necesario, pero los
demás consejos permiten tener información completa de la traza de
llamadas, una gran ayuda para depurar.

## Simplificar con excepciones

Separar el flujo de control del programa del flujo de control de
errores tiene una consecuencia clara a la hora de escribir programas.
Se fortalecen las abstracciones y es más fácil escribirlos.  Veamos un
ejemplo sencillo para ver esto.

Una función muy frecuente en los programas de Raspberry Pi es `write`.
Se trata de una llamada al sistema operativo que escribe un conjunto
de datos en un archivo identificado por un descriptor de archivo.  Ese
archivo puede representar un puerto serie, un canal de comunicaciones
WiFi con otra Raspberry Pi, o incluso una salida digital.  La esencia
de Unix y todos sus derivados e imitaciones es que todo en el sistema
se vea como un archivo desde el punto de vista del sistema operativo.

``` C
ssize_t write(int fd, const void* buf, size_t size);
```

La llamada al sistema `write` devuelve un valor que normalmente
corresponde al número de bytes escritos.  En general no tiene por qué
coincidir con el número de bytes totales y no por ello implica error.
Para poder mandar todo hay que volver a llamar a `write` con el resto.
Por tanto este uso es típico:

``` C
int escribir_datos(int fd, const tipo_datos* datos)
{
    const void* buf = datos;
    int size = sizeof(tipo_datos);
    while (size > 0) {
        int n = write(fd, buf, size);
        if (n < 0)
            return -1;
        buf += n;
        size -= n;
    }
    return 0;
}
```

Es innecesariamente largo debido a que `write` no devuelve los bytes
escritos, sino que tiene dos posibles significados.  Con excepciones
esto no pasa, vamos a envolver `write` en una función más amigable:

``` C
int write_ex(int fd, const void* buf, size_t size)
{
    if (0 > write(fd, buf, size))
        Throw Exception(errno, "write error");
}
```

No ha sido muy complicado pero hemos eliminado la fuente de la
confusión.  Ahora si devuelve algo es el número de bytes escritos, no
devuelve ningún error.  Si hay errores se trata aparte, en el
`Try/Catch` correspondiente.  Ahora se puede simplificar
considerablemente:

``` C
void escribir_datos(int fd, const tipo_datos* datos)
{
    const void *buf = datos, *end = buf + sizeof(tipo_datos);
    while ((buf += write_ex(fd, buf, end - buf)) < end)
        ;
}
```

No hace falta devolver otro código de error, si hay excepción ya se
propagará adecuadamente.

No te quedes en esta breve explicación, mira tranquilamente los
ejemplos que te proporcionamos en el taller, especialmente la
biblioteca `reactor`.  Aunque las excepciones te parezcan algo
completamente nuevo empieza a usarlas desde ya.  Tu código ganará
mucho en legibilidad y en robustez, aunque no tengas muy claro cómo
funciona.


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

``` C
void* thread(thread_handler* t)
{
    current_exception_context = 1;
    ...
}
```

Es así de simple, pero hay que tener cuidado de que todos los hilos
con excepciones tengan un `current_exception_context` diferente.  El
número de contextos disponible es reducido, pero puede cambiarse
fácilmente en `reactor/exception.c` editando el valor de la constante
`NUM_EXCEPTION_CONTEXTS` y recompilando.
