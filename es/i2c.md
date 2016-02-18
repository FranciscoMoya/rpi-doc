[//]: # (-*- markdown -*-)

# Uso I2C
\label{sec:i2c}

El primer paso para utilizar I2C es activarlo en la sección
\emph{Advanced Options} de \texttt{raspi-config}.  No está activado
por defecto para no perder los pines GPIO2 y GPIO3.  La propia
utilidad \texttt{raspi-config} puede activar también la carga
automática del driver en el arranque, aunque siempre se puede hacer
con la utilidad \texttt{gpio} de \emph{wiringPi}.

\begin{term}
$ gpio load i2c
$ gpio i2cdetect
\end{term}

La segunda orden (\texttt{i2cdetect}) permite conocer qué dispositivos
hay conectados y con ello sus direcciones en el bus I2C.

A continuación ya podemos usar las funciones específicas de I2C que
están disponibles en el archivo de cabecera \texttt{wiringPiI2C.h}.
La programación precisa de un dispositivo I2C depende mucho del
fabricante.  En general necesitaremos primero una configuración y
luego entraremos en un bucle que lee o escribe datos.  Por ejemplo,
veamos cómo se maneja un acelerómetro ADXL335.

\begin{lstlisting}[language=C]
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
\end{lstlisting}

La función de inicialización \texttt{wiringPiI2CSetup} necesita la
dirección del dispositivo, que podemos obtener con \texttt{i2cdetect}
y devuelve un descriptor de archivo que hay que usar en todas las
demás llamadas relacionadas con ese dispositivo.  Podemos hacer
transferencias de lectura y escritura de/a registros de 8 y 16 bits.
Típicamente se configura previamente el dispositivo con llamadas a
\texttt{wiringPiI2CWriteReg8} según la hoja de datos del fabricante.

En el kit de iniciación que entregamos en el taller no hemos incluido
dispositivos I2C, pero disponemos de algunos para hacer pequeñas
pruebas.  Hay varios tipos de acelerómetro, incluso alguno con
giróscopo y brújula, sensor de temperatura, \ldots
