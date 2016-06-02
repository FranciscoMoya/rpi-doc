[//]: # (-*- mode: markdown; coding: utf-8 -*-)

# Arquitectura de software

La arquitectura de un programa no es más que su organización interna,
los componentes en los que se divide y la relación entre ellos.  Si
buscas bibliografía sobre el tema verás centenares de libros que
hablan de la arquitectura de software como una evolución de la
artesanía a la ingeniería en el campo del software.  No leerás en este
manual nada sobre esto, porque sigo pensando que la programación es
una labor artesanal, como el trabajo de los mejores ingenieros.  Y
como tal tiene que tener un componente estético muy importante.
Valorar y entrenar el arte de programar es una parte importante de la
profesionalización de la actividad.

Podemos escribir decenas de páginas sobre requisitos, diseño
arquitectónico, diseño detallado, etc.  Pero nada de esto podrá
despertar el gusanillo de los *hackers* originales, los apasionados
por la tecnología, que hacían de la mejora continua su propia forma de
vida.

En este capítulo te ofrecemos una posible forma de organizar tu
código.  Ni es la única posible ni posiblemente sea la mejor.  Cumple
con su función de equilibrar la eficiencia con la simplicidad y la
facilidad de uso por programadores principiantes.  Pero si tienes
sugerencias sobre posibles mejoras no te las guardes, envianos
realimentación para mejorar las próximas ediciones.

A diferencia de la versión C en Python tenemos soporte del lenguaje
para todas las técnicas básicas.  Programación orientada a objetos con
clases y excepciones son elementos que vamos a usar extensivamente.
El soporte nativo de constructores y destructores en el lenguaje abre
la puerta de una técnica muy importante en el software actual: RAII
(*Resource Acquisition Is Initialization*) que usaremos
extensivamente.  Sin embargo no necesitamos contar todo esto, porque
hay muchos libros libres que lo cuentan.  Usa tu libro favorito de
Python si hay algun detalle que no entiendes.
