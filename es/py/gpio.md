[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Entradas y salidas digitales en Python

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

### Explorando GPIO Zero

No hay mejor forma de explorar la jerarquía de clases de GPIO Zero que
leyendo el código con ayuda de IDLE. Hay multitud de ejemplos en los
comentarios y documentación de cada clase de la biblioteca.
Simplemente recuerda que las funciones que empiezan por un carácter de
subrayado (`_`) no están pensadas para ser usadas fuera de la clase.

Abre IDLE y en el menú *File* selecciona *Open Module*. Escribe
`gpiozero`. Aparecerá el archivo principal de la biblioteca, que
importa todos los demás.  Selecciona otra ver el menú *File* pero
ahora pincha en *Open...*.  Elige `input_devices.py`.  En la nueva
ventana vuelve a seleccionar el menú *File* pero ahora pincha en
*Class Browser*.  Aparecerá una ventana con todas las clases definidas
en este archivo.  El mismo procedimiento lo puedes repetir con
`output_devices.py`, `spi_devices.py` o cualquier otro archivo de la
biblioteca.

Es importante conocer cómo está hecha la biblioteca porque es muy
probable que nos enfrentemos a algún dispositivo que no está
actualmente modelado en GPIO Zero.  Veamos la estructura de `Button` y
`LED`.  Navega primero por las clases de `output_devices.py`

``` Python
class LED(DigitalOutputDevice):
    pass
```

La clase `LED` no es más que un sinónimo de `DigitalOutputDevice`.

``` Python
class DigitalOutputDevice(OutputDevice):
    def __init__(self, pin=None, active_high=True, initial_value=False):
        # ...

    @property
    def value(self):
        return self._read()

    @value.setter
    def value(self, value):
        self._stop_blink()
        self._write(value)

    def close(self):
        self._stop_blink()
        super(DigitalOutputDevice, self).close()

    def on(self):
        self._stop_blink()
        self._write(True)

    def off(self):
        self._stop_blink()
        self._write(False)

    def blink(self, on_time=1, off_time=1, n=None, background=True):
    	# ...
```

Tiene métodos `on`, `off` y `blink` (parpadear), así como una
propiedad `value` que cuando se escribe cancela el parpadeo si lo hay
y escribe el valor.  Las propiedades funcionan igual que los atributos
pero al asignar valores o leer valores se invocan funciones en lugar
de acceder a los valores directamente.

Al derivar de `OutputDevice` incorpora toda la funcionalidad de esta
clase:

``` Python
class OutputDevice(SourceMixin, GPIODevice):
    def __init__(self, pin=None, active_high=True, initial_value=False):
        # ...

    def _write(self, value):
        if not self.active_high:
            value = not value
        try:
            self.pin.state = bool(value)
        except AttributeError:
            self._check_open()
            raise

    def on(self):
        self._write(True)

    def off(self):
        self._write(False)

    def toggle(self):
        with self._lock:
            if self.is_active:
                self.off()
            else:
                self.on()

    @property
    def value(self):
        return super(OutputDevice, self).value

    @value.setter
    def value(self, value):
        self._write(value)

    @property
    def active_high(self):
        return self._active_state

    @active_high.setter
    def active_high(self, value):
        self._active_state = True if value else False
        self._inactive_state = False if value else True
```

Básicamente proporciona los mismos servicios que `DigitalOutputDevice`
salvo por el parpadeo.  Además implementa la lógica necesaria para
tratar las señales desde un punto de vista lógico, independientemente
de si son activas a nivel alto o bajo.

Pero interesa especialmente las clases de las que deriva,
`SourceMixin` y `GPIODevice`.  La primera puede encontrarse en
`mixins.py`.  Un *mixin* es una clase que incorpora funcionalidad
adicional a la clase a la que se la aplique, que depende de la propia
clase.  En este caso añade una propiedad `source` que dado un iterable
lo recorre y asigna el valor leído a la propiedad `value`. Cada
lectura se separa un tiempo que es definible en otra propiedad
`source_delay`.  Veremos en seguida cómo coopera con los dispositivos
de entrada.

La otra clase ancestro es `GPIODevice` que está definida en
`devices.py`:

``` Python
class GPIODevice(Device):
    """
    Extends :class:`Device`. Represents a generic GPIO device and provides
    the services common to all single-pin GPIO devices (like ensuring two
    GPIO devices do no share a :attr:`pin`).

    :param int pin:
        The GPIO pin (in BCM numbering) that the device is connected to. If
        this is ``None``, :exc:`GPIOPinMissing` will be raised. If the pin is
        already in use by another device, :exc:`GPIOPinInUse` will be raised.
    """
    ...
```

Es decir, simplemente se encarga de garantizar que el pin no está
siendo usado por otro dispositivo.  El resto lo delega en `Device` que
está en el mismo archivo:

``` Python
class Device(ValuesMixin, GPIOBase):
    @property
    def value(self):
        raise NotImplementedError

    @property
    def is_active(self):
        return bool(self.value)
```

Define una propiedad `value` que no está implementada (se implementa
en las clases derivadas) y deriva de dos nuevas clases (`ValuesMixin`
y `GPIOBase`).

`ValuesMixin` está en `mixins.py`.  Simplemente añade la propiedad
`values` que se construye como un generador que lee una y otra vez la
propiedad `value`.  Esto permite conectarlo con la propiedad
`source`. Piensa por ejemplo qué hace el siguiente fragmento:

``` Python
led1 = LED(17)
led2 = LED(18)
led1.source = led2.values
```

El resto de clases no interesa de momento.  La biblioteca llega a una
sofisticación notable impidiendo que un usuario ponga atributos sin
darse cuenta, por un error sintáctico.  Por ejemplo, prueba esto:

``` Python
from gpiozero import *
led1 = LED(17)
led2 = LED(18)
led1.surce = led2.values
```

En Python esta sintaxis sería completamente legal, el error
tipográfico se traduciría en que el programa no funciona y el `led1`
tiene un atributo extraño `surce`.  En GPIO Zero es un error intentar
añadir atributos después de la construcción.

Vamos a explorar un poco la clase `Button`.  Navega ahora por las
clases de `input_devices.py`.

``` Python
class Button(HoldMixin, DigitalInputDevice):
    def __init__(
            self, pin=None, pull_up=True, bounce_time=None,
            hold_time=1, hold_repeat=False):
        # ...
```

Osea, que simplemente es una combinación de `HoldMixin` y
`DigitalInputDevice`.  La primera es un *mixin* y por tanto está en
`mixins.py`.

``` Python
class HoldMixin(EventsMixin):
    """
    Extends :class:`EventsMixin` to add the :attr:`when_held` event and the
    machinery to fire that event repeatedly (when :attr:`hold_repeat` is
    ``True``) at intervals defined by :attr:`hold_time`.
    """
    ...
```

Simplemente añade el evento `when_held` a la clase a la que se aplica.
Si se mantiene pulsado el botón por un tiempo configurable en la
propiedad `hold_time` entonces se invocará a la función `when_held`.

La clase `EventsMixin` es otro *mixin* que añade los eventos
`when_activated` y `when_deactivated`, así como los métodos
`wait_for_active` y `wait_for_inactive`.

Por otro lado `DigitalInputDevice` es una combinación de `EventsMixin`
(que ya conocemos) y `InputDevice`:

``` Python
class DigitalInputDevice(EventsMixin, InputDevice):
    def __init__(self, pin=None, pull_up=False, bounce_time=None):
    	...
```

En cuanto a `InputDevice` lo único que añade a un `GPIODevice` es la
propiedad `pull_up` para configurar un *pull-up* o un *pull-down*.

``` Python
class InputDevice(GPIODevice):
    def __init__(self, pin=None, pull_up=False):
    	...

    @property
    def pull_up(self):
        return self.pin.pull == 'up'
```

Pero si te has fijado bien no aparecen las propiedades `when_pressed`
o `when_released`. ¿Cómo se definen?  La respuesta está justo después
de la definición de `Button`:

``` Python
Button.is_pressed = Button.is_active
Button.pressed_time = Button.active_time
Button.when_pressed = Button.when_activated
Button.when_released = Button.when_deactivated
Button.wait_for_press = Button.wait_for_active
Button.wait_for_release = Button.wait_for_inactive
```

Añade nombres alternativos, más fáciles de recordar, para las
propiedades estándar de todos los dispositivos.

## Clases básicas

GPIO Zero tiene un buen montón de clases directamente utilizables:

Clase  | Descripción
-------|------------
`LED`    | LED conectado a pin con resistencia a masa.
`Buzzer` | *Buzzer* o chicharra conectado a pin directamente con cátodo a masa.
`PWMLED` | LED conectado a pin con resistencia a masa, con intensidad variable.
`RGBLED` | LED tricolor a tres pines con resistencia a masa.
`Motor`  | Motor conectado con un puente H a dos pines para movimiento bidireccional.
`Button` | Pulsador conectado a masa y al pin.
`LineSensor` | Sensor infrarrojo de línea para robot seguidor de línea.
`MotionSensor` | Sensor PIR con *OUT* conectado a pin a través de un divisor de tensión o *level converter*.
`LightSensor` | LDR a 3.3V con condensador de 1uF a masa y ambas patas a un pin.
`DistanceSensor` | Sensor de ultrasonidos con *TRIG* a un pin y *ECHO* a otro pin a traves de un divisor de tensión o *level converter*.
`MCP3xxx` | Convertidores AD de Microchip MCP300x, MCP320x y MCP330x.

Actualmente GPIO Zero no soporta el ADS1118, pero se trata de un
dispositivo SPI, como los MCP3xxx, así que su incorporación es
sencilla. Lo veremos en seguida.

Además GPIO Zero incluye una clase `CompositeDevice` que permite
agrupar varios dispositivos individuales.  Tienes varios ejemplos en
el archivo `boards.py`.

## Ejemplos de uso

Poco más hay que decir de las entradas y salidas digitales, pero para
ilustrar su uso vamos a poner los mismos ejemplos que en C.

En C poníamos un ejemplo en que el pin 18 conmutaba cada segundo.  En
Python podría traducirse directamente así:

``` Python
from gpiozero import LED
from time import sleep

led = LED(18)
while True:
      led.toggle()
      sleep(1)
```

Pero todavía hay una forma más fácil:

``` Python
from gpiozero import LED
from signal import pause

led = LED(18)
led.blink()
pause()
```

Si queremos cambiar el periodo no hay más que pasar parámetros
adicionales.  Por ejemplo:

``` Python
led.blink(on_time=0.3, off_time=0.7)
```

El uso de PWM es también trivial.  Por ejemplo, consideremos el caso
de un parpadeo de un LED variando la intensidad de forma sinusoidal,
como el LED de los MacBook cuando están en *stand-by*:

``` Python
led.source = scaled(sin_values(100), 0, 1, -1, 1)
```

Al definir la propiedad *source* empieza un hilo que lee valores de
esta fuente cada 0.01 segundos por defecto.  La función `sin_values`
va produciendo valores de un seno entre -1 y 1 con un periodo de 100
muestras.  Simplemente tenemos que escalar estos valores que van de -1
a 1 en los correspondientes valores entre 0 y 1.  Esa adaptación la
hace el generador `scaled`.  Todo este tipo de adaptadores que están
en el módulo `gpiozero.tools` son extremadamente útiles, vamos a
resumir los más importantes.

Función | Descripción
--------|------------
`negated` | Para cada valor *v* del iterable devuelve *not v*.
`inverted` |  Para cada valor *v* del iterable devuelve *1 - v*.
`scaled` | Escalado lineal de los valores del iterable.
`clamped` | Recortado por arriba y por abajo.
`absoluted` | Para cada valor *v* del iterable devuelve *abs(v)*.
`quantized` | Discretiza en un número de pasos homogéneos.
`all_values` | Genera *True* cuando todos los iterables generan *True*.
`any_values` | Genera *True* cuando alguno de los iterables generan *True*.
`averaged` | Promedia los iterables.
`queued` | Encola valores y hasta que no se llena la cola no genera.
`pre_delayed` | Antes de generar espera un tiempo.
`post_delayed` | Después de generar espera un tiempo.
`random_values` | Genera números aleatorios entre 0 y 1.
`sin_values` | Genera valores del seno. Se indica el periodo en muestras.
`cos_values` | Genera valores del coseno. Se indica el periodo en muestras.
`izip` | Combina varios generadores generando tuplas de elementos (está en la biblioteca `itertools`).
`cycle` | Genera una secuencia infinita a partir de un iterable finito repitiéndolo (está en la biblioteca `itertools`).

