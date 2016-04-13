[//]: # (-*- mode: markdown; coding: utf-8 -*-)
# Matriz de botones

La biblioteca *Reactor* tal y como la hemos explicado hasta ahora
implementa un tipo de entradas `input_handler` que vale perfectamente
para botones independientes.  Sin embargo es bastante frecuente que
los botones se agrupen en botoneras dispuestas como una matriz de
hilos en la que los botones simplemente cortocircuitan el hilo
horizontal y el hilo vertical que corresponde a su posición.

