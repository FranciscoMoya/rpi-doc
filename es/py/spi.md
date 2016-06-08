[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Programación en Python de SPI

En Python disponemos de varias bibliotecas para acceder a dispositivos
SPI:

* GPIO Zero, que incorpora varios convertidores AD de Microchip.

* La biblioteca *spidev* proporciona acceso desde Python a la
  funcionalidad del driver SPI de Linux.

* La biblioteca *wiringPi2* que no es sino un envoltorio de la
  biblioteca C.

* La biblioteca *pigpio* que también ofrece un envoltorio a la versión
  remota de la biblioteca C.

La biblioteca más sencilla es nuevamente GPIO Zero. Veamos un ejemplo
de la documentación de `scaled`:

``` Python
from gpiozero import Motor, MCP3008
from gpiozero.tools import scaled
from signal import pause

motor = Motor(20, 21)
pot = MCP3008(channel=0)
motor.source = scaled(pot.values, -1, 1)
pause()
```

El potenciómetro está conectado al canal 0 de un conversor AD
Microchip MCP3008.  Se trata de un dispositivo SPI, igual que nuestro
ADS1118.  El usuario está por tanto completamente aislado de la
interacción SPI.  Pero vamos a ver lo que implicaría definir un módulo
específico para el ADS1118.

## La clase *SPIDevice*

Todos los dispositivos SPI derivan de la clase `SPIDevice`.  Esta
define un atributo privado `self._spi` que se construye llamando a la
función `SPI`. Esta función devuelve un `SPIHardwareInterface` o un
`SPISoftwareInterface` dependiendo de los pines utilizados.  Si los
pines corresponden con un puerto SPI de la Raspberry Pi devolverá un
`SPIHardwareInterface`, que es mucho más eficiente.

Para cambiar la polaridad del reloj y la fase podemos utilizar la
propiedad `clock_mode` del atributo `self._spi`, que en nuestro caso
(ADS1118) debe tener el valor 1.

En lugar de usar `SPIDevice` como base podemos aprovechar la clase
`AnalogInputDevice`, que añade la lógica necesaria para decodificar el
valor escalado al rango [0, 1] dependiendo del número de bits de las
lecturas.

Con esto la clase para implementar un dispositivo ADS1118 queda
prácticamente igual que los MCP3xxx.

``` Python
class ADS1118(AnalogInputDevice):

    def __init__(self, channel=0, differential=False, **spi_args):
        self._channel = channel
        self._bits = 16
        self._differential = bool(differential)
        super(ADS1118, self).__init__(16, **spi_args)
        self._spi.clock_mode = 1
        data = self._spi.transfer(2 * self._config_reg())

    @property
    def channel(self):
        return self._channel

    @property
    def differential(self):
        return self._differential

    def _read(self):
        data = self._spi.transfer(2 * self._config_reg())
        result = (data[0] << 8) | data[1]
        if self.differential and result > 32767:
            result = -(65536 - result)
        assert -32768 <= result < 32768
        return result

    def _config_reg(self):
        ''' Configuración en modo continuo a 128 SPS y con rango a
            plena escala de +-2.048V'''
        #     Byte        0        1
        #     ==== ======== ========
        #          sMCCGGGS rrrTP011
        #
        #   s = start single shot conversion
        #   M = differential (1 = single-ended, 0 = differential)
        #   C = channel
        #   G = gain
        #   S = single shot mode (1 = single shot, 0 = continuous)
        #   r = sample rate
        #   T = temperature sensor
        #   P = pull-up in DOUT
        return [0b00000100 + (self.channel << 4) + [0b01000000, 0][self.differential],
                0b10001011]
```

Ya está, no hay más, con esto ya se puede usar como cualquiera de los
módulos AD incluidos en GPIO Zero.  Conecta un potenciómetro entre
3.3V y masa y el cursor utilízalo como entrada AN0 del módulo
CJMCU-1118. Conecta el módulo a la interfaz SPI, usando como línea de
selección CE0.

``` Python
led = PWMLED(4)
pot = ADS1118(channel=0)
led.source = pot.values
```

Un LED con intensidad controlada por un potenciómetro.
