[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Programación de la interfaz I2C

Desde Python la comunicación con los dispositivos I2C se puede
realizar de tres formas:

* La biblioteca *smbus* proporciona acceso desde Python a la
  funcionalidad del driver I2C de Linux.

* La biblioteca *wiringPi2* que no es sino un envoltorio de la
  biblioteca C.

* La biblioteca *pigpio* que también ofrece un envoltorio a la versión
  remota (a través de *pigpiod*) de la biblioteca C.

Actualmente GPIO Zero no proporciona ningún ejemplo de dispositivo
I2C.  Eso no significa que no pueda integrarse de forma relativamente
sencilla, pero los detalles de la programación I2C no se abstraen en
modo alguno.

Veamos los detalles de la comunicación con *smbus* y ya nos ocuparemos
de la construcción de abstracciones más adelante.

I2C es un protocolo que se utiliza en multitud de dispositivos en
ordenadores convencionales.  Las placas bases actuales disponen de un
*System Management Bus* que utiliza el protocolo I2C para comunicar
con los sensores de temperatura, y de voltajes, gestionar la velocidad
de los ventiladores, etc.  El kernel Linux incorpora un driver que no
es específico para Raspberry Pi y es precisamente esta interfaaz la
que explota la biblioteca *smbus*.

El uso de la biblioteca es sumamente sencillo:

```Python
from smbus import SMBus

i2c = SMBus(1)
```

Solo el bus 1 está disponible en la Raspberry Pi 3.  El bus 0 está
reservado para el VideoCore, pero si vamos a prescindir de su uso y de
la cámara CSI es posible habilitarlo en `/boot/config.txt` utilizando
la opción `i2c_vc=on`.

Ahora tenemos disponible un conjunto de métodos cuyo significado es
muy similar al de las funciones I2C de *pigpio* (ver capítulo 3).

``` Python
from smbus import SMBus
from struct import pack, unpack
from time import sleep

i2c = SMBus(1);
assert 0x68 == i2c.read_byte_data(0x68, 117)
i2c.write_byte_data(0x68, 107, 0)
while True:
    bytes = i2c.read_i2c_block_data(0x68, 59, 14)
    data = unpack('>7h', pack('14B', bytes))
    print("temperatura:", data[3]/340. + 36.53)
    print("ax={}, ay={}, az={}".format(*data))
    sleep(.5)
i2c.close()
```

Método | Significado
-------|------------
`open(bus)` | Conecta el objeto con un canal I2C para el bus indicado.
`close()`   | Desconecta el objeto del bus I2C.
`write_byte_data(addr, reg, v)` | Escribe un byte *v* en un registro *reg*.
`write_word_data(addr, reg, v)` | Escribe un short *v* en un registro *reg*.
`read_byte_data(addr, reg)`     | Lee un byte de un registro *reg*.
`read_word_data(addr, reg)`     | Lee un short de un registro *reg*.
`write_i2c_block_data(addr, reg, data)` | Escribe un bloque a partir de *reg*.
`read_i2c_block_data(addr, reg, len)`   | Lee un bloque a partir de *reg*.

Evidentemente es relativamente sencillo adaptar esta interfaz a la
filosofía de GPIO Zero.  Queda propuesto como ejercicio traducir el
ejemplo anterior en un *InputDevice* cuyos valores son tuplas con los
tres datos de aceleración.