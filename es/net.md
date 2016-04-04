[//]: # (-*- mode: markdown; coding: utf-8 -*-)
# Red

Tanto los modelos B+ como la 2B y la 3B incluyen interfaz Ethernet.
La Raspberry Pi 3 modelo B que utilizamos en este taller incluye WiFi
pero cualquiera de las otras puede tener también WiFi por un precio de
unos 4€ empleando una interfaz WiFi USB.  Por tanto cualquier proyecto
de Raspberry Pi debe plantearse la posibilidad de comunicar datos a
través de una red TCP/IP.

Para programar en red en GNU/Linux, como en la mayoría de los sistemas
operativos modernos, se utiliza una interfaz de programación
denominada *socket API*.  Se trata de un conjunto de funciones
diseñadas para que la programación de redes se parezca mucho a la
entrada/salida con archivos.
