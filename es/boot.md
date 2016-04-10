[//]: # (-*- markdown; coding: utf-8 -*-)
# Secuencia de arranque

Una vez que la Raspberry Pi recibe alimentación se inicia una
secuencia de operaciones que se conoce como *secuencia de arranque*.
Está básicamente documentada en
[elinux.org](http://elinux.org/RPi_Software):

* Cuando la Raspberrypi Pi se enciende el procesador ARM está apagado,
  y la GPU VideoCore IV está encendida.  En este punto incluso la
  propia memoria SDRAM está deshabilitada.

* El pequeño núcleo RISC de la GPU empieza a ejecutar la primera etapa
  del *bootloader* que está almacenada en la ROM del BCM2835. Esta
  primera etapa lee la tarjeta microSD, que debe tener una primera
  partición en formato FAT32, y carga la segunda etapa del
  *bootloader*, que corresponde al contenido del archivo
  `bootcode.bin`.  Puesto que la memoria SDRAM sigue deshabilitada
  esta carga se produce en la memoria cache de segundo nivel, y la
  ejecuta.

* El código de `bootcode.bin` habilita la SDRAM y lee el archivo
  `start.elf` que contiene el firmware de la GPU en formato ELF (el
  típico de los binarios en GNU/Linux). Este archivo es también
  ejecutado por la GPU.  Entre otras cosas este archivo contiene los
  codecs de video y la implementación de OpenGL ES.

* El código de `start.elf` arranca la CPU ARM.  Un archivo adicional
  `fixup.dat` se usa para configurar la partición de la SDRAM entre la
  GPU y la CPU. A partir de aquí el sistema operativo contenido en el
  archivo `kernel.img` se ejecuta en la CPU ARM.
  
* El archivo `kernel.img` es el primer archivo de la secuencia de
  arranque que ejecuta el procesador ARM.  Habitualmente contendrá el
  núcleo del sistema operativo pero puede contener programas
  arbitrarios. Como ejemplos puede consultarse
  [el repositoro GitHub de David Welch](https://github.com/dwelch67/raspberrypi).

Esta secuencia de arranque implica que la Raspberry Pi requiere de una
tarjeta microSD para poder arrancar.  Sin embargo esto tiene una
ventaja clara, es imposible inutilizarla por errores de programación.
En la jerga habitual en sistemas empotrados se dice que no puede ser
enladrillada (*bricked* en inglés).

> **Warning**
> La primera etapa del *bootloader* solo sabe leer particiones
> formateadas en FAT32. Esto es especialmente relevante con las
> tarjetas microSD de gran tamaño, que habitualmente vienen
> formateadas en exFAT y Microsoft Windows 8 y posteriores también
> utilizan por defecto exFAT para formatearlas.  Hay multitud de
> mensajes en los foros alertando de tarjetas que no funcionan con
> Raspberry Pi cuando lo único que pasa es que no están adecuadamente
> formateadas.
