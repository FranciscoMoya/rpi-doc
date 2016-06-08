[//]: # (-*- mode: markdown ; coding: utf-8 -*-)
# Nuestra personalización de Raspbian

Este capítulo describe todas las acciones de personalización que se
han realizado en la tarjeta microSD que se distribuye en el taller.

* Las tarjetas están ya formateadas en VFAT por el fabricante. No
  se ha realizado ningún formateo adicional.
  
* Partimos de la versión de NOOBS 1.9 descargada del
  [sitio oficial](https://www.raspberrypi.org/downloads/noobs/).  El
  archivo `zip` se ha descomprimido íntegramente en la tarjeta vacía.
  
* En el primer arranque aparece el menú de NOOBS con una primera
  opción de instalación, **Raspbian**. Se instala de forma automática.
  
* Configurar la disposición (*layout*) de teclado en
  castellano por defecto.
  
* Ejecutar la aplicación de configuración y en la pestaña de
  *Interfaces* habilitar SPI, I2C y Serial.  También en la pestaña de
  *Localisation* seleccionar *Locale* `es/ES UTF-8`.  Además seleccionar la
  zona horaria (*Timezone*) `Europe/Madrid`, el teclado
  como `Spain/Spanish` y finalmente la zona WiFi como `ES/Spain`.
  
* Al reiniciar, el sistema entra automáticamente en modo
  gráfico. Configurar los iconos para lanzamiento rápido de
  aplicaciones de manera que tengamos acceso rápido a la aplicación de
  configuración de la Raspberry Pi, un terminal, un navegador, un
  editor de textos y el entorno de programación IDLE.
  
* Configurar el fondo de escritorio al archivo
  [`rpi-uclm.svg`](https://github.com/FranciscoMoya/rpi-doc/blob/master/es/img/rpi-uclm.svg).

* Instalar algunos paquetes adicionales: `tmux`, `screen`, `bc`,
  `i2c-tools`, `python-smbus`, `python3-smbus`, `ipython`, `ipython3`,
  `zile`, `python-dev`, `python-gpiozero`, `python3-dev`,
  `python3-gpiozero`, `mpg123`, `manpages-es`, `gcc-4.9-doc`,
  `gdb-doc`, `wireshark`, `liblo-dev`, `python-liblo`,
  `python3-liblo`.

* Añadir el usuario `pi` al grupo `staff` para que funcione `pip
  install`, y al grupo `wireshark` para que podamos capturar tráfico
  sin ser superusuario.
  
* Instalar la biblioteca *wiringPi* para Python con `pip install
  wiringpi2` y `pip3 install wiringpi2`.
  
* Instalar la biblioteca *bcm2835* con `wget
  http://www.airspayce.com/mikem/bcm2835/bcm2835-1.50.tar.gz`, `tar
  xzvf bcm2835*tar.gz`, `cd bcm2835-1.50 ; ./configure ; make ; sudo
  make install`.
  
* Instalar la biblioteca *pigpio* con `wget
  https://github.com/joan2937/pigpio/archive/master.zip`, `unzip
  master.zip`, `cd pigpio-master ; make ; sudo make install`.
  
* Instalar `pygubu` (editor de interfaces gráficas) con `sudo pip install
  pygubu` y `sudo pip3 install pygubu`.
  
* Descargar las pruebas del sistema del repositorio GitHub del taller con 
  `git clone https://github.com/FranciscoMoya/rpi-src.git src` y 
  `git clone https://github.com/FranciscoMoya/rpi-doc.git doc`.
  
* Instalar `nodejs`, `npm` y `node-semver`.

* Instalar `gitbook` con `sudo npm install gitbook-cli -g`.

* Compilar la documentación con `gitbook install`, `gitbook build` en
  la carpeta `~/doc`.
  
* Actualizar firmware con `rpi-update`.
