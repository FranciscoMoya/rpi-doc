[//]: # (-*- mode: markdown -*-)

# Uso I2C

El primer paso para utilizar I2C es activarlo en la sección
*Advanced Options* de `raspi-config`.  No está activado
por defecto para no perder los pines GPIO2 y GPIO3.  La propia
utilidad `raspi-config` puede activar también la carga
automática del driver en el arranque, aunque siempre se puede hacer
con la utilidad `gpio` de *wiringPi*.

```
$ gpio load i2c
$ gpio i2cdetect
```

La segunda orden (`i2cdetect`) permite conocer qué dispositivos
hay conectados y con ello sus direcciones en el bus I2C.

A continuación ya podemos usar las funciones específicas de I2C que
están disponibles en el archivo de cabecera `wiringPiI2C.h`.
La programación precisa de un dispositivo I2C depende mucho del
fabricante.  En general necesitaremos primero una configuración y
luego entraremos en un bucle que lee o escribe datos.  Por ejemplo,
veamos cómo se maneja un acelerómetro ADXL335.

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

La función de inicialización `wiringPiI2CSetup` necesita la
dirección del dispositivo, que podemos obtener con `i2cdetect`
y devuelve un descriptor de archivo que hay que usar en todas las
demás llamadas relacionadas con ese dispositivo.  Podemos hacer
transferencias de lectura y escritura de/a registros de 8 y 16 bits.
Típicamente se configura previamente el dispositivo con llamadas a
`wiringPiI2CWriteReg8` según la hoja de datos del fabricante.

## MPU6050

En el kit de iniciación que entregamos en el taller hemos incluido el
[InvenSense MPU6050](http://www.invensense.com/products/motion-tracking/6-axis/mpu-6050/),
un dispositivo que combina un acelerómetro de tres ejes con un
giróscopo de tres ejes y con un *digital motion processor* que
descarga al procesador principal de una parte importante del
procesamiento.  Puede incluso combinarse con un magnetómetro I2C de
tres ejes para tener una solución completa de seguimiento de posición
de 9 ejes.

La hoja de datos del MPU 6050 está dividida en dos partes.  La
[primera parte](http://43zrtwysvxb2gf29r5o0athu.wpengine.netdna-cdn.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf)
describe las características eléctricas, descripción funcional, y
esquemas de aplicación típica.  Lo más interesante es que se trata de
un dispositivo I2C de 3.3V y la frecuencia del reloj máxima debe ser
de 400KHz.

La
[segunda parte](http://43zrtwysvxb2gf29r5o0athu.wpengine.netdna-cdn.com/wp-content/uploads/2015/02/MPU-6000-Register-Map1.pdf)
contiene una descripción de los registros internos y cómo se pueden
leer o escribir para obtener los datos del giróscopo y del
acelerómetro.
