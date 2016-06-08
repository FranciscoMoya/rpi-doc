[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Programación de la interfaz I2C

La programación precisa de un dispositivo I2C depende mucho del
fabricante.  En general necesitaremos primero una configuración y
luego entraremos en un bucle que lee o escribe datos.  Por ejemplo,
veamos cómo se maneja el acelerómetro MPU6050.


## Programación I2C con *wiringPi*

Como siempre *wiringPi* sacrifica flexibilidad en aras de una mayor
simplicidad.  Ni siquiera proporciona una función para cambiar la
frecuencia del reloj y no soporta transferencias de bloques, sino solo
de registros de 8 y 16 bits.

``` C
#include <wiringPiI2C.h>
#include <stdio.h>
#include <stdlib.h>

short swap(short n) { return (n << 8) | (n >> 8) & 0xff; }

int main()
{
    int fd = wiringPiI2CSetup(0x68);
    if (0x68 != wiringPiI2CReadReg8(fd, 117)) {
        puts("No se encuentra el dispositivo");
        return 1;
    }
    wiringPiI2CWriteReg8(fd, 107, 0);
    for(;;) {
        short t = wiringPiI2CReadReg16(fd, 65);
        printf("temperatura: %f\n", swap(t)/340. + 36.53);
        
        short ax = wiringPiI2CReadReg16(fd, 59);
        short ay = wiringPiI2CReadReg16(fd, 61);
        short az = wiringPiI2CReadReg16(fd, 63);
        printf("ax=%d, ay=%d, az=%d\n", ax, ay, az);
        delay(500);
    }
    close(fd);
    return 0;
}
```

La función de inicialización `wiringPiI2CSetup` necesita la
dirección del dispositivo, que podemos obtener con `i2cdetect`
y devuelve un descriptor de archivo que hay que usar en todas las
demás llamadas relacionadas con ese dispositivo.  Podemos hacer
transferencias de lectura y escritura de/a registros de 8 y 16 bits.
Típicamente se configura previamente el dispositivo con llamadas a
`wiringPiI2CWriteReg8` según la hoja de datos del fabricante.

Un resumen de las funciones empleadas sería:

Función                             | Descripción
------------------------------------|----------------------
`wiringPiI2CSetup(dir)`             | Inicializa un canal I2C.
`wiringPiI2CWriteReg8(fd, reg, v)`  | Escribe un registro de 8 bits.
`wiringPiI2CWriteReg16(fd, reg, v)` | Escribe un registro de 16 bits.
`wiringPiI2CReadReg8(fd, reg)`      | Lee un registro de 8 bits.
`wiringPiI2CReadReg16(fd, reg)`     | Lee un registro de 16 bits.
`close(fd)`                         | Cierra el canal.

## Programación de I2C con *bcm2835*

En *bcm2835* la interfaz es también sencilla pero no se requiere un
descriptor de archivo para cada dispositivo I2C y la comunicación es
un poco engorrosa, porque el direccionamiento de los registros debe
hacerse de forma manual con un write.


``` C
#include <bcm2835.h>
#include <stdio.h>

short swap(short n) { return (n << 8) | (n >> 8) & 0xff; }

int main()
{
    bcm2835_init();
    bcm2835_i2c_begin();
    bcm2835_i2c_setSlaveAddress(0x68);
    bcm2835_i2c_set_baudrate(100000);
    char dir = 117;
    bcm2835_i2c_write(&dir, 1);
    bcm2835_i2c_read(&dir, 1);
    if (0x68 != dir) {
        puts("No se encuentra el dispositivo");
        return 1;
    }
    char cfg[] = {107, 0};
    bcm2835_i2c_write(cfg, 2);
    for(;;) {
        dir = 59;
        bcm2835_i2c_write(&dir, 1);
        short data[7];
        bcm2835_i2c_read(data, 14);
        printf("temperatura: %f\n", swap(data[3])/340. + 36.53);
        printf("ax=%d, ay=%d, az=%d\n",
               swap(data[0]), swap(data[1]), swap(data[2]));
        delay(500);
    }
    bcm2835_i2c_end();
    bcm2835_close();
    return 0;
}
```

Un resumen de las funciones utilizadas.

Función                             | Descripción
------------------------------------|----------------------
`bcm2835_i2c_begin()`               | Inicializa las comunicaciones I2C.
`bcm2835_i2c_end()`                 | Libera los recursos ocupados por *begin*.
`bcm2835_i2c_setSlaveAddress(addr)` | Asigna la dirección destino.
`bcm2835_i2c_set_baudrate(rate)`    | Configura la velocidad del reloj I2C.
`bcm2835_i2c_write(buf, len)`       | Escribe un conjunto de bytes en ráfaga.
`bcm2835_i2c_read(buf, len)`        | Lee un conjunto de bytes en ráfaga.

En realidad la biblioteca soporta alguna otra función para tratar con
casos especiales.  Si te enfrentas a un módulo I2C nuevo consulta la
documentación de *bcm2835* por si te facilita las cosas.

## Programación de I2C con *pigpio*

La biblioteca *pigpio* implementa una interfaz que combina las dos
aproximaciones anteriores.  Puede leer o escribir bytes o palabras,
pero también bloques, lo que simplifica sensiblemente el programa.
Además permite acceder a los dos buses I2C.

``` C
#include <pigpio.h>
#include <stdio.h>

short swap(short n) { return (n << 8) | (n >> 8) & 0xff; }

int main()
{
    gpioInitialise();
    int i2c = i2cOpen(1, 0x68, 0);
    if (0x68 != i2cReadByteData(i2c, 117)) {
        puts("No se encuentra el dispositivo");
        return 1;
    }
    i2cWriteByteData(i2c, 107, 0);
    for(;;) {
        short data[7];
        i2cReadI2CBlockData(i2c, 59, (char*)data, 14);
        printf("temperatura: %f\n", swap(data[3])/340. + 36.53);
        printf("ax=%d, ay=%d, az=%d\n",
               swap(data[0]), swap(data[1]), swap(data[2]));
        delay(500);
    }
    i2cClose(i2c)
    gpioTerminate();
    return 0;
}
```

Soporta muchas más funciones, pero habitualmente no necesitaremos más de estas.

Función                             | Descripción
------------------------------------|----------------------
`i2cOpen(bus, dir, 0)`              | Abre un canal I2C para el dispositivo *dir*.
`i2cClose(h)`                       | Cierra el canal asociado a *h*.
`i2cWriteByteData(h, reg, v)`       | Escribe un byte *v* en un registro *reg*.
`i2cWriteWordData(h, reg, v)`       | Escribe un short *v* en un registro *reg*.
`i2cReadByteData(h, reg)`           | Lee un byte de un registro *reg*.
`i2cReadWordData(h, reg)`           | Lee un short de un registro *reg*.
`i2cWriteI2CBlockData(h, reg, buf, len)` | Escribe un bloque a partir de *reg*.
`i2cReadI2CBlockData(h, reg, buf, len)`  | Lee un bloque a partir de *reg*.

Como en el resto de módulos *pigpio* es la más completa de las tres, y
*bcm2835* la más cercana al hardware y por tanto la más pequeña.
Nuestra recomendación personal es que utilices *wiringPi* para
empezar, por su mayor simplicidad.
