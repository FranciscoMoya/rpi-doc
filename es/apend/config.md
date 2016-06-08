[//]: # (-*- mode:markdown; coding: utf-8 -*-)

# Parámetros configurables

Todos los parámetros de configuración inicial de la Raspberry Pi se
buscan en un archivo opcional denominado `config.txt` que debe estar
en la misma partición FAT32 que el *bootloader*. Este fichero es leído
por la GPU antes de arrancar la CPU y puede verse como el equivalente
a las memorias Flash o EEPROM empleadas por el BIOS de los
computadores personales para guardar los parámetros de inicialización.

El formato del fichero es extremadamente simple. Consiste en líneas
del tipo `parámetro=valor` o líneas de comentarios que empiezan por el
signo #. Entre los parámetros configurables se encuentran los
diferentes modos de video y su configuración, la partición de la
memoria cache entre GPU y CPU, la posibilidad de deshabilitar la
memoria cache de nivel 2, la posibilidad de elevar la frecuencia de
diversos componentes por encima de las especificaciones del fabricante
(*overclocking*), los códigos de activación de los codecs no libres, o
los parámetros de arranque del sistema operativo.  Una detallada
explicación de la mayoría de estos parámetros puede encontrarse en
[elinux.org](http://elinux.org/RPiconfig).

Actualmente la distribución de software *Raspbian*, que es la que
usamos en el taller, ya dispone de una herramienta de configuración,
que no hace sino editar de forma interactiva este archivo.

Actuamente los detalles de configuración están plenamente documentados
por la Raspberry Pi Foundation en
[raspberrypi.org](https://www.raspberrypi.org/documentation/configuration/).
