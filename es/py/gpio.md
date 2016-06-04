[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Entradas y salidas digitales

Para programar entradas y salidas digitales en Python tenemos también
una amplia variedad de bibliotecas:

* La biblioteca *GPIO Zero* proporciona una interfaz abstracta muy
  sencilla de usar para una amplia variedad de periféricos.  Es
  especialmente cómoda para el uso de entradas y salidas digitales.
* La biblioteca *RPi.GPIO* es un módulo que proporciona acceso
  exclusivamente a las entradas y salidas digitales.  Actualmente no
  soporta SPI, I2C, hardware PWM o comunicaciones serie, aunque está
  previsto que lo soporte en el futuro.
* La biblioteca *wiringPi2* no es más que un envoltorio de la
  biblioteca C del mismo nombre.  Por tanto gran parte de lo que
  decimos en el capítulo anterior sobre *wiringPi* es también
  aplicable a Python.
* La biblioteca *pigpio* es un envoltorio de la interfaz a *pigpiod*
  en C.  El programa *pigpiod* ya hemos tenido ocasión de utilizarlo
  en el segundo capítulo, a través de su interfaz de línea de órdenes
  `pigs`.  El módulo Python ofrece unas características similares y
  permite una conexión remota a *pigpiod*.  Eso significa que podemos
  ejecutar nuestro programa en un ordenador convencional y que éste se
  comunique con la Raspberry Pi para realizar la interacción con el
  mundo físico.

En este capítulo describiremos exclusivamente la primera, aunque
animamos a experimentar con otras alternativas una vez que se conozcan
los fundamentos.

 *GPIO Zero* fue desarrollada fundamentalmente por Ben Nuttal de la
Raspberry Pi Foundation y Dave Jones, el creador de la biblioteca
*picamera*.  Sigue fielmente el propósito de la Fundación, impulsar la
docencia de la electrónica y la informática en edades tempranas.  Por
tanto es extremadamente fácil de usar.

## Luces y botones

Un aspecto importante de *GPIO Zero* es que cambia el foco del
programador hacia el sistema.  No se piensa en términos de los
periféricos del BCM2835, sino en términos de los componentes del
sistema.  Esto hace que su uso sea tremendamente intuitivo.

``` Python
from gpiozero import LED, Button
from signal import pause

led = LED(17)
button = Button(3)

button.when_pressed = led.on
button.when_released = led.off

pause()
```

Practicamente se explica solo.  Utiliza una interfaz declarativa, en
la que simplemente se describe lo que tiene que pasar cuando suceden
eventos en los componentes del sistema.  No se indica expícitamente
qué patas son salidas o entradas, pero lo hace de forma automática al
indicar que en GPIO17 hemos puesto un LED y en GPIO3 un pulsador.  No
se indica explícitamente que se active un *pull-up*, pero lo hace por
el mero hecho de decir que en la pata GPIO3 hemos puesto un pulsador.
Podemos cambiar el *pull-up* por un *pull-down* añadiendo un parámetro
en el constructor `button = Button(3, pull_up = False)`.

La línea `button.when_pressed = led.on` indica que cuando ocurra el
evento *pressed* sobre el botón se invoque a la función `led.on()`.
Es así de simple.  Evidentemente por debajo hay un hilo que está
consultando el estado de la entrada digital, pero se oculta
completamente al usuario.

Es muy frecuente que una vez declarado todo lo que debe ocurrir ya no
tengamos nada más que hacer.  Si no hacemos nada más el programa
terminaría.  Así que no nos queda más remedio que quedarnos en algún
tipo de bucle que no haga nada.  La forma más conveniente en Python es
llamar a la función `signal.pause()` que simplemente espera hasta que
reciba una señal.  Ya hemos hablado de señales cuando describíamos la
orden `kill`.  Un simple *Control-C* (interrupción de usuario) es
recibido como una señal y por tanto saldría.

Cada vez que asociamos un dispositivo a un *pin* la biblioteca lo
registra como ocupado y generaría una excepción si intentamos asociar
dos dispositivos al mismo pin.  Esto es extremadamente útil para
evitar errores de programación, pero en ocasiones tenemos
multiplexados varios dispositivos a una misma pata.  Para eso se
proporciona el método *close*, que libera el pin para otros usos:

``` Python
bz = Buzzer(16)
bz.on()
bz.off()
bz.close()
led = LED(16)
led.blink()
```

La lista de las clases y dispositivos soportados crece continuamente,
así que la mejor forma de estudiar la biblioteca es emplear los
métodos de consulta de documentación de Python.

## Explorando GPIO Zero

