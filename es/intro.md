[//]: # (-*- markdown; coding: utf-8 -*-)

# Taller de Raspberry Pi

Un taller no aspira a dar unos conocimientos teóricos profundos, sino
que tiene una orientación estrictamente práctica.  Te enseñaremos lo
que incluye la Raspberry Pi, cómo configurarla y cómo usarla para
realizar tus propios proyectos.  Se pretende que los ejemplos sean
abordables incluso por alumnos de primer curso que ya hayan cursado la
asignatura de *Informática*.

Una importante incorporación en esta edición del taller son las
nociones de arquitectura software.  Queremos que se usen las
*Raspberry Pi* para abordar problemas reales de ingeniería.  Y para
eso es necesario que el software desarrollado sea considerablemente
más evolucionado que lo que solemos ver en los *trabajos fin de
grado*.  Los ejemplos que abordaremos son sencillos, pero no
triviales.  Se proporcionarán plantillas de componentes reusables para
construir sistemas relativamente sofisticados.

> **Info** 
> [<img src="img/gpl3.png" height="90" style="float:right"/>](http://www.gnu.org/licenses/gpl-3.0.en.html)
> Todo el código que te entregamos con el taller puedes usarlo en tus
> propios trabajos y proyectos.  Se distribuye bajo la 
> [licencia pública de GNU](http://www.gnu.org/licenses/gpl-3.0.en.html),
> una licencia permisiva que te permite incluso
> modificar el software o explotar comercialmente tus proyectos.  Solo
> hay una condición, los trabajos derivados solo se pueden distribuir
> bajo esta licencia.

Una limitación importante de este taller es que no tratamos con
sistemas de tiempo real estricto pero no podemos hacer más en dos
créditos.  En un futuro próximo intentaremos ofrecer cursos
complementarios de tiempo real y robótica con *Raspberry Pi*.


## Kit del alumno

Este taller está concebido como una actividad de motivación
*pro-bono*, sin ningún tipo de remuneración para el personal
involucrado en el curso.  El 100% del dinero recaudado en las
matrículas se invierte en el material que se lleva el alumno.  Las
compras se realizan con meses de antelación, gracias a la colaboración
de la Escuela de Ingeniería Industrial de Toledo, para poder
aprovechar ofertas y proveedores extranjeros.

Cada edición del taller tiene su propia selección de componentes.  En
esta edición la selección ha sido la siguiente:

<figure style="padding:10px">
  <iframe src="https://docs.google.com/spreadsheets/d/16aW5zV-DAbm8R-N74DJ7_KGVBSAacWIodptxNJuLd38/pubhtml?gid=1395231998&amp;single=true&amp;headers=false&amp;range=A1:B25&amp;chrome=false&amp;gridlines=false" frameborder="0" style="width:100%;height:330px"></iframe>

  <figcaption style="font-size:smaller; font-style:italic">
  <div style="width:600px">Preselección de componentes para esta edición.</div>
  </figcaption>
</figure>

Los enlaces a la derecha te llevarán al sitio del fabricante
seleccionado.  Los precios pueden variar ligeramente respecto al
momento de compra.  Por ejemplo, en marzo ya teníamos las Raspberry Pi
3 modelo B, los alimentadores originales y las tarjetas microSD.
Hemos tenido problemas en el pasado con otro tipo de alimentadores más
baratos y no hemos querido arriesgar.

## Diferencias con ediciones pasadas

Todo es diferente.  Las ediciones pasadas del taller dedicaban la
mayor parte del tiempo a conseguir un entorno de desarrollo cómodo con
la Raspberry Pi.  Esto limitaba enormemente el tiempo que podíamos
dedicar a hacer proyectos y, por tanto, la utilidad del taller.

En esta edición todo el material se proporcionará completamente
configurado y listo para usarse y no será necesario ningún software en
otro ordenador personal.  El kit del alumno incluirá un conversor
HDMI-VGA para poder usar los monitores del laboratorio y se conectará
directamente el ratón y el teclado USB del puesto de laboratorio.

También hemos tratado los problemas de infraestructura que sufrimos en
ediciones pasadas.  En primer lugar hemos preparado el taller para que
no sea necesario ningún tipo de comunicación con el exterior, ni
soporte de aplicaciones externas tales como *Bonjour*.  Proporcionamos
todos los mecanismos de comunicación posibles previamente configurados
pero no necesitaremos ninguno.

> **Warning** 
> Dada la radicalidad de los cambios que hemos hecho en el
> taller es posible que convoquemos una edición 3b en septiembre.  El
> objetivo es reducir sensiblemente el precio de la matrícula
> eliminando del kit del alumno la Raspberry Pi 3 y los componentes ya
> incluidos en otras ediciones.  De esta forma alumnos que ya han
> cursado ediciones pasadas del taller pueden actualizar su formación.
> En cualquier caso estas ediciones intermedias solo se producirán si
> hay demanda suficiente.

Otra diferencia importante es que en esta edición incorporamos un
[nuevo sitio web](https://sites.google.com/site/tallerraspberrypi/) de
soporte al curso.  Queremos activar una comunidad de usuarios
interesados alrededor de este sitio.  Participa y haznos llegar tus
sugerencias.

## Estructura del manual

Este manual está dividido en tres partes:

* La primera parte introduce la Raspberry Pi, sus características, su
  historia, el sistema operativo que vamos a emplear, y el entorno de
  desarrollo.

* La segunda parte describe los diferentes componentes de la Raspberry
  Pi desde un punto de vista aislado.  Se trata de que el alumno sepa
  cómo se programa cada componente y qué limitaciones tiene.

* Finalmente la última parte se dedica a temas de arquitectura
  software.  Cómo construimos programas que tratan con múltiples
  fuentes de eventos heterogéneas.  Cómo se organiza un programa
  complejo para que no sea imposible modificarlo.

## Repositorios GitHub

La última versión del material del curso está disponible en todo
momento en [GitHub](http://github.com) en los siguientes repositorios:

```
https://github.com/FranciscoMoya/rpi-doc.git
https://github.com/FranciscoMoya/rpi-src.git
```

El primer repositorio corresponde a la documentación del taller, a los
archivos a partir de los cuales se genera este manual.  A menos que
quieras adaptarlo para otro fin es probable que prefieras descargarla
de
[gitbooks.io](https://franciscomoya.gitbooks.io/taller-de-raspberry-pi/).
Ten presente que el manual no se distribuye bajo la licencia de GNU,
sino bajo la licencia
[Creative Commons Reconocimiento-NoComercial-CompartirIgual 4.0 Internacional](http://creativecommons.org/licenses/by-nc-sa/4.0/).

El segundo repositorio corresponde al software de apoyo, que tienes
preinstalado en tu *Raspberry Pi*.  Para actualizarlo ejecuta un
terminal de órdenes y escribe:

```
cd src
git pull -u
```
