[//]: # (-*- markdown -*-)

# GPIO (*General Purpose Input/Output*)

La Raspberry Pi cuenta con un número de pines que pueden configurarse
como entradas o salidas digitales para controlar cualquier periférico,
sensor o actuador externo.  Se trata de los pines de *GPIO*. En este
capítulo trataremos su uso desde una perspectiva general y en
capítulos posteriores nos centraremos en proyectos concretos que los
aprovechan de diferentes formas. También cubriremos algunas placas de
extensión que ofrecen mayor protección a los pines de *GPIO*.

<figure style="padding:10px">
  <iframe src="https://docs.google.com/spreadsheets/d/1PgLL0qUC8KiJsmHOVumi-X9tgjcEINyay8LSjMT4br0/pubhtml?gid=0&amp;single=true&amp;headers=false&amp;range=A1:N21&amp;chrome=false&amp;gridlines=false" frameborder="0" style="width:100%;height:460px"></iframe>

  <figcaption style="font-size:smaller; font-style:italic">
  <div style="width:600px">
  Comparación de los pines de E/S para los modelos originales y los
  modelos A+ y B+. Consulta <a
  href="https://docs.google.com/spreadsheets/d/1PgLL0qUC8KiJsmHOVumi-X9tgjcEINyay8LSjMT4br0/edit?usp=sharing">la
  hoja de cálculo original</a> para ver la función de los pines
  especiales.
  </div>
  </figcaption>
</figure>


<figure style="float:right; padding:10px">
  <img src="img/rpi-b2-p1p5.svg" width="200"/>

  <figcaption style="font-size:smaller; font-style:italic">
  <div style="width:200px">
  Configuración de los pines de E/S en los zócalos P1 y P5
  de los modelos A y B revisión 2.
  </div>
  </figcaption>
</figure>

## Conector de *GPIO*

Las Raspberry Pi modelos A y B cuentan con un conector de 2x13 pines
etiquetado como P1.  Todos los modelos posteriores son compatibles con
estos primeros 26 pines.  Éste es el único pensado realmente para
proporcionar entradas y salidas digitales de propósito general.  El
conector P1 proporciona acceso a 17 pines de *GPIO*.  En la revisión 2
de la Raspberry Pi modelo B, junto a P1 se encuentra un zócalo
despoblado, el P5 de 2x4 pines, que proporciona acceso a 4 pines de
*GPIO* adicionales.

Los modelos A+ y B+ amplían considerablemente el número de pines
disponibles del conector P1 de 26 pines a un nuevo conector J8 de 40
pines.  Entre ellos 9 pines más de GPIO.  Sin embargo la
compatibilidad es total, puesto que los 26 primeros pines mantienen su
función original.

En lo sucesivo utilizaremos la numeración correspondiente al nuevo
conector de 40 pines.  Su equivalente en los conectores P1 y P5 puede
verse en la tabla al inicio de este capítulo.

Los pines de J8 tienen fundamentalmente cinco funciones:

* Por un lado la mayoría de ellos son entradas/salidas digitales de
  propósito general. Pueden configurarse como entradas o salidas,
  pueden leerse o pueden escribirse con un valor digital, alto o bajo,
  uno o cero.  Ten presente que el nivel alto es de 3.3V y no son
  tolerantes a tensiones de 5V.

* Los pines 8 y 10 pueden configurarse como interfaz UART para un
  puerto serie convencional. De hecho ésta es su configuración por
  defecto en Raspbian, ya que la UART se usa como consola.

* Por otro lado los pines 3 y 5, se pueden configurar como interfaz
  I2C para interactuar con periféricos que siguen este protocolo.  En
  el taller ya lo hemos configurado de este modo.

* El pin 12 puede configurarse como salida PWM.  En teoría los pines
  12 y 13 pueden configurarse también como interfaz I2S (audio
  digital) pero hacen falta pines que no están disponibles fácilmente.

* Por último los pines 19, 21, 23, 24 y 26 se pueden configurar como
  interfaz SPI para interactuar con periféricos que siguen este
  protocolo.  En el taller ya los hemos configurado de este modo.

> **Warning**

> El zócalo P5 está originalmente pensado para poblarlo desde la capa
> inferior del circuito. Los pines de la figura adjunta están
> consignados según este criterio. Si se monta en la capa superior la
> asignación de pines será su reflejo especular.

En total disponemos de 17 pines para entradas y salidas digitales y de
ellos uno solo para control PWM.  En este capítulo veremos las
funciones de manipulación de estos pines de GPIO, incluyendo el
control PWM.

## Protección de *GPIO*

Cuando se utilizan los pines de *GPIO* para interfaz con hardware de
cualquier tipo hay que poner mucho cuidado para no dañar la propia
Raspberry Pi.  Es muy importante comprobar los niveles de tensión y la
corriente solicitada.  Los pines de GPIO pueden generar y consumir
tensiones compatibles con los circuitos de 3.3V (no son tolerantes a
5V) y
[pueden sacar hasta 16mA siempre que toda la corriente drenada por los pines de GPIO no supere los 51mA](http://www.thebox.myzen.co.uk/Raspberry/Understanding_Outputs.html).

> **Warning**

> Los pines GPIO de la Raspberry Pi no son tolerantes a tensiones de
> 5V. Están pensados para su utilización con circuitos de 3.3V y no
> tienen ningún tipo de protección. No debes drenar más de 16mA por
> pin y hasta un total de 51mA.

Para evitar problemas se han fabricado una amplia variedad de tarjetas
de expansión, que protegen de diversas formas los pines de GPIO.  Las
más conocidas son

* [Pi-Face Digital](http://www.piface.org.uk/products/piface_digital/)

* [Gertboard](http://www.element14.com/community/docs/DOC-51726/l/assembled-gertboard-for-raspberry-pi)
  de Gert van Loo, uno de los primeros voluntarios de la Raspberry Pi
  Foundation.

Una amplia variedad de métodos de protección está disponible en el
tutorial de `elinux.org` titulado
[GPIO Protection Circuits](http://elinux.org/RPi_Tutorial_EGHS:GPIO_Protection_Circuits).

## GPIO con `wiringPi`

La referencia definitiva para programar cualquiera de los periféricos
del BCM2835 es
\href{http://www.raspberrypi.org/wp-content/uploads/2012/02/BCM2835-ARM-Peripherals.pdf}{la
  hoja de datos del fabricante}, aunque se trata de un documento
denso, árido y poco claro.

Para empezar probablemente el mejor tutorial es el de
\url{http://elinux.org}, que lleva por título
\href{http://elinux.org/RPi_Tutorial_Easy_GPIO_Hardware_&_Software}{\emph{RPi
    Tutorial: Easy GPIO Hardware \& Software}} y especialmente los
\href{http://elinux.org/RPi_Low-level_peripherals#Code_examples}{ejemplos
  de programación utilizando diversos lenguajes y mecanismos}.  El
\href{http://www.element14.com/community/servlet/JiveServlet/previewBody/51727-102-1-265829/Gertboard_UM_with_python.pdf}{manual
  de la Gertboard} también tiene abundante información pero el
software es difícil de encontrar.

Para los intereses de este taller los dos procedimientos más
relevantes son el uso de la biblioteca \emph{wiringPi} y el uso de la
interfaz del sistema \emph{sysfs}.  El primero tiene dependencias que
hay que instalar y configurar.  El segundo no tiene dependencias, pero
hay que programar más.

La biblioteca \emph{wiringPi} está pensada para ser compilada en la
propia Raspberry Pi y ya la hemos instalado como un paquete más del
sistema.

## Compilación manual de wiringPi

\warning{Para el taller ya debes haber instalado la biblioteca
  \emph{wiringPi}.  Puedes saltarte esta sección.}

La biblioteca \emph{wiringPi} está ya disponible en el sistema y se
trata de una versión razonablemente moderna.  Sin embargo la evolución
de \emph{wiringPi} es con frecuencia más rápida que la actualización
de los paquetes de \emph{Raspbian}.  Por este motivo conviene explicar
el proceso de instalación manual.  Nos vendrá bien si la última
versión incorpora soporte para dispositivos que necesitemos emplear.

La verdad es que \emph{wiringPi} no es un buen ejemplo de programación
empotrada.  Su programador no se ha molestado en permitir su
compilación cruzada.  Por tanto vamos a trabajar con la Raspberry Pi
directamente para construir esta biblioteca. Entra en tu Raspberry Pi
con un terminal SSH, por ejemplo PuTTY y ejecuta las siguientes
instrucciones:

``` console
$ git clone --depth 1 git://git.drogon.net/wiringPi
$ cd wiringPi
```

\info{GIT es el sistema de control de versiones distribuido más
  eficiente de la actualidad. Fue inicialmente creado por los
  desarrolladores del kernel Linux y hoy en día es utilizado por
  \href{http://github.com}{GitHub}, o
  \href{http://bitbucket.org}{BitBucket}.  Nuestro Raspbian por defecto
  viene con \emph{git} instalado, pero si no lo estuviera bastaría ejecutar
  \texttt{sudo apt-get install git-core}.}

Esto se trae la última versión del repositorio GIT de \emph{wiringPi}
en el directorio \texttt{wiringPi}.  A continuación solo tenemos que
compilar. En este caso el programador sigue un convenio muy atípico,
utiliza un pequeño script.


\begin{term}
$ ./build
\end{term} %$

### Programando salidas digitales
\label{sec:prog-gpio}

Echa un vistazo al
\href{http://wiringpi.com/reference/}{manual de referencia de
  \emph{wiringPi}} y muy especialmente a las
\href{http://wiringpi.com/reference/core-functions/}{funciones
  principales}.  Explica qué hace el siguiente programa:

\begin{lstlisting}[language=C]
#include <wiringPi.h>
#include <unistd.h>

int main() {
    wiringPiSetupGpio();

    pinMode(18,OUTPUT);
    for(int v = 0;;v = !v) {
        digitalWrite(18,v);
        delay(1000);
    }
}
\end{lstlisting}

Compílalo en la Raspberry Pi y demuestra que funciona usando un LED en la pata J8-12, la
correspondiente a GPIO 18. Si intentas ejecutarlo verás que no funciona, porque el usuario
\emph{pi} no tiene suficientes privilegios. Hay que ejecutarlo con \texttt{sudo}. Esto va
a ser bastante frecuente en software que manipula dispositivos físicos. Ningún sistema
operativo serio puede permitir que un usuario normal manipule directamente los
dispositivos, podría comprometer incluso la integridad física del sistema.

Esto es especialmente problemático en el momento en que intentamos depurar el programa con
Eclipse. Eclipse ejecuta automáticamente el programa por nosotros y no utiliza
\texttt{sudo}.  Es más, ni siquiera se ejecuta automáticamente, sino a través de una
utilidad del depurador llamada \texttt{gdbserver}. Es decir, para poder depurar un
programa y que éste se ejecute con permisos de superusuario, hay que ejecutar
\texttt{gdbserver} con permiso de superusuario.  Pero Eclipse carece de opciones para
poder hacer esto.  La única alternativa que nos queda es hacer nuestro propio
\texttt{gdbserver} y cambiar la ruta en Eclipse para que utilice nuestra versión. Veamos
esto con un pequeño ejemplo.

Edita un archivo de texto llamado \texttt{/usr/local/bin/gdbserver-root} en la Raspberry
Pi utilizando el editor \texttt{nano} y después otorga permiso de ejecución para todos.

\begin{term}
$ sudo nano /usr/local/bin/gdbserver-root
$ sudo chmod a+x /usr/local/bin/gdbserver-root
\end{term}

Como puede verse utilizamos \texttt{sudo} para editar el archivo. Esto se debe a que el
usuario \emph{pi} no puede crear archivos en el directorio \texttt{/usr/local/bin} y por
eso editamos como administrador. El contenido del archivo debe ser el siguiente:

\begin{lstlisting}[language=]
#!/bin/sh
exec sudo gdbserver "$@"
\end{lstlisting}%$

\imagenmargen{img/dennis-ritchie.png}{fig:shebang}{La pareja de caracteres \texttt{\#!} se
  denomina \emph{shebang} como contracción de \emph{SHArp BANG} (los nombres de los
  caracteres en inglés vulgar). Este convenio para identificar los \emph{scripts} se debe
  a Dennis Ritchie, inventor del lenguaje C y uno de los creadores de Unix. }

La primera línea lo identifica como un \emph{shell script} que interpreta el programa
\texttt{/bin/sh}. Este programa es la Bourne shell, el intérprete de órdenes estándar en
todos los Unix y variantes. Cuando entramos con SSH en una Raspberry Pi también estamos
utilizando la Bourne shell, de forma interactiva.

La segunda línea utiliza \texttt{exec}, que termina el intérprete de órdenes ejecutando lo
que viene a continuación.  Si no apareciera \texttt{exec} el programa funcionaría igual
pero sería algo más ineficiente porque el intérprete de órdenes no terminaría hasta que
terminara el programa.  A continuación usamos \texttt{sudo} para ejecutar con privilegios
de administrador \texttt{gdbserver}, y por último pasamos a \texttt{gdbserver} los mismos
parámetros que le hayan llegado a nuestro propio programa. La sintaxis \verb+"$@"+ sirve
para preservar correctamente los argumentos con espacios, si los hubiera.%$

Con esta nueva versión de \emph{gdbserver} podemos depurar igual que como se describió en
la sección~\ref{sec:debug}.  El único cambio necesario es que hay que especificar el nuevo
\emph{gdbserver} en la subpestaña \emph{Gdbserver Settings} de la pestaña \emph{Debugger}
cuando se define la configuración de depuración, tal y como muestra la
figura~\ref{fig:eclipse-gdbserver-root}.

\imagenhere{img/eclipse-gdbserver-root.png}{12cm}{Configuración de
  los ajustes de \emph{gdbserver}.}{fig:eclipse-gdbserver-root}

### Programando con \emph{wiringPi} en Eclipse
\label{sec:wiringpi-eclipse}

El programa anterior vamos a hacerlo empleando el entorno Eclipse en un PC
convencional. Para ello hay que tener en cuenta que tanto la biblioteca \emph{wiringPi}
como sus archivos de cabecera están ahora en la Raspberry Pi.  Esto es una constante en el
desarrollo de sistemas empotrados.  Habrá bibliotecas y componentes suministrados por el
fabricante o por terceros que se instalan directamente sobre el sistema empotrado.  Pero
como desarrolladores tenemos que ser capaces de ponerlos a disposición de nuestras
herramientas.

La biblioteca \emph{wiringPi} y todos sus componentes se instalan durante el proceso de
compilación en el directorio correspondient de \texttt{/usr/local} en la Raspberry Pi. Es
decir, la biblioteca \texttt{libwiringPi.so} estará disponible en \texttt{/usr/local/lib}
y las cabeceras (tales como \texttt{wiringPi.h}) estarán disponibles en
\texttt{/usr/local/include}.  Para darlos a conocer al entorno Eclipse vamos a proceder
como se indicó en la sección~\ref{sec:detalles}.

En primer lugar actualizaremos la copia de las bibliotecas de la Raspberry Pi empleando la
utilidad \emph{UpdateSysRoot}. El uso de la herramienta es muy intuitivo. Basta indicar en
la configuración de la conexión la dirección IP de la Raspberry Pi o su nombre completo
(e.g. \texttt{rpiXX.local}) si tienes soporte de Bonjour.

Una vez actualizado la biblioteca y los archivos de cabecera quedarán en los siguientes
directorios:

\begin{term}
C:\RPi\tools\sysprogs\arm-linux-gnueabihf\sysroot\usr\local\include
C:\RPi\tools\sysprogs\arm-linux-gnueabihf\sysroot\usr\local\lib
\end{term}

Cada uno de ellos hay que añadirlo a la sección correspondiente de la configuración del
proyecto. Con el botón derecho del ratón sobre el proyecto a configurar seleccionamos
\emph{Properties} para abrir el diálogo de configuración.  En la sección \emph{C/C++
  General} y dentro de ésta en la subsección \emph{Paths and Symbols} se dispone de todas
las pestañas para la configuración de la nueva biblioteca. La pestaña \emph{Includes} se
configura como muestra la figura~ref{fig:wiringpi-include}, añadiendo el directorio donde
están los nuevos archivos de cabecera.

\imagenhere{img/wiringpi-include.png}{11cm}{Añadir la ruta de los archivos de cabecera al
  proyecto.}{fig:wiringpi-include}

A continuación añadimos la nueva ruta de las bibliotecas dinámicas en la pestaña
\emph{Library Paths} como muestra la figura~\ref{fig:wiringpi-ldflags}.

\imagenhere{img/wiringpi-ldflags.png}{11cm}{Añadir la ruta de las bibliotecas al
  proyecto.}{fig:wiringpi-ldflags}

Por último añadimos la biblioteca (basta indicar \emph{wiringPi} en la pestaña
\emph{Libraries} como muestra la figura~\ref{fig:wiringpi-ldlibs}.

\imagenhere{img/wiringpi-ldlibs.png}{11cm}{Añadir la biblioteca \emph{wiringPi} al
  proyecto.}{fig:wiringpi-ldlibs}

### Uso de PWM
\label{sec:pwm}

\emph{Pulse Width Modulation} (PWM) es una técnica que consiste en la variación del
\emph{duty cycle} de una señal digital periódica, fundamentalmente con dos posibles
objetivos:
\begin{itemize}
\item Por un lado se puede utilizar como mecanismo para transmitir información.  Por
  ejemplo, los servo-motores tienen una entrada digital por la que se transmite el ángulo
  deseado codificado en PWM.
\item Por otro lado se puede utilizar para regular la cantidad de potencia suministrada a
  la carga.  Por ejemplo, las luminarias LED frecuentemente utilizan reguladores PWM para
  permitir el control de intensidad.
\end{itemize}

La Raspberry Pi tiene una sola pata de GPIO (GPIO 18) que puede configurarse como salida
PWM. El propio BCM2835 se encarga de gestionar la generación de la señal, liberando
completamente al procesador principal. El usuario puede configurar el rango de valores
disponible (hasta 1024) y posteriormente sacar un valor determinado, de forma que el
módulo PWM se encarga de mantener el \emph{duty cycle} en la relación valor/rango.

También podemos programar la generación de señales PWM con la ayuda de la biblioteca
\emph{wiringPi}. La frecuencia base para PWM en Raspberry Pi es de 19.2Mhz. Esta
frecuencia puede ser dividida mediante el uso de un divisor indicado con
\texttt{pwmSetClock}, hasta un máximo de 4095.  A esta frecuencia funciona el algoritmo
interno que genera la secuencia de pulsos, pero en el caso del BCM2835 se dispone de dos
modos de funcionamiento, un modo equilibrado (\emph{balanced}) en el que es difícil
controlar la anchura de los pulsos, pero permite un control PWM de muy alta frecuencia, y
un modo \emph{mark and space} que es mucho más intuitivo y más apropiado para controlar
servos.  El modo \emph{balanced} es apropiado para controlar la potencia suministrada a la
carga o para transmisión de información.

En el modo \emph{mark and space} el módulo PWM incrementará un contador interno hasta
llegar a un límite configurable, el rango de PWM, que puede ser de como máximo 1024. Al
comienzo del ciclo el pin se pondrá a 1 lógico, y se mantendrá hasta que el contador
interno llegue al valor puesto por el usuario.  En ese momento el pin se pondrá a 0 lógico
y se mantendrá hasta el final del ciclo.

Veamos su aplicación al control de un servomotor. Un servomotor tiene una entrada de señal
para indicar la inclinación deseada. Cada 20ms espera un pulso y la anchura de este pulso
determina la inclinación del servo.  Alrededor de 1.5ms es la anchura del pulso necesaria
para la posición centrada. Una anchura menor hace girar el servo en sentido antihorario
(hasta 1ms aproximadamente) y una duración mayor lo hace girar en sentido horario (hasta
2ms aproximadamente).  En este caso hay que calcular el rango y el divisor para que el
pulso se produzca cada 20ms y el control de la anchura del pulso alrededor de los 1.5ms
sea con la máxima resolución posible.

El montaje es tal como muestra la figura~\ref{fig:servo-gpio18}. El cable rojo del servo
(V+) se conecta a +5V en P1-2 o P1-4, el cable negro o marrón (V-) a GND en P1-6 y el
cable amarillo, naranja o blanco (signal) a GPIO18 en P1-12. No se necesita ningún otro
componente.

\imagenhere{img/servo-gpio18.pdf}{11cm}{Montaje de un microservo para ser controlado
  directamente desde GPIO 18 configurado como salida PWM.}{fig:servo-gpio18}

\begin{lstlisting}[language=C]
#include <wiringPi.h>
#include <stdlib.h>

int main(int argc, char* argv[])
{
  if (argc < 5) {
    printf("Usage: %s divisor rango min max\n", argv[0]);
    exit(0);
  }

  int div = atoi(argv[1]);
  int range = atoi(argv[2]);
  int min = atoi(argv[3]);
  int max = atoi(argv[4]);

  wiringPiSetupGpio();

  pinMode(18,PWM_OUTPUT);
  pwmSetMode(PWM_MODE_MS);
  pwmSetClock(div);
  pwmSetRange(range);

  for(;;) {
    pwmWrite(18,min);
    delay(1000);
    pwmWrite(18,max);
    delay(1000);
  }
}
\end{lstlisting}

Para tener máximo control de la posición del servo probaremos con el rango máximo, de
1024.  En ese caso el divisor tiene que ser tal que la frecuencia del pulso PWM sea:
\[
f = \frac{f_{base}}{rango \times div} = \frac{19.2\times 10^6Hz}{1024 \times div} = \frac{1}{20ms} = 50Hz
\]

Es decir, el divisor habría que configurarlo a 390.  El rango completo
del servo depende del modelo concreto.  Teóricamente debería ser entre
52 y 102, siendo el valor completamente centrado 77.  En la práctica
habrá que probar el servo concreto, nuestros experimentos dan un rango
útil entre 29 y 123 para el microservo de TowerPro disponible en el
laboratorio.

\section{Ejercicios}
\label{sec:gpio-ej}

\begin{enumerate}
\item Configura y programa el hardware y el software necesario para
  tener dos LEDs parpadeando al mismo ritmo pero manteniendo solo uno
  de ellos encendido a la vez.
\item Modifica el ejemplo anterior para que la conmutación se produzca
  solamente cuando se aprieta un pulsador.  Uno de los LEDs estará
  encendido cuando el pulsador no esté apretado, el otro estará
  encendido solamente cuando el pulsador esté apretado.
\end{enumerate}

\section{Retos para la semana}
\label{sec:gpio-retos}

\begin{enumerate}
\item[\textbf{Fácil}] Compila la biblioteca \emph{wiringPi} como un proyecto Eclipse y
  configura tus ejemplos para que usen este proyecto como dependencia.
\item[\textbf{Fácil}] Diseña un mecanismo para poder controlar tiras
  de LEDs empleando la interfaz SPI.
\item[\textbf{Fácil}] Diseña un mecanismo para poder controlar una matriz de LEDs de como
  mínimo 16x32 con la Raspberry Pi.
\end{enumerate}



## GPIO Zero
