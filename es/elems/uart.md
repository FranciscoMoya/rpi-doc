[//]: # (-*- mode: markdown; coding: utf-8 -*-)
# Comunicación con UART

UART son las siglas de *Universal Asynchronous Receiver Transmiter*.
Se trata de la interfaz de comunicación serie dominante en los
microcontroladores de gama baja y en los ordenandores antiguos.  Antes
de USB la UART era el mecanismo para conectar el teclado, el ratón o
incluso el modem de comunicaciones.  Antes aún era la interfaz
empleada por los terminales o consolas para interactuar con un
ordenador.  El programa Terminal que ejecutas en tu Raspberry Pi no es
sino una emulación de estos terminales antiguos.

Hoy en día es casi imposible encontrar una de estas interfaces serie
en un ordenador, y prácticamente ha sido reemplazada por USB.
Afortunadamente tenemos cables USB especiales que incorporan un
adaptador UART-USB.  En el kit del alumno de este taller tienes uno.
Este adaptador te permite comunicar la UART de la Raspberry Pi con un
puerto USB cualquiera, ya sea de un ordenador o de otra Raspberry Pi.
Esto permite utilizar la Raspberry Pi sin necesidad de disponer de un
monitor y sin necesidad de tener conexión de red ni teclado USB. Por
ejemplo, desde otra Raspberry Pi o desde un portátil.

El puerto serie ocupa los pines GPIO14 (TxD) y GPIO15 (RxD). Por
defecto *Raspbian* utiliza estos pines para una consola serie.  El SoC
de Broadcom soporta dos UARTs pero ambas se configuran en los mismos
pines de la cabecera J8. UART0 es un periférico estandar de ARM
(PL011) e implementa todas las capacidades esperables en una UART de
alto rendimiento.  UART1 implementa una versión simplificada que se
denomina mini-UART por el fabricante.  Está pensada para ser usada con
dispositivos de baja tasa de transferencia, como la consola.

En todos los modelos de Raspberry Pi salvo en el Compute Module solo
disponemos de los pines GPIO14 y GPIO15 para ambas UART.  Por este
motivo *Raspbian* utiliza UART0 para la consola y no se suele emplear
UART1.

Pero claro, eso significa que la UART está ocupada en la consola. Es
necesario quitar la consola para poder emplear la UART en otros fines.
Para desactivar la consola hay que editar el archivo
`/boot/cmdline.txt` como superusuario, eliminar el fragmento que dice
`console=serial0,115200`, y reiniciar la Raspberry Pi.

Para poder utilizar los pines GPIO14 y GPIO15 como pines de
entrada/salida digital utiliza la herramienta de configuración y en la
pestaña *Interfaces* desactiva la opción *Serial*.

Pero volvamos a su uso como consola, que es muy útil cuando quieras
poner tu Raspberry Pi en un robot u otro tipo de equipos donde
enchufar un monitor es impensable.  Es posible conectar un cable USB a
estos pines para conectarse a la Raspberry Pi sin necesidad de ningún
tipo de configuración de red.  En el kit del alumno habrás recibido un
cable USB en un extremo y con cuatro conectores Dupont hembra en el
otro.  Puedes conectarlo de la siquiente manera:

* El cable rojo es la alimentación del USB (5V).  La Raspberry Pi ya
  tiene su propia alimentación, así que este cable **no debes
  conectarlo**.
* El cable negro es la masa, conéctalo al pin J8-6.
* El cable blanco es el de transmisión de datos desde la UART al
  puerto USB. Conéctalo al pin J8-8 (GPIO14).
* El cable verde es el de recepción de datos desde el puerto USB hacia
  la UART. Conéctalo al pin J8-10 (GPIO15).

Con esto debería ser suficiente para tener la consola funcionando en
cualquier modelo de Raspberry Pi anterior a la 3B. El problema es que
en la Raspberry Pi 3 la UART se utiliza para la interfaz Bluetooth y
la consola se configura con la mini-UART.  Esto tiene importantes
consecuencias que no están del todo resueltas aún.  Como usuario no
vas a enfrentarte en absoluto a los problemas que plantea, pero ten
presente que para que la mini-UART funcione correctamente hay que
fijar la frecuencia del core a 250 MHz, no es posible modular la
frecuencia para ahorrar consumo.

Probar la consola es muy sencillo, utilizando la propia Raspberry Pi.
Conecta también el extremo USB del cable a un puerto USB libre y
ejecuta:

```
pi@raspberrypi:~ $ miniterm.py /dev/ttyUSB0 115200
Raspbian GNU/Linux 8 raspberrypi ttyS0

raspberrypi login:
```

Entra como usuario `pi` y contraseña `raspberry`.  Como ves dispones
de una consola completamente funcional en la UART.  Puedes conectarte
a ella con cualquier portátil y usar la Raspberry Pi sin necesidad de
un monitor y un teclado.  Para equipos móviles es una gran ventaja.
Para salir de `miniterm.py` presiona la tecla *Ctrl*, *AltGr* y *]*.

Queda fuera de los objetivos del taller explicar cómo usar la consola
desde un portátil con Windows.  Si dispones de un GNU/Linux en tu
portátil el sistema es el mismo que hemos explicado.  Instala la
herramienta `screen` y úsala igual que hemos usado `miniterm.py`.  En
ese caso se sale con *Ctrl-A* seguido de *\*.
