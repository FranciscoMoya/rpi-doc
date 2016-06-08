[//]: # (-*- mode: markdown; coding: utf-8 -*-)
# Comunicaciones en red

Tanto los modelos B+ como la 2B y la 3B incluyen interfaz Ethernet.
La Raspberry Pi 3 modelo B que utilizamos en este taller incluye WiFi
pero cualquiera de las otras puede tener también WiFi por un precio de
unos 4€ empleando una interfaz WiFi USB.  Por tanto cualquier proyecto
de Raspberry Pi debe plantearse la posibilidad de comunicar datos a
través de una red TCP/IP.

Para programar en red en GNU/Linux, como en la mayoría de los sistemas
operativos modernos, se utiliza una interfaz de programación
denominada *socket API*.  Se trata de un conjunto de funciones
diseñadas para que la programación de redes se parezca mucho a la
entrada/salida con archivos.

En el capítulo 2 ya hemos introducido la interfaz *socket*.  En C se
incluye esta interfaz en la biblioteca del sistema `libc` que se
incorpora automáticamente.  La forma de usar esta biblioteca es
prácticamente lo que hemos visto en el capítulo 2.

## Comunicaciones UDP

El siguiente ejemplo muestra el servidor UDP más simple posible.

``` C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <assert.h>
#include <string.h>
#include <netdb.h>
#include <arpa/inet.h>

static int udp_server_socket(const char* service);

int main() {
    int fd = udp_server_socket("9999");
    for(;;) {
	char buf[1024];
	int n = read(fd, buf, sizeof(buf));
	assert(n >= 0);
	if (n == 0) break;
	buf[n]='\0';
	printf("%s", buf);
    }
    close(fd);
    return 0;
}
```

La función `udp_server_socket` simplemente crea un nuevo *socket* UDP
con los parámetros indicados y llama a *bind* para fijar el puerto
donde escucha mensajes.

``` C
static struct sockaddr_in ip_address(const char* host, 
				     const char* service, 
				     const char* proto);

static int udp_server_socket(const char* service)
{
    struct sockaddr_in sin = ip_address("0.0.0.0", service, "udp");
    struct protoent* pe = getprotobyname("udp");
    assert(pe != NULL);
    int fd = socket(PF_INET, SOCK_DGRAM, pe->p_proto);
    assert (fd >= 0);
    assert (bind(fd, (struct sockaddr*)&sin, sizeof(sin)) >= 0);
    return fd;
}
```

La función `ip_address` construye una estructura que representa una
dirección completa de un servicio IP (host, puerto y protocolo).

``` C
static struct sockaddr_in ip_address(const char* host, 
                                     const char* service, 
                                     const char* proto)
{
    struct sockaddr_in sin;
    memset(&sin, 0, sizeof(sin));
    sin.sin_family = AF_INET;
    struct hostent* he = gethostbyname(host);
    if (he != NULL)
        memcpy(&sin.sin_addr, he->h_addr_list[0], he->h_length);
    else
        sin.sin_addr.s_addr = inet_addr(host);
    struct servent* se = getservbyname(service, proto);
    sin.sin_port = (se != NULL? se->s_port : htons(atoi(service)));
    return sin;
}
```

Mete estas tres funciones en un archivo y compílalo.  Prueba que
funciona usando un cliente *netcat*.

El cliente correspondiente en C es similar.

``` C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <assert.h>
#include <string.h>
#include <netdb.h>
#include <arpa/inet.h>

static int udp_client_socket(const char* host, const char* service);

int main() {
    int fd = udp_client_socket("localhost", "9999");
    for(;;) {
        char buf[1024];
        fgets(buf, sizeof(buf), stdin);
        int n = write(fd, buf, strlen(buf));
        assert(n >= 0);
        if (n == 0) break;
    }
    close(fd);
    return 0;
}
```

La función `udp_client_socket` es similar a `udp_server_socket` pero
en lugar de llamar a *bind* llama a *connect* para fijar la dirección
destino de los mensajes.

``` C
static struct sockaddr_in ip_address(const char* host, 
				     const char* service, 
				     const char* proto);

static int udp_client_socket(const char* host, const char* service)
{
    struct sockaddr_in sin = ip_address(host, service, "udp");
    struct protoent* pe = getprotobyname("udp");
    assert(pe != NULL);
    int fd = socket(PF_INET, SOCK_DGRAM, pe->p_proto);
    assert (fd >= 0);
    assert (connect(fd, (struct sockaddr*)&sin, sizeof(sin)) >= 0);
    return fd;
}
```

La función `ip_address` ya la comentamos en el servidor.  Construye el
cliente y prueba que funciona correctamente con *netcat*.  Después haz
la prueba de integración con servidor y cliente.

## Comunicaciones TCP

Desde el punto de vista de la programación la diferencia fundamental
con el caso anterior radica en la necesidad de establecer conexiones
para realizar una comunicación.  Esto complica algo el servidor.

``` Python
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <assert.h>
#include <string.h>
#include <netdb.h>
#include <arpa/inet.h>

static int tcp_master_socket(const char* service);

int main() {
    int master = tcp_master_socket("9999");
    for(;;) {
        int fd = accept(master, (struct sockaddr*) NULL, NULL);
        assert (fd >= 0);
        for(;;) {
            char buf[1024];
            int n = read(fd, buf, sizeof(buf));
            assert(n >= 0);
            if (n == 0) break;
            buf[n]='\0';
            printf("%s", buf);
        }
        close(fd);
    }
    close(master);
    return 0;
}
```

Ahora el socket maestro simplemente se usa para crear nuevos sockets
esclavos cuando ocurre una conexión con *accept*.  El esclavo se
utiliza para manejar los datos relativos a esa conexión.

La función `tcp_master_socket` simplemente crea el socket TCP y llama
a *bind* y a *listen*.

``` C
static struct sockaddr_in ip_address(const char* host, 
				     const char* service, 
				     const char* proto);

static int tcp_master_socket(const char* service)
{
    struct sockaddr_in sin = ip_address("0.0.0.0", service, "tcp");
    struct protoent* pe = getprotobyname("tcp");
    assert(pe != NULL);

    int fd = socket(PF_INET, SOCK_STREAM, pe->p_proto);
    assert (fd >= 0);
    assert(bind(fd, (struct sockaddr*)&sin, sizeof(sin)) >= 0);
    assert(listen(fd, 10) >= 0);
    return fd;
}
```

Al igual que antes *bind* asigna la dirección y puerto en los que
escucha, pero ahora tenemos que configurar el *backlog* con *listen*
(cuántas conexiones se retienen sin aceptar) y aceptar nuevas
conexiones con *accept*.

En realidad esto es una sobre-simplificación.  Una vez aceptada la
conexión se puede atender en un hilo independiente y aceptar nuevas
conexiones en el hilo principal inmediatamente.

El cliente es prácticamente idéntico al de UDP.

``` C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <assert.h>
#include <string.h>
#include <netdb.h>
#include <arpa/inet.h>

static int tcp_client_socket(const char* host, const char* service);

int main() {
    int fd = tcp_client_socket("localhost", "9999");
    for(;;) {
        char buf[1024];
        fgets(buf, sizeof(buf), stdin);
        int n = write(fd, buf, strlen(buf));
        assert(n >= 0);
        if (n == 0) break;
    }
    close(fd);
    return 0;
}
```

Y la función `tcp_client_socket` es prácticamente igual que su
homóloga en UDP.

``` C
static struct sockaddr_in ip_address(const char* host,
                                     const char* service,
				     const char* proto);

static int tcp_client_socket(const char* host, const char* service)
{
    struct sockaddr_in sin = ip_address(host, service, "tcp");
    struct protoent* pe = getprotobyname("tcp");
    assert(pe != NULL);
    int fd = socket(PF_INET, SOCK_STREAM, pe->p_proto);
    assert (fd >= 0);
    assert (connect(fd, (struct sockaddr*)&sin, sizeof(sin)) >= 0);
    return fd;
}
```
