[//]: # (-*- mode: markdown ; coding: utf-8 -*-)
# Nuestra personalización de Raspbian

Este capítulo describe todas las acciones de personalización que se
han realizado en la tarjeta microSD que se distribuye en el taller.

* Las tarjetas están ya formateadas en VFAT por el fabricante. No
  se ha realizado ningún formateo adicional.
* Partimos de la versión de NOOBS 1.9.0 descargada del
  [sitio oficial](https://www.raspberrypi.org/downloads/noobs/).
  El archivo `zip` se ha descomprimido íntegramente en la
  tarjeta vacía.
* En el primer arranque aparece el menú de NOOBS con una única
  opción de instalación, **Raspbian**. Se instala de forma automática.
* Al reiniciar, el sistema entra automáticamente en modo
  gráfico. Configurar los iconos para lanzamiento rápido de
  aplicaciones de manera que tengamos acceso rápido a la aplicación de
  configuración de la Raspberry Pi, un terminal, un navegador, un
  editor de textos y el entorno de programación IDLE.
* Configurar el fondo de escritorio al archivo [`UCLM-EII-bg.png`](img/UCLM-EII-bg.png).
* Configurar la disposición (*layout*) de teclado en
  castellano por defecto.
* Ejecutar la aplicación de configuración y en la pestaña de
  *Interfaces* habilitar SPI e I2C.  También en la pestaña de
  *Localisation* seleccionar *Locale* `es/ES`.  Además seleccionar la
  zona horaria (*Timezone*) `Europe/Madrid`, el teclado
  como `Spain/Spanish` y finalmente la zona WiFi como `ES/Spain`.
* Instalar algunos paquetes adicionales: `tmux`, `i2c-tools`,
  `python-smbus`, `ipython`, `zile`, `python-dev`, `python-gpiozero`, 
  `mpg123`.
* Añadir el usuario `pi` al grupo `staff` para que funcione `pip install`.
* Instalar la biblioteca *wiringPi* para Python con `pip install wiringpi2`.
* Descargar las pruebas del sistema del repositorio GitHub del taller con 
  `git clone https://github.com/FranciscoMoya/rpi-src.git src` y 
  `git clone https://github.com/FranciscoMoya/rpi-workshop.git doc`.
