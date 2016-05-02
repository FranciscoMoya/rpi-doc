[//]: # (-*- markdown; coding:utf-8 -*-)

#Uso de la UART

El puerto serie ocupa los pines GPIO14 (TxD) y GPIO15 (RxD). Por
defecto \emph{Raspbian} utiliza estos pines para una consola serie.
Es posible conectar un cable USB a estos pines para conectarse a la
\emph{Raspberry Pi} sin necesidad de ningún tipo de configuración de
red.  El
[tutorial
  5 de Adafruit](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable/) explica cómo conectar el cable USB y cómo emplear un
software emulador de terminal para hablar con la Raspberry Pi.

Pero claro, eso significa que la UART está ocupada en la consola. Es
necesario quitar la consola para poder emplear la UART en otros fines.
Para desactivar la consola hay que editar el archivo
`/boot/cmdline.txt`, eliminar el fragmento que dice
`console=ttyAMA0,115200`, y reiniciar la Raspberry Pi.

La programación de la UART puede realizarse también con *wiringPi*.

``` C
#include <wiringSerial.h>

int main(int argc, char* argv[])
{
    int fd = serialOpen("/dev/ttyAMA0", 19200);
    serialPuts(fd, "START 3\r\n");
    serialFlush(fd);
    serialClose(fd);
    return 0;
}
```

Más información sobre las funciones disponibles puede encontrarse en
la [página de
  documentación de referencia para la biblioteca de puerto serie de
  wiringPi](http://wiringpi.com/reference/serial-library/).
