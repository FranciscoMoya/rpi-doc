[//]: # (-*- markdown; coding:utf-8 -*-)

#Uso de la UART


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
