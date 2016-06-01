[//]: # (-*- mode: markdown -*-)

# Programación 

El primer paso para utilizar I2C es activarlo en la secciÃ³n
*Advanced Options* de `raspi-config`.  No estÃ¡ activado
por defecto para no perder los pines GPIO2 y GPIO3.  La propia
utilidad `raspi-config` puede activar tambiÃ©n la carga
automÃ¡tica del driver en el arranque, aunque siempre se puede hacer
con la utilidad `gpio` de *wiringPi*.

```
$ gpio load i2c
$ gpio i2cdetect
```

La segunda orden (`i2cdetect`) permite conocer quÃ© dispositivos
hay conectados y con ello sus direcciones en el bus I2C.

A continuaciÃ³n ya podemos usar las funciones especÃ­ficas de I2C que
estÃ¡n disponibles en el archivo de cabecera `wiringPiI2C.h`.
La programaciÃ³n precisa de un dispositivo I2C depende mucho del
fabricante.  En general necesitaremos primero una configuraciÃ³n y
luego entraremos en un bucle que lee o escribe datos.  Por ejemplo,
veamos cÃ³mo se maneja un acelerÃ³metro ADXL335.

``` C
#include <wiringPiI2C.h>
#include <stdio.h>
#include "adxl345.h"

int main(int argc, char* argv[])
{
    int fd = wiringPiI2CSetup(0x53);
    wiringPiI2CWriteReg8(fd, POWER_CTL, POWER_CTL__Measure);
    wiringPiI2CWriteReg8(fd, DATA_FORMAT, DATA_FORMAT__FULL_RES);
    wiringPiI2CWriteReg8(fd, BW_Rate, BW_RATE__Rate_200);


    for(;;) {
        short ax = wiringPiI2CReadReg16(fd, DATAX),
        short ay = wiringPiI2CReadReg16(fd, DATAY),
        short az = wiringPiI2CReadReg16(fd, DATAZ)
        printf("%d\t%d\t%d\n", ax, ay, az);
    }

    close(fd);
}
```

La funciÃ³n de inicializaciÃ³n `wiringPiI2CSetup` necesita la
direcciÃ³n del dispositivo, que podemos obtener con `i2cdetect`
y devuelve un descriptor de archivo que hay que usar en todas las
demÃ¡s llamadas relacionadas con ese dispositivo.  Podemos hacer
transferencias de lectura y escritura de/a registros de 8 y 16 bits.
TÃ­picamente se configura previamente el dispositivo con llamadas a
`wiringPiI2CWriteReg8` segÃºn la hoja de datos del fabricante.

## MPU6050

En el kit de iniciaciÃ³n que entregamos en el taller hemos incluido el
[InvenSense MPU6050](http://www.invensense.com/products/motion-tracking/6-axis/mpu-6050/),
un dispositivo que combina un acelerÃ³metro de tres ejes con un
girÃ³scopo de tres ejes y con un *digital motion processor* que
descarga al procesador principal de una parte importante del
procesamiento.  Puede incluso combinarse con un magnetÃ³metro I2C de
tres ejes para tener una soluciÃ³n completa de seguimiento de posiciÃ³n
de 9 ejes.

La hoja de datos del MPU 6050 estÃ¡ dividida en dos partes.  La
[primera parte](http://43zrtwysvxb2gf29r5o0athu.wpengine.netdna-cdn.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf)
describe las caracterÃ­sticas elÃ©ctricas, descripciÃ³n funcional, y
esquemas de aplicaciÃ³n tÃ­pica.  Lo mÃ¡s interesante es que se trata de
un dispositivo I2C de 3.3V y la frecuencia del reloj mÃ¡xima debe ser
de 400KHz.

La
[segunda parte](http://43zrtwysvxb2gf29r5o0athu.wpengine.netdna-cdn.com/wp-content/uploads/2015/02/MPU-6000-Register-Map1.pdf)
contiene una descripciÃ³n de los registros internos y cÃ³mo se pueden
leer o escribir para obtener los datos del girÃ³scopo y del
acelerÃ³metro.
