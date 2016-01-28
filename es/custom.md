[//]: # (-*- markdown -*-)
# Nuestra personalización de Raspbian

Este capítulo describe todas las acciones de personalización que se
han realizado en la tarjeta microSD que se distribuye en el taller.

* Las tarjetas están ya formateadas en VFAT por el fabricante. No
  se ha realizado ningún formateo adicional.
* Partimos de la versión de NOOBS 1.5.0 descargada del
  [sitio oficial](https://www.raspberrypi.org/downloads/noobs/).
  El archivo `zip` se ha descomprimido íntegramente en la
  tarjeta vacía.
* En el primer arranque aparece el menú de NOOBS con una única
  opción de instalación, **Raspbian**. Se instala de forma automática.
* Al reiniciar, el sistema entra automáticamente en modo
  gráfico. Configuramos los iconos para lanzamiento rápido de
  aplicaciones de manera que tengamos acceso rápido a la aplicación de
  configuración de la Raspberry Pi, un terminal, un navegador, un
  editor de textos y el entorno de programación IDLE.
* Configuramos la disposición (*layout*) de teclado en
  castellano por defecto.
* Ejecutamos la aplicación de configuración y en la pestaña de
  *Interfaces* habilitamos SPI e I2C.  También en la pestaña de
  *Localisation* seleccionamos *Locale* `es/ES`.  Además
  seleccionaremos la zona horaria `Europe/Madrid` y finalmente
  configuraremos el teclado como `Spain/Spanish`.
* Instalamos algunos paquetes adicionales: `tmux`, `i2c-tools`,
  `python-smbus`, `ipython`, `zile`, `python-dev`.
* Añadir el usuario `pi` al grupo `staff` para que funcione `pip install`.
* Instalar la biblioteca *wiringPi* para Python con `pip install wiringpi2`.
* Descargar las pruebas del sistema del repositorio GitHub del taller.
