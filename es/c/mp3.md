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

Tienes un ejemplo de uso del manejador `media_player` en
`test/test_media_player.c`.  La única diferencia con un media player
convencional es que un dispositivo portátil no puede tener conectado
un teclado.  Partiendo de este ejemplo y de
`reactor/test/test_input.c` modifica `ejemplos/mp3/audio_player.c`
para que se controle con botones y no con el teclado.  Veamos un
posible esqueleto.

``` C
#include <reactor/reactor.h>
#include <reactor/music_player.h>
#include <reactor/input_handler.h>

enum {
    NEXT= 18,
    PREV = 23,
    PLAY = 24,
    PAUSE = 25
};

int main(int argc, char* argv[])
{
    char* path = (argc > 1
                  ? argv[1]
                  : "/usr/share/scratch/Media/Sounds/Music Loops");
    music_player* mp = music_player_new(path);

    void press(input_handler* ev, int key) {
        // Actuar sobre mp según la tecla
        // ...
    }
    void release(input_handler* ev, int key) {}    
    int keys[] = { NEXT, PREV, PLAY, PAUSE };
    wiringPiSetupGpio();
    input_handler* ih = input_handler_new(keys, sizeof(keys),
                                          press, release)

    reactor* r = reactor_new();
    reactor_add(r, (event_handler*)mp);
    reactor_add(r, event_handler_new(0, keyboard));
    reactor_run(r);
    reactor_destroy(r);
    console_restore(0, state);
    return 0;
}
```

Al igual que en `test_music_player.c` usamos las funciones anidadas de
las versiones modernas de C para pasar el *music_player* a las
funciones de *press* y *release*.  El mismo efecto (pero con más
código) puede conseguirse derivando de *input_handler* y añadiendo el
*music_player* como un atributo más.

Este tipo de aplicaciones son ideales para la Raspberry Pi Zero pero
es un desperdicio para una Raspberry Pi 3.  Por lo menos vamos a
aprovechar que tiene un mayor factor de forma para ponerle un
reproductor de video.

## Construir un *video player*

Para construir un reproductor de video necesitaríamos añadirle una
pantalla a la Raspberry Pi.  Las opciones disponibles son:

* Pantalla HDMI. Puede ser la opción más barata y no consume pines
  adicionales.  Por ejemplo, en BangGood una de 5 pulgadas con *touch
  panel* no llega a 40€.

* [La pantalla oficial de la Raspberry Pi Foundation](https://www.raspberrypi.org/blog/the-eagerly-awaited-raspberry-pi-display/)
  que utiliza la interfaz DSI de la Raspberry Pi y tiene panel táctil
  multipunto.  Puede usarse simultáneamente con un monitor HDMI.
  Aunque el precio oficial es de 60$ es difícil encontrarla a ese
  precio.  RS la tiene actualmente por 67.19€

* [PiTFT Plus de Adafruit](https://www.adafruit.com/product/2441) que
  utiliza la interfaz SPI y dos o tres pines de GPIO.  Es una pantalla
  que tiene el mismo factor de forma que los modelos B, lo que desde
  el punto de vista estético es muy conveniente, pero el consumo
  excesivo de pines reduce las posibles aplicaciones.  El coste
  depende de la resolución, entre 35$ y 45$.  Para los modelos
  anteriores a la 2B tal vez te interese
  [este clon chino de BangGood](http://www.banggood.com/3_2-Inch-TFT-LCD-Display-Module-Touch-Screen-For-Raspberry-Pi-B-B-A-p-1011516.html)
  a tan solo 10.81€.

Para construir un video player hemos hecho un nuevo manejador a partir
de `process_handler`, pero redirigiendo entrada y salida con dos
pipes, como hace el `pipe_handler`.  Esto es así porque controlamos
el programa *omxplayer*, incluido en la propia Raspberry Pi empleando
su propia entrada estándar.  Es decir, simulamos pulsaciones de
teclas.  El programa *omxplayer* es un reproductor multimedia que
aprovecha todas las características del Videocore IV, incluyendo los
codecs privativos que se venden en la tienda online de la Raspberry Pi
Foundation. Esto permitiría reproducir video full-HD a 30 fps.

Algunos programas no permiten la redirección de la entrada a una
*pipe*.  Es típico en los casos en los que se utilizan capacidades
avanzadas del terminal, como cuadros de diálogo, animaciones de texto,
ventanas, etc.  En esos casos lo más probable es que requiera una
entrada y salida estándar con capacidades de terminal.  Es el momento
de usar *pseudoterminales*.  Ampliaremos esto más adelante, en el
sitio web del taller.  Sigue las novedades.

En `ejemplos/mp3/test_video_player.c` tienes un ejemplo de cómo usar
el nuevo manejador que hemos hecho para este caso.  Tu trabajo ahora
es convertirlo en un video player con botones.  Combina ahora ese
manejador `video_player` con el `input_handler` de forma similar a
como hiciste con el *audio player*.
