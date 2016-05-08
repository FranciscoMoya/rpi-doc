[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Entradas y salidas digitales

La Raspberry Pi cuenta con un número de pines que pueden configurarse
como entradas o salidas digitales para controlar cualquier periférico,
sensor o actuador externo.  Se trata de los pines de *GPIO (General
Purpose Input/Output)*.  En este capítulo trataremos su uso desde una
perspectiva general y en capítulos posteriores nos centraremos en
proyectos concretos que los aprovechan de diferentes formas. También
cubriremos algunas placas de extensión que ofrecen mayor protección a
los pines de *GPIO*.

<figure style="padding:10px">
  <iframe src="https://docs.google.com/spreadsheets/d/1PgLL0qUC8KiJsmHOVumi-X9tgjcEINyay8LSjMT4br0/pubhtml?gid=0&amp;single=true&amp;headers=false&amp;range=A1:N21&amp;chrome=false&amp;gridlines=false" style="overflow:hidden;border-style:none;width:700px;height:470px"></iframe>

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


## Conector de *GPIO*

<figure style="float:right; padding:10px">
  <img src="../img/rpi-b2-p1p5.svg" width="200"/>

  <figcaption style="font-size:smaller; font-style:italic">
  <div style="width:200px">
  Configuración de los pines de E/S en los zócalos P1 y P5
  de los modelos A y B revisión 2.
  </div>
  </figcaption>
</figure>

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
verse en la tabla al inicio de este capítulo.  Cuando hablemos de
números de pines nos referiremos al nuevo conector J8 a menos que se
indique lo contrario.

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
5V) y pueden sacar hasta 16 mA.

Sin embargo hay que tener presente que la corriente que sale de esos
pines proviene de la fuente de alimentación de 3.3V y esta fuente está
diseñada para una carga pico de unos
[3 mA por cada pin de GPIO](http://www.scribd.com/doc/101830961).  Es
decir, aunque el SoC de Broadcom permita drenar hasta 16 mA por cada
pin la fuente no podrá dar mas de unos 78 mA en total (51 mA en los
modelos originales).  No supone riesgo alguno si intentas superar este
límite, pero no funcionará.

> **Warning**

> Los pines GPIO de la Raspberry Pi no son tolerantes a tensiones de
> 5V. Están pensados para su utilización con circuitos de 3.3V y no
> tienen ningún tipo de protección. No debes drenar más de 16mA por
> pin.

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

## Programar entradas y salidas digitales

La referencia definitiva para programar cualquiera de los periféricos
del BCM2835 es
[la hoja de datos del fabricante](http://www.raspberrypi.org/wp-content/uploads/2012/02/BCM2835-ARM-Peripherals.pdf),
aunque se trata de un documento denso y árido.  Es también
ilustrativo el documento de Gert van Loo
[GPIO pads control](http://www.scribd.com/doc/101830961).

Para empezar probablemente el mejor tutorial es el de
[elinux.org](http://elinux.org), que lleva por título
[RPi Tutorial: Easy GPIO Hardware & Software](http://elinux.org/RPi_Tutorial_Easy_GPIO_Hardware_&_Software)
y especialmente los
[ejemplos de programación utilizando diversos lenguajes y mecanismos](http://elinux.org/RPi_GPIO_Code_Samples).
El
[manual de la Gertboard](http://www.element14.com/community/servlet/JiveServlet/previewBody/51727-102-1-265829/Gertboard_UM_with_python.pdf)
también tiene abundante información pero el software es difícil de
encontrar.

Desde el punto de vista del programador los pines de GPIO de la
Raspberry Pi se ven como dispositivos mapeados en memoria.  Es decir,
para configurar los pines, sacar valores digitales o leer señales
digitales tenemos que leer o escribir en posiciones de memoria
determinadas.  Sin embargo el procesador de la Raspberry Pi utiliza un
mecanismo denominado *memoria virtual*, en el que cada proceso ve un
espacio de direcciones diferente, que no necesariamente tiene que
corresponder con el espacio físico y que garantiza el aislamiento
entre procesos.  En GNU/Linux para poder acceder a direcciones físicas
determinadas es necesario emplear un dispositivo (`/dev/mem`) que a
todos los efectos se comporta como un archivo normal.  Por ejemplo, en
la posición FIXME del dispositivo `/dev/mem` puede leerse el valor de
la entrada GPIO18.

Obviamente acceder a todo el espacio físico de direcciones es muy
peligroso, puesto que permite que desde un proceso se pueda acceder a
todo el espacio de direcciones de los demás procesos.  No solo se
compromete el aislamiento entre procesos sino también la seguridad del
sistema.  Un proceso malicioso podría utilizar funciones
privilegiadas.  Si piensas que eso no te afecta es que no sabes lo
suficiente de seguridad informática.  De vez en cuando echa un vistazo
a las ponencias de [Blackhat](http://blackhat.com) para ver qué se
cuece en el mundo de la seguridad y verás que afecta a todo (robots,
equipamiento médico, televisores, equipamiento industrial, domótica,
sistemas de telecomunicación, ...).  Un usuario malicioso podría
incluso dañar físicamente la Raspberry Pi.  Por este motivo el
dispositivo `/dev/mem` tiene permisos de escritura solamente para el
superusuario.

Las versiones recientes de Raspbian tienen un dispositivo
`/dev/gpiomem` que permite acceder solo al rango de direcciones de los
pines de GPIO y tiene permiso de escritura para el grupo `gpio`.  El
usuario `pi` es del grupo `gpio`.  Por tanto los programas del usuario
`pi` pueden actuar sobre los pines de GPIO.  En la práctica esto puede
no ser así porque muchas bibliotecas no utilizan aún `/dev/gpiomem`.

## Características de los pines de GPIO

Los pines de GPIO de la Raspberry Pi incorporan un conjunto de
características muy interantes:

* Tienen capacidad de limitar el *slew rate*.

* y programar su *drive stregth*, es decir, su capacidad de
  entregar corriente, entre 2 mA y 16 mA en saltos de 2 mA (ocho
  posibles valores).  Básicamente consiste en la posibilidad de
  activar más o menos drivers en paralelo.  Para mayor detalle
  consulta el documento de Gert van Loo referido arriba.  Normalmente
  en el arranque está configurado a 8 mA.  Esto no significa que no
  podamos pedir más corriente.  Hasta 16 mA es seguro.  Sin embargo si
  superamos los 8 mA la tensión bajará hasta el punto de que un uno
  lógico pueda dejar de interpretarse como uno y la disipación de
  calor será mayor.  Por otro lado si programamos los pines a su
  máxima capacidad tendremos picos de corriente que afectan al consumo
  y pueden llegar a afectar al funcionamiento de la trajeta microSD,
  especialmente con cargas capacitivas.  Este efecto se nota más
  cuanto mayor número de salidas conmuten simultáneamente.

Del mismo modo es posible configurar las entradas con o sin *Schmitt
trigger* de manera que la transición a nivel bajo y a nivel alto
tengan umbrales diferentes.

La configuración de limitación de *slew rate*, *drive strength* y
entrada con *Schmitt trigger* no se realiza pin a pin sino en bloques
(GPIO0-27, GPIO28-45, GPIO46-53).  Para los intereses del taller no
debería haber problemas con la configuración por defecto.
