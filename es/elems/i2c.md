[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Comunicaciones I2C

I2C {{ "nxp14:_i2c" | cite }} es un bus de comunicaciones entre
circuitos integrados desarrollado por Phillips Semiconductors (ahora
NXP Semiconductors).  Se trata de un bus muy sencillo con solo dos
hilos, una línea de datos (SDA) y una línea de reloj (SCL).  Se pueden
realizar transmisiones serie bidireccionales de hasta 100 kbit/s en
modo estándar y 400 kbit/s en modo rápido.  Las versiones más modernas
de I2C incorporan modos con mayores tasas de transferencia (hasta
5Mbit/s) pero el procesador de la Raspberry Pi no los implementa.

La Raspberry Pi dispone de dos periféricos para implementar I2C
{{ "12:_bcm28_arm_perip" | cite }}, el BSC (*Broadcom Serial
Controller*) que implementa el modo maestro, y el BSI (*Broadcom
Serial Interface*) que implementa el modo esclavo.  Describiremos
únicamente el BSC, que tiene mucho mayor interes para este taller.

El BSC implementa tres maestros independientes que tienen que estar en
buses I2C separados (no permite multi-maestro).  Sin embargo BSC0 se
reserva para la identificación de las placas de expansión
(especificación HAT, pines 27 y 28) y BSC2 es de uso exclusivo para la
interfaz HDMI.  Usaremos por tanto habitualmente BSC1, mediante las
patas 3 y 5 del conector J8.

La interfaz de programación es, como en todos los periféricos de la
Raspberry Pi, un conjunto de registros mapeados en memoria.  Sin
embargo en este caso se dispone de un *driver* del kernel del sistema
operativo que nos simplifica significativamente la vida.

## Explorar el bus

Los dispositivos conectados a un bus I2C tienen una dirección de 7
bits.  Aunque existe la posibilidad de utilizar direcciones de 10 bits
lo cierto es que es bastante raro encontrar dispositivos con
direcciones de más de 7 bits.  En este taller asumiremos direcciones
de 7 bits.  Esto implica que como máximo podemos tener 117
dispositivos conectados (algunas direcciones están reservadas).

Lo primero que podemos hacer es descubrir qué interfaces I2C tenemos
disponibles.  En los modelos originales solo se disponía de BSC0,
mientras que en los nuevos BS0 se usa para la identificación de placas
de expansión HAT (*hardware attached on top*).  Puedes examinarlo tú
mismo ejecutando `i2cdetect`:

```
pi@raspberrypi:~ $ i2cdetect -l
i2c-1	i2c       	3f804000.i2c                    	I2C adapter
pi@raspberrypi:~ $ ▂
```

Solo nos aparece una interfaz I2C (`i2c-1`).  Ahora podemos mirar qué
dispositivos hay conectados en el bus.  Conecta el módulo MPU6050 que
viene incluído en el kit.  Solo hay que conectar *GND*, *VCC* a 3.3V,
*SDA* y *SCL*.  Volveremos a usar `i2cdetect` indicando ahora el
número de bus I2C disponible:

```
pi@raspberrypi:~ $ i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- 68 -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --
pi@raspberrypi:~ $ ▂
```

Solo se ve un dispositivo conectado con dirección 68 en hexadecimal
(`0x68`).  En realidad el MPU6050 puede tener dos direcciones
configurando la patita *AD0*.  Normalmente esa patita tiene un
pull-down que la pone a cero lógico, que corresponde a la dirección
*0x68* , pero podemos conectarla a 3.3V para tener la dirección
*0x69*.  Esto permite tener dos MPU6050 en el mismo bus.  Uno con
dirección *0x68* y otro con dirección *0x9*.

Ya podemos leer y escribir en cualquiera de los dispositivos I2C
conectados.  Para leer podemos utilizar `i2cget` y para escribir
podemos usar `i2cset`:

```
pi@raspberrypi:~ $ i2cget -y 1 0x68 117
0x68
pi@raspberrypi:~ $ ▂
```

Hemos leído el registro *117* del dispositivo con dirección *0x68* del
bus *i2c-1*.  El MPU6050 es un
[acelerómetro y giróscopo de InvenSense](http://www.invensense.com/products/motion-tracking/6-axis/mpu-6050/)
que mide también la temperatura y puede combinarse de forma directa
con un magnetómetro para tener una IMU (*Inertial Measurement Unit*)
completa.

Si miramos el
[mapa de registros](http://43zrtwysvxb2gf29r5o0athu.wpengine.netdna-cdn.com/wp-content/uploads/2015/02/MPU-6000-Register-Map1.pdf)
el registro 117 corresponde a *Who Am I*, es un registro que
simplemente devuelve la dirección base del dispositivo.  Puede usarse
para asegurarnos de que el dispositivo está en la dirección esperada y
está respondiendo.  Es un registro de solo lectura, así que si
intentamos escribir en él lo ignorará completamente:

```
pi@raspberrypi:~ $ i2cset -y 1 0x68 117 0x70
pi@raspberrypi:~ $ i2cget -y 1 0x68 117
0x68
pi@raspberrypi:~ $ ▂
```

Vamos a leer la última medida de la temperatura.  La medida está en
los regstros 65 y 66, pero si leemos los dos registros de forma
independiente podemos estar leyendo parte de la temperatura de una
medida y otra parte de otra medida.  Por eso hay que leer los dos
registros de golpe, haciendo una transferencia de 16 bits (modo
palabra):

```
pi@raspberrypi:~ $ i2cget -y 1 0x68 65 w 
0x0000
pi@raspberrypi:~ $ ▂
```

Ups, un valor extraño. Algo falla.  Lo normal es tomarse un poco de
tiempo en leer detenidamente la hoja de datos y el mapa de registros,
para entender cómo funciona.  La pista del problema nos llega en
cuanto intentamos leer el registro 107 (*Power Management 1*):

```
pi@raspberrypi:~ $ i2cget -y 1 0x68 107 
0x40
pi@raspberrypi:~ $ ▂
```

Eso significa que está puesto a uno el bit 6, que curiosamente
corresponde con el modo *SLEEP*.  El dispositivo arranca en modo
*dormido* para no consumir batería, tenemos que despertarlo quitando
ese bit:

```
pi@raspberrypi:~ $ i2cset -y 1 0x68 107 0
pi@raspberrypi:~ $ i2cget -y 1 0x68 65 w 
0x90f2
pi@raspberrypi:~ $ ▂
```

Esto ya es otra cosa, ya tenemos lecturas de temperatura. Pero es muy
importante interpretarlas bien.  I2C es un protocolo orientado a
bytes.  Se transfiere byte a byte.  Al transferir una palabra por I2C
el ordenador entiende que primero llega el byte bajo y luego el alto.
Este convenio se llama *little-endian* y es el dominante hoy en día.
Sin embargo si miramos el mapa de registros veremos que la dirección
65 corresponde al byte alto y el 66 al byte bajo.  ¡Por tanto los
bytes están cambiados!  La lectura hay que interpretarla como
`0xf290`.  Vamos a ver cómo podemos hacerlo con la *shell*:

```
pi@raspberrypi:~ $ T=$(i2cget -y 1 0x68 65 w)
pi@raspberrypi:~ $ echo "0x${T:4:2}${T:2:2}"
0xf230
pi@raspberrypi:~ $ ▂
```

Lo que saca la orden `i2cget` lo metemos en una variable `T` y luego
imprimimos un `0x` seguido de los dos caracteres a partir del cuarto
de `T` y seguido de los dos caracteres a partir del segundo de `T`.
Esto empieza a complicarse, ¿verdad?  Va siendo hora de pensar en
hacer un programa en C o en Python.  Pero ya aunque sea por dejar el
ejemplo completo vamos a hacer la transformación completa a una
temperatura en grados Celsius:

```
pi@raspberrypi:~ $ echo "$(($T - 0x10000))/340 + 36.53" | bc -l
26.13000000000000000000
pi@raspberrypi:~ $ ▂
```

Componemos la fórmula que viene en el mapa de registros.  La
temperatura es el valor del registro correctamente interpretado
partido por `340` mas `36.5`.  El problema es que la *shell* no sabe
hacer operaciones aritméticas con reales, así que se lo dejamos a otro
programa, en este caso `bc`.  A `bc` le pasamos la expresión a
calcular, que es `0xf230/340 + 36.5`.  El problema es que `bc` no
entiende de números hexadecimales, así que lo tenemos que pasar a
decimal con signo.  Eso se puede hacer con el operador `$((x))` de la
shell, que sirve para calcular operaciones sencillas con enteros.
Como es negativo su valor es el resultado de restar `0x10000`.

¿Demasiado complicado? Sí, yo también lo creo.  Por eso es necesario
que simplifiquemos esto con el mecanismo más poderoso que nos ofrece
la informática: la *abstracción*.  Pero dejaremos esto para el momento
en que veamos cómo implementar todas estas lecturas en C o en Python.

Vamos a terminar este capítulo ilustrando otra forma de leer, de una
ráfaga, todos los registros de medida:

```
pi@raspberrypi:~ $ i2cdump -y -r 59-72 1 0x68 b
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
30:                                  fd f4 fe 5c 3e               ???\>
40: f0 f1 60 ff a2 ff 39 ff 78                         ??`.?.9.x
pi@raspberrypi:~ $ ▂
```

La orden `i2cdump` permite leer de una ráfaga un conjunto de
registros.  Leemos de golpe desde el registro 61 hasta el 72.
Corresponden a las lecturas de (por orden):

* Acelerómetro.
  * Aceleración en X: `0xfdf4`
  * Aceleración en Y: `0xfe5c`
  * Aceleración en Z: `0x3ef0`
* Temperatura: `0xf160`.
* Giróscopo:
  * Rotación en X: `0xffa2`
  * Rotación en Y: `0xff39`
  * Rotación en Z: `0xff78`

Solo queda interpretar estos números como enteros con signo en
complemento a 2.  Queda propuesto como ejercicio.

El giróscopo mide la velocidad de giro en los tres ejes.  Obviamente
si no se dispone de una referencia global se irá acumulando error y su
medida absoluta no será muy útil para determinar la orientación real.
Por eso el MPU6050 tiene la opción de acoplarse con un magnetómetro,
que proporciona una referencia real del norte, que permite corregir
los errores acumulados.  En el taller no vamos a usar magnetómetro,
pero plantéatelo para tus proyectos.
