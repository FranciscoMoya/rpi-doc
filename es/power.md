[//]: # (-*- markdown; coding: utf-8 -*-)
# Alimentación de Raspberry Pi

Todos los modelos de Raspberry Pi con la única excepción del *Compute
Module* reciben la alimentación a través de un conector microUSB
independiente del que se utiliza para la conexión de periféricos.
Esto garantiza que no podemos meter la pata en la alimentación, todos
los alimentadores microUSB son seguros porque proporcionan una tensión
de 5V con la polaridad adecuada.

Con un modelo A o B original, para evitar problemas con los
dispositivos USB (especialmente cuando se usan interfaces de red WiFi
o Ethernet, o discos duros portátiles) es necesario utilizar un *hub
USB* alimentado externamente.

Es más, muchos de los hubs USB disponibles en el mercado
(especialmente los más baratos) permiten alimentar la Raspberry Pi
directamente desde el puerto USB al que se conecta (*upstream port*),
dejando desconectado el puerto de alimentación.  Es preciso aclarar
que esta característica conocida como retro-alimentación (*back
powering*) hace que la alimentación no pase por el circuito de
protección contra sobretensiones.  Por tanto es muy importante que el
hub empleado tenga
[protección contra sobre-corriente con un límite no superior a 2.5A](http://www.raspberrypi.org/upcoming-board-revision/).
Afortunadamente casi todos los circuitos integrados de los hub USB
incorporan el mecanismo de detección de sobre-corriente en el propio
chip, por lo que es bastante probable que el hub implemente dicha
protección.

Ha habido una amplia polémica en los foros acerca de qué formas de
alimentación son las ideales. Se han propuesto otras alternativas,
como la de
[utilizar el conector de GPIO para alimentar el circuito conectando directamente una fuente a los pines +Vcc y GND](http://www.raspberrypi.org/forums/viewtopic.php?f=44&t=55645). Pero
nuevamente se salta el fusible de protección, por lo que debemos
asegurar que se implementan las debidas protecciones contra
sobre-corriente.

Si investigas un poco este tema verás que hay una amplia variedad de
comentarios que parecen prevenir contra el *back powering* por
saltarse el fusible de protección.  Sin embargo ten presente que lo que
se pretende proteger es una Raspberry Pi de menos de 30€.  En el
caso de la Raspberry Pi Zero no llega a 5€. Si el hub no
implementa las protecciones necesarias es verdad que se pondría en
riesgo la Raspberry Pi, pero también se pondría en riesgo cualquier
otro dispositivo que se conecte en los puertos *downstream*.  En
particular se pondría en riesgo los discos duros, las cámaras, etc. La
mayoría de ellos tienen un coste comparable o superior a la Raspberry
Pi y nadie cuestiona que se conecten al hub. A fin de cuentas está
para eso.

> **Info** Si tu hub alimentado no permite *back powering* la
>  Raspberry Pi no se encenderá al conectarla al hub. En ese caso es
>  todavía mejor, es lo correcto según la especificación de USB.
>  Simplemente hay que conectar un cable de uno de los puertos del hub
>  al puerto microUSB de alimentación. Ocupamos un puerto más pero
>  estaremos seguros de que el circuito está protegido contra
>  sobrecorrientes.

Los modelos A+, B+, 2B y 3B ya tienen un *hub* USB incorporado y el
fusible de protección tiene una corriente nominal sensiblemente
superior (2A), por lo que se suele alimentar con un alimentador
microUSB de 2A normal, como los que se usan para cargar los teléfonos
modernos.  En el caso del modelo 3B es necesario disponer de un
alimentador de 2.5A.

El circuito de alimentación de la Raspberry Pi no es muy tolerante a
tensiones de alimentación por debajo de la nominal, pero todos los
modelos a partir del B+ incorporan un mecanismo de detección de
tensión de alimentación demasiado baja o de sobrecalentamiento.

* Cuando la tensión de alimentación cae por debajo de 4.65V se apaga
  el LED de alimentación y se muestra un pequeño cuadradito de colores
  en la esquina superior derecha de la pantalla.

* Cuando la temperatura sube por encima de 85ºC se muestra un pequeño
  cuadradito rojo en la esquina superior derecha de la pantalla.

Hemos detectado en múltiples ocasiones la condición de *under-voltage*
al emplear alimentadores de muy bajo precio en los modelos 2B y 3B, en
los que el consumo es significativamente superior a los basados en
BCM2835.  También hemos detectado la condición de *under-voltage* al
usar cables microUSB demasiado largos o de baja calidad, y al conectar
periféricos USB junto a un conversor HDMI-VGA.

Por este motivo en el *kit del alumno* empleamos fuentes de
alimentación oficiales diseñadas por la *Raspberry Pi Foundation*.  Se
trata de diseños de alta eficiencia ligeramente sobredimensionados.
Generan una tensión de alimentación de 5.1V que está dentro de los
límites tolerados y compensa la posible caída de tensión cuando
aumenta la carga del sistema (p.ej. en el arranque).

## La fuente de alimentación de la Raspberry Pi

La alimentación externa de la Raspberry Pi se utiliza para generar las
diferentes tensiones de alimentación que requieren.  La fuente de
alimentación de la Raspberry Pi genera tensiones de 5V, 3.3V, 2.5V y
1.8V que se necesitan en varias partes del PCB.

Esta parte del circuito de la Raspberry Pi es la que ha experimentado
una evolución más significativa.  Una amplia discusión sobre las
diferencias puede consultarse en el
[blog de Adafruit](https://learn.adafruit.com/introducing-the-raspberry-pi-model-b-plus-plus-differences-vs-model-b/power-supply).

Los modelos iniciales empleaban reguladores lineales mientras que los
modelos B+ y posteriores emplean fuentes conmutadas mucho más
eficientes.  La diferencia en consumo puede ser notable.

Además es frecuente leer en los foros que los modelos originales se
resetean al desconectar dispositivos USB. Con los modelos A+ y B+ se
añade un circuito protector para la alimentación especialmente
diseñado para permitir las conexiones y desconexiones en caliente.  En
cambio, a partir de la revisión 2 del modelo original ya no se incluye
limitación de corriente de alimentación.
