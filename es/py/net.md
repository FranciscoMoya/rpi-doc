[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Comunicaciones en red

En el capítulo 2 ya hemos introducido la interfaz *socket*.  En Python
se incluye esta interfaz en la biblioteca `socket`.  La forma de usar
esta biblioteca es prácticamente lo que hemos visto en el capítulo 2,
con la única diferencia de que se usa `send` en lugar de `write` y
`recv` en lugar de `read`.

## Comunicaciones UDP

El siguiente ejemplo muestra el servidor UDP más simple posible.

``` Python
from socket import *

s = socket(AF_INET, SOCK_DGRAM)
s.bind(('0.0.0.0', 8888))
while True:
    data = s.recv(1024)
    if not data:
        break
    print(data)
```

Al construir un *socket* Python devuelve un objeto que implementa toda
la *API socket*.  En UDP lo único que tenemos que hacer es asignar una
dirección y puerto de escucha y a partir de ese momento empezamos a
recibir o enviar mensajes indicando un tamaño de buffer.  Recuerda que
en Python se lee con `recv` (o `recvfrom`).

El cliente correspondiente es aún más simple.

``` Python
from socket import *

s = socket(AF_INET, SOCK_DGRAM)
s.connect(('localhost', 8888))
while True:
    s.send(input().encode('utf-8'))
```

Para establecer la dirección destino usamos `connect` y desde ese
momento ya podemos enviar mensajes con `send`.

## Comunicaciones TCP

Desde el punto de vista de la programación la diferencia fundamental
con el caso anterior radica en la necesidad de establecer conexiones
para realizar una comunicación.  Esto complica algo el servidor.

``` Python
from socket import *

s = socket(AF_INET, SOCK_STREAM)
s.bind(('0.0.0.0', 8888))
s.listen(10)

con, addr = s.accept()
while True:
    data = con.recv(1024)
    if not data:
        break
    print(data)
con.close()
```

Al igual que antes asignamos la dirección y puerto en los que escucha,
pero ahora tenemos que configurar el *backlog* con `listen` (cuántas
conexiones se retienen sin aceptar) y aceptar nuevas conexiones con
`accept`.  El método `accept` devuelve otro socket que usamos para el
diálogo con ese cliente conectado.

En realidad esto es una sobre-simplificación.  Una vez aceptada la
conexión se puede atender en un hilo independiente y aceptar nuevas
conexiones en el hilo principal inmediatamente.

El cliente es prácticamente idéntico al de UDP.

``` Python
from socket import *

s = socket(AF_INET, SOCK_STREAM)
s.connect(('localhost', 8888))
while True:
    s.send(input().encode('utf-8'))
```
