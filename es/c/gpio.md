[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Entradas y salidas digitales en C

Para la programación de entradas y salidas digitales en C tenemos tres
bibliotecas disponibles: [*wiringPi*](http://wiringpi.com/) de Gordon
Henderson, [*bcm2835*](http://www.airspayce.com/mikem/bcm2835/) de
Mike McCauley y [*pigpio*](http://abyz.co.uk/rpi/pigpio/index.html) de
joan@abyz.co.uk.  Todas ellas son más o menos equivalentes para los
propósitos del taller, aunque cada una tiene sus ventajas e
inconvenientes.  Para empezar te recomiendo que uses
*wiringPi*. Veamos ejemplos con las tres.

## Entradas y salidas digitales con *wiringPi*

Echa un vistazo al
[manual de referencia de `wiringPi`](http://wiringpi.com/reference/) y
muy especialmente a las
[funciones principales](http://wiringpi.com/reference/core-functions/).
Explica qué hace el siguiente programa:

``` C
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
```

Para compilarlo podemos hacer un archivo `makefile` sumamente
sencillo.  Asumiendo que el archivo anterior se llama `test-gpio.c`:

```
LDLIBS=-lwiringPi
test-gpio: test-gpio.o
```

Compílalo usando `make` y demuestra que funciona usando un LED en la
pata J8-12, correspondiente a GPIO 18.  Cuidado con no superar la
corriente de 16 mA.

Si intentas ejecutarlo probablemente obtendrás un mensaje que indica
que el usuario *pi* no tiene suficientes privilegios.  En ese caso hay
que ejecutarlo con `sudo`.  Esto va a ser bastante frecuente en
software que manipula dispositivos físicos. Ningún sistema operativo
serio puede permitir que un usuario normal manipule directamente los
dispositivos, podría comprometer incluso la integridad física del
sistema.

Un resumen de las funciones necesarias de *wiringPi* son las
siguientes:

Función                          | Descripción
---------------------------------|----------------------
`wiringPiSetupGpio()`            | Inicializa la biblioteca con la numeración de pines normal.
`pinMode(pin,OUTPUT)`            | Configura un pin como salida.
`pinMode(pin,INPUT)`             | Configura un pin como entrada.
`digitalWrite(pin,v)`            | Saca un valor por un pin.
`digitalRead(pin)`               | Lee el valor de un pin.
`pullUpDnControl(pin, PUD_DOWN)` | Configura un *pull-down* en un pin.
`pullUpDnControl(pin, PUD_UP)`   | Configura un *pull-up* en un pin.
`delay(msec)`                    | Retardo de un número de milisegundos.


### Uso de PWM

También podemos programar la generación de señales PWM con la ayuda de
la biblioteca *wiringPi*. La frecuencia base para PWM en Raspberry Pi
es de 19.2Mhz.  Esta frecuencia puede ser dividida mediante el uso de
un divisor indicado con `pwmSetClock`, hasta un máximo de 4095.  A
esta frecuencia funciona el algoritmo interno que genera la secuencia
de pulsos, pero en el caso del BCM2835 se dispone de dos modos de
funcionamiento, un modo equilibrado (*balanced*) en el que es difícil
controlar la anchura de los pulsos, pero permite un control PWM de muy
alta frecuencia, y un modo *mark and space* que es mucho más intuitivo
y más apropiado para controlar servos.  El modo *balanced* es
apropiado para controlar la potencia suministrada a la carga o para
transmisión de información.

En el modo *mark and space* el módulo PWM incrementará un contador
interno hasta llegar a un límite configurable, el rango de PWM, que
puede ser de como máximo 1024. Al comienzo del ciclo el pin se pondrá
a 1 lógico, y se mantendrá hasta que el contador interno llegue al
valor puesto por el usuario.  En ese momento el pin se pondrá a 0
lógico y se mantendrá hasta el final del ciclo.

Veamos su aplicación al control de un servomotor. Un servomotor tiene
una entrada de señal para indicar la inclinación deseada. Cada 20ms
espera un pulso y la anchura de este pulso determina la inclinación
del servo.  Alrededor de 1.5ms es la anchura del pulso necesaria para
la posición centrada. Una anchura menor hace girar el servo en sentido
antihorario (hasta 1ms aproximadamente) y una duración mayor lo hace
girar en sentido horario (hasta 2ms aproximadamente).  En este caso
hay que calcular el rango y el divisor para que el pulso se produzca
cada 20ms y el control de la anchura del pulso alrededor de los 1.5ms
sea con la máxima resolución posible.

El montaje es tal como muestra la figura. El cable rojo del servo (V+)
se conecta a +5V en P1-2 o P1-4, el cable negro o marrón (V-) a GND en
P1-6 y el cable amarillo, naranja o blanco (signal) a GPIO18 en
P1-12. No se necesita ningún otro componente.

<figure style="padding:10px">
  <img src="../img/servo-gpio18.svg" width="400"/>

  <figcaption style="font-size:smaller; font-style:italic">
  <div style="width:500px">
  Montaje de un microservo para ser controlado
  directamente desde GPIO18 configurado como salida PWM.
  </div>
  </figcaption>
</figure>


``` C
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
```

Para tener máximo control de la posición del servo probaremos con el
rango máximo, de 1024.  En ese caso el divisor tiene que ser tal que
la frecuencia del pulso PWM sea:

$$
f = \frac{f_{base}}{rango \times div} = \frac{19.2\times 10^6Hz}{1024 \times div} = \frac{1}{20ms} = 50Hz
$$

Es decir, el divisor habría que configurarlo a 390.  El rango completo
del servo depende del modelo concreto.  Teóricamente debería ser entre
52 y 102, siendo el valor completamente centrado 77.  En la práctica
habrá que probar el servo concreto, nuestros experimentos dan un rango
útil entre 29 y 123 para el microservo de TowerPro disponible en el
laboratorio.

Función                          | Descripción
---------------------------------|----------------------
`pinMode(pin,PWM_OUTPUT)`        | Configura un pin como salida PWM.
`pwmSetMode(PWM_MODE_MS)`        | Configura el modo PWM a *mark-and-space*.
`pwmSetClock(div)`               | Configura el divisor del reloj.
`pwmSetRange(rango)`             | Configura el rango del pulso.
`pwmWrite(pin,v)`                | Configura la anchura del pulso.

## Programación de entradas y salidas digitales con *bcm2835*

La biblioteca *bcm2835* es un envoltorio muy fino de las capacidades
del hardware.  Es decir, prácticamente describe en C lo que aparece en
la hoja de datos de Broadcom.  En este sentido es ideal para explorar
la arquitectura y extraer el máximo jugo de tu Raspberry Pi.  Para
tareas no triviales no tendrás más remedio que estudiar bien
{{"12:_bcm28_arm_perip" | cite}} para entender el funcionamiento.

En cuanto al manejo de entradas y salidas digitales la biblioteca
*bcm2835* tiene una interfaz muy similar a *wiringPi* pero soporta
muchas más capacidades del hardware subyacente.  En este taller no
usaremos las características avanzadas.

Una característica interesante de los programas que usan *bcm2835* es
que no requieren ser ejecutados como superusuario si solo acceden a
los pines de GPIO, basta con que el usuario pertenezca al grupo
`gpio`.

``` C
#include <bcm2835.h>

int main() {
    bcm2835_init();
    bcm2835_gpio_fsel(18, BCM2835_GPIO_FSEL_OUTP);
    for(int v = 0;;v = !v) {
        bcm2835_gpio_write(18, v);
        bcm2835_delay(1000);
    }
    bcm2835_close();
}
```

Como puede apreciarse el código es prácticamente equivalente a
*wiringPi*.  La compilación es también similar.  Para compilarlo
podemos hacer un archivo `makefile` casi equivalente al ejemplo de
*wiringPi*.

```
CFLAGS=-I/usr/local/include
LDFLAGS=-L/usr/local/lib
LDLIBS=-lbcm2835
test-gpio: test-gpio.o
```

Al margen del nombre de la biblioteca vemos que es preciso indicar que
busque archivos de cabecera en `/usr/local/include` y busque las
bibliotecas en `/usr/local/lib`.  Esto se debe a que *bcm2835* todavía
no está como un paquete del sistema, y la hemos instalado de forma
manual.

Veamos un resumen de las funciones más importantes.

Función                          | Descripción
---------------------------------|----------------------
`bcm2835_init()`                 | Inicializa la biblioteca.
`bcm2835_close()`                | Libera los recursos empleados por la biblioteca.
`bcm2835_gpio_fsel(pin, BCM2835_GPIO_FSEL_OUTP)` | Configura un pin como salida.
`bcm2835_gpio_fsel(pin, BCM2835_GPIO_FSEL_INPT)` | Configura un pin como entrada.
`bcm2835_gpio_write(pin, v)`     | Saca un valor por un pin.
`bcm2835_gpio_lev(pin)`          | Lee el valor de un pin.
`bcm2835_gpio_set_pud(pin, BCM2835_GPIO_PUD_DOWN)` | Configura un *pull-down* en un pin.
`bcm2835_gpio_set_pud(pin, BCM2835_GPIO_PUD_UP)`   | Configura un *pull-up* en un pin.
`bcm2835_delay(msec)`            | Retardo de un número de milisegundos.
`bcm2835_delayMicroseconds(usec)`| Retardo de un número de microsegundos.

Entre las características avanzadas *bcm2835* implementa la
posibilidad de cambiar el valor de un conjunto de pines de golpe, de
leer un conjunto de pines de golpe, mejor control sobre los eventos de
flanco de subida, bajada o cambio de nivel, etc.

### PWM con *bcm2835*

Al programar la modulación de anchura de pulsos *bcm2835* requiere
conocer un poco del funcionamiento del hardware.  Por ejemplo,
requiere saber que los dos canales PWM no están asociados a un único
pin y que se puede asociar uno u otro canal a algunos de los pines
seleccionando funciones alternativas concretas.  Siguiendo el mismo
ejemplo de *wiringPi* configuraremos el pin GPIO18 como salida PWM.
Para ello tendremos que seleccionar la función alternativa 5 de ese
pin, que corresponde al canal PWM0.  ¿Entiendes ahora la utilidad del
*flyer* que te damos en el taller? A partir de ese momento solo
trabajamos con el canal, no con el pin.

``` C
#include <bcm2835.h>
#include <stdio.h>

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

    bcm2835_init();
    bcm2835_gpio_fsel(18, BCM2835_GPIO_FSEL_ALT5);
    bcm2835_pwm_set_clock(div);
    bcm2835_pwm_set_mode(0, 1, 1);
    bcm2835_pwm_set_range(0, range);

    for(;;) {
        bcm2835_pwm_set_data(0, min);
        bcm2835_delay(1000);
        bcm2835_pwm_set_data(0, max);
        bcm2835_delay(1000);
    }
    bcm2835_close();
    return 0;
}
```

Resumiendo, las siguientes funciones nos permiten trabajar con la modulación PWM:

Función                                   | Descripción
------------------------------------------|----------------------
`bcm2835_gpio_fsel(pin, BCM2835_GPIO_FSEL_ALTn)` | Selecciona la función alternativa *n* del pin. 
`bcm2835_pwm_set_clock(div)`              | Configura el divisor para el reloj de 19.2MHz.
`bcm2835_pwm_set_mode(canal, ms, activo)` | Configura el modo del canal PWM y/o lo activa.
`bcm2835_pwm_set_range(canal, range)`     | Configura el rango del pulso en un canal PWM.
`bcm2835_pwm_set_data(canal, v)`          | Configura la anchura de pulso del canal PWM.


## Programación de entradas y salidas digitales con *pigpio*

La biblioteca *pigpio* está a caballo entre las dos anteriores.  Por
un lado implementa una interfaz de bajo nivel que prácticamente
reproduce la hoja de datos de Broadcom en C.  Por otro lado implementa
capas de abstracción que amplían la funcionalidad considerablemente.
Así por ejemplo añade la posibilidad de simular modulaciópn PWM en
cualquier pin e incorpora capacidades de depuración muy interesantes.

Desde el punto de vista de la programación de entradas y salidas
digitales es muy similar a las otras bibliotecas.

``` C
#include <pigpio.h>

int main() {
    gpioInitialise();
    gpioSetMode(18, PI_OUTPUT);
    for(int v = 0;;v = !v) {
        gpioWrite(18, v);
        gpioDelay(1000000);
    }
    gpioTerminate();
}
```

Prácticamente idéntico a los demás ejemplos salvo por los nombres de
las funciones.  La compilación es muy similar a la de los ejemplos de
*bcm2835* porque la biblioteca *pigpio* tampoco está disponible como
paquete del sistema y hemos tenido que instalarla de forma manual.

```
CFLAGS=-I/usr/local/include -pthread
LDFLAGS=-L/usr/local/lib -pthread
LDLIBS=-lpigpio -lpthread
test-gpio: test-gpio.o
```

Una diferencia importante con respecto a las demás bibliotecas es que
hay que habilitar el uso de hilos y añadir la biblioteca de hilos del
sistema.  Esto es necesario porque muchas de las capacidades añadidas
de *pigpio* se implementan como hilos.


Función                          | Descripción
---------------------------------|----------------------
`gpioInitialise()`               | Inicializa la biblioteca.
`gpioTerminate()`                | Libera los recursos empleados por la biblioteca.
`gpioSetMode(pin, PI_OUTPUT)`    | Configura un pin como salida.
`gpioSetMode(pin, PI_INPUT)`     | Configura un pin como entrada.
`gpioWrite(pin, v)`              | Saca un valor por un pin.
`gpioRead(pin)`                  | Lee el valor de un pin.
`gpioSetPullUpDown(pin, PI_PUD_DOWN)` | Configura un *pull-down* en un pin.
`gpioSetPullUpDown(pin, PI_PUD_UP)`   | Configura un *pull-up* en un pin.
`gpioDelay(usec)`                | Retardo de un número de microsegundos.

Desde el punto de vista de diseño la interfaz *pigpio* está mejor
diseñada que *wiringPi* pero en los ejemplos del taller no lo vas a
notar.  Lo notarás por ejemplo cuando uses `gpioSetAlertFunc` de
*pigpio* en lugar de `wiringPiISR` de *wiringPi*.

### Modulación PWM con *pigpio*

La modulación PWM con *pigpio* puede hacerse con funciones de bajo
nivel de forma equivalente a como se hace en *bcm2835* pero soporta
además una interfaz de alto nivel mucho más sencilla.  Existe una
función para controlar servos con PWM y otra para controlar el
*duty-cycle* y con ello la potencia entregada a una carga.

Veamos el ejemplo usando esta interfaz:

``` C
#include <pigpio.h>
#include <stdio.h>

int main(int argc, char* argv[])
{
    if (argc < 5) {
        printf("Usage: %s min max\n", argv[0]);
        exit(0);
    }

    int min = atoi(argv[1]);
    int max = atoi(argv[2]);

    gpioInitialise();
    for(;;) {
        gpioServo(18, min);
        gpioDelay(1000000);
        gpioServo(18, max)
        gpioDelay(1000000);
    }
    gpioTerminate();
    return 0;
}
```

No es necesario especificar los parámetros de bajo nivel, tan solo la
anchura del pulso en milisegundos.  Un ancho de cero para la señal
PWM.  El resto de valores válidos están entre 500 y 2500, aunque el
rango real del servo depende del modelo concreto.

Otra característica interesante de pigpio es que estas funciones se
pueden aplicar sobre cualquier pin.  Si es uno de los que soportan PWM
por hardware lo utilizará de forma transparente y si no lo simulará
con un hilo independiente.  La frecuencia de la señal PWM que genera
es en todos los casos de 50Hz.

Un resumen de las funciones de alto nivel en pigpio:

Función                     | Descripción
----------------------------|----------------------
`gpioServo(pin, ancho)`     | Señal PWM para controlar servo con una anchura de pulso dada.
`gpioPWM(pin, duty)`        | Genera una señal PWM con un *duty-cycle* determinado (hasta *rango*).
`gpioSetPWMrange(pin,rango)`| Cambia el rango de la señal PWM generada.
`gpioSetPWMfrequency(pin,f)`| Cambia la frecuencia de la señal PWM generada.

## Ejercicios

No intentes usar todas las bibliotecas de golpe.  Elige una y espera a
sentirte cómodo con ella para probar otra. De momento te proponemos
los siguientes ejercicios.

1. Configura y programa el hardware y el software necesario para
  tener dos LEDs parpadeando al mismo ritmo pero manteniendo solo uno
  de ellos encendido a la vez.
  
1. Modifica el ejemplo anterior para que la conmutación se produzca
  solamente cuando se aprieta un pulsador.  Uno de los LEDs estará
  encendido cuando el pulsador no esté apretado, el otro estará
  encendido solamente cuando el pulsador esté apretado.

## Retos para la semana

1. **Moderado** Diseña un mecanismo para poder controlar una matriz de
  LEDs de como mínimo 16x32 con la Raspberry Pi.
