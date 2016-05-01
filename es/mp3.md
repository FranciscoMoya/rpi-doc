[//]: # (-*- mode: markdown; coding: utf-8 -*-)
# Dispositivo MP3

Como primer ejemplo vamos a diseñar un *media player*.  Empezaremos
por un dispositivo de audio, empleando el manejador `music_player` que
ya hemos descrito.

Pero tenemos una *Raspberry Pi* con una GPU Videocore IV capaz de
decodificar video FullHD a 30 fps. ¿No es un desperdicio?

Por este motivo haremos una revisión posterior del diseño para mostrar
video y música indistintamente.  Para ello emplearemos un excelente
reproductor específico para Raspberry Pi, el `omxplayer`.

Esto nos dará la oportunidad de implementar otra forma de controlar
programas externos, usando su propia entrada/salida estándar.

## Construir un *audio player*



## Construir un *video player*

`process_handler` redirigiendo entrada y salida con las dos pipes,
como en el `pipe_handler`.

¿Por qué no es un pipe_handler?  Porque a veces no vale con una pipe,
hace falta un pseudoterminal.
