<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Licencia de Creative Commons" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br /><span xmlns:dct="http://purl.org/dc/terms/" property="dct:title">Taller de Raspberry Pi</span> by <span xmlns:cc="http://creativecommons.org/ns#" property="cc:attributionName">Francisco Moya</span> is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Reconocimiento-NoComercial-CompartirIgual 4.0 Internacional License</a>.<br />Creado a partir de la obra en <a xmlns:dct="http://purl.org/dc/terms/" href="https://www.gitbook.com/book/franciscomoya/taller-de-raspberry-pi/" rel="dct:source">https://www.gitbook.com/book/franciscomoya/taller-de-raspberry-pi/</a>.

# Taller de Raspberry Pi

Este pequeño manual es la documentación oficial de la tercera edición
del *Taller de iniciación a Raspberry Pi*, que se celebrará en junio
de 2016 en la Escuela de Ingeniería Industrial de Toledo, de la
Universidad de Castilla-La Mancha, España.

Raspberry Pi es un pequeño computador diseñado inicialmente como una
herramienta docente para enseñar a programar y electrónica digital.
Un taller no aspira a dar unos conocimientos teóricos profundos, sino
que tiene una orientación estrictamente práctica.

Te enseñaremos lo que incluye la Raspberry Pi, cómo configurarla y
cómo usarla para realizar tus propios proyectos.  Invertiremos un
considerable esfuerzo en explicar cómo comunicar la Raspberry Pi con
el mundo exterior, añadiendo sensores y actuadores de diferentes
tipos.  Se pretende que los ejemplos sean abordables incluso por
alumnos de primer curso que ya hayan cursado la asignatura de
*Informática*.

Una importante incorporación en esta edición del taller son las
nociones de arquitectura software.  Queremos que se usen las
*Raspberry Pi* para abordar problemas reales de ingeniería.  Y para
eso es necesario que el software desarrollado sea considerablemente
más evolucionado que lo que solemos ver en los *trabajos fin de
grado*.  Los ejemplos que abordaremos son sencillos, pero no
triviales.  Se proporcionarán plantillas de componentes reusables para
construir sistemas relativamente sofisticados.

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
  <iframe src="https://docs.google.com/spreadsheets/d/16aW5zV-DAbm8R-N74DJ7_KGVBSAacWIodptxNJuLd38/pubhtml?gid=1395231998&amp;single=true&amp;headers=false&amp;range=A1:B15&amp;chrome=false&amp;gridlines=false" frameborder="0" style="width:100%;height:350px"></iframe>

  <figcaption style="font-size:smaller; font-style:italic">
  <div style="width:600px">Preselección de componentes para esta edición.</div>
  </figcaption>
</figure>

Los precios pueden variar ligeramente respecto al momento de compra.
Por ejemplo, en marzo ya teníamos las Raspberry Pi 3 modelo B, los
alimentadores originales y las tarjetas microSD.  Hemos tenido
problemas con otro tipo de alimentadores más baratos y no hemos
querido arriesgar.

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

> **Warning** 
> Dada la radicalidad de los cambios que hemos hecho en el
> taller es posible que convoquemos una edición 3b en septiembre.  El
> objetivo es reducir sensiblemente el precio de la matrícula
> eliminando del kit del alumno la Raspberry Pi 3 y los componentes ya
> incluidos en otras ediciones.  En cualquier caso estas ediciones
> intermedias solo se producirán si hay demanda suficiente.

## Estructura del manual

El manual está dividido en tres partes:

* La primera parte introduce la Raspberry Pi, sus características y su
  historia, el sistema operativo que vamos a emplear, y el entorno de
  desarrollo que vamos a emplear.

* La segunda parte describe los diferentes componentes de la Raspberry
  Pi desde un punto de vista aislado.  Se trata de que el alumno sepa
  cómo se programa cada componente y qué limitaciones tiene.

* Finalmente la última parte se dedica a temas de arquitectura
  software.  Cómo construimos programas que tratan con múltiples
  fuentes de eventos heterogéneas.  Cómo se organiza un programa
  complejo para que no sea imposible modificarlo.
