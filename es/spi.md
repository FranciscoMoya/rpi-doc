[//]: # (-*- markdown -*-)

# Uso de SPI
\label{sec:spi}

SPI es un protocolo alternativo a I2C que utilizan algunos
dispositivos. Hoy en día es frecuente encontrar dispositivos que
implementan tanto I2C como SPI y en ese caso probablemente es mejor
decantarse por I2C porque utiliza menos pines.

Al igual que con I2C, el primer paso para realizar programas que
utilizan la interfaz SPI es activarlo con \texttt{raspi-config} en el
apartado \emph{Advanced Configuration}.  Al igual que en el caso de
I2C el programa ofrece la opción de cargar automáticamente los drivers
en el arranque, aunque siempre se puede hacer con la utilidad
\texttt{gpio} de \emph{wiringPi}.

\begin{term}
$ gpio load spi
\end{term}%$

La programación es sencilla en cuanto que solo utiliza dos funciones,
pero puede ser realmente enrevesada de entender la comunicación con
algunos dispositivos SPI.  El motivo es que en SPI para poder leer
datos hay que escribir datos, de hecho se lee a la vez que se escribe.
Esto hace que en los dispositivos reales haya que hacer muchas
transacciones que se descartan por completo.

Este ejemplo escribe en una memoria EEPROM 25LC010A de Microchip.

\begin{lstlisting}[language=C]
#include <stdio.h>
#include <wiringPiSPI.h>
#include "25LC010A.h"

int main (void) {
    wiringPiSPISetup(0, 10000000);

    char enable = _25LC010_WREN;
    wiringPiSPIDataRW (0, &enable, 1);

    char cmd[] = {
      _25LC010_WRITE, 0,
      1, 2, 3, 4, 5
    };
    wiringPiSPIDataRW (0, cmd, sizeof(cmd));

    return 0;
}
\end{lstlisting}

La función \texttt{wiringPiSPISetup} inicializa la comunicación para
el canal 0 a 10Mz. Hay dos canales disponibles en el modelo B+ (0 y
1).

Las llamadas a \texttt{wiringPiSPIDataRW} realizan una transacción SPI
donde se escribe y se lee de manera concurrente un conjunto de bytes.
El significado preciso de lo que se lee y se escribe depende del
dispositivo y en algunos casos puede requerir descartar parte o toda
la información.  En este caso el primer byte es la orden y a
continuación se envían los argumentos.

Pueden consultarse más detalles en el artículo de Gordon Henderson
disponible en
\url{https://projects.drogon.net/understanding-spi-on-the-raspberry-pi/}.

\subsection{Uso de la UART}
\label{sec:uart}

El puerto serie ocupa los pines GPIO14 (TxD) y GPIO15 (RxD). Por
defecto \emph{Raspbian} utiliza estos pines para una consola serie.
Es posible conectar un cable USB a estos pines para conectarse a la
\emph{Raspberry Pi} sin necesidad de ningún tipo de configuración de
red.  El
\href{https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable/}{tutorial
  5 de Adafruit} explica cómo conectar el cable USB y cómo emplear un
software emulador de terminal para hablar con la Raspberry Pi.

Pero claro, eso significa que la UART está ocupada en la consola. Es
necesario quitar la consola para poder emplear la UART en otros fines.
Para desactivar la consola hay que editar el archivo
\texttt{/boot/cmdline.txt}, eliminar el fragmento que dice
\texttt{console=ttyAMA0,115200}, y reiniciar la Raspberry Pi.

La programación de la UART puede realizarse también con \emph{wiringPi}.

\begin{lstlisting}[language=C]
#include <wiringSerial.h>

int main(int argc, char* argv[])
{
    int fd = serialOpen("/dev/ttyAMA0", 19200);
    serialPuts(fd, "START 3\r\n");
    serialFlush(fd);
    serialClose(fd);
    return 0;
}
\end{lstlisting}

Más información sobre las funciones disponibles puede encontrarse en
la \href{http://wiringpi.com/reference/serial-library/}{página de
  documentación de referencia para la biblioteca de puerto serie de
  wiringPi}.
