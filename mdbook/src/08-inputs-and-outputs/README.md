# Entradas y sondeo

En el capítulo anterior, exploramos los pines GPIO principalmente como salidas, encendiendo y apagando LED. Sin embargo, los pines GPIO también pueden configurarse como entradas, lo que permite al programa leer señales del mundo físico, como pulsaciones de botones o cambios de interruptores. En este capítulo, aprenderemos cómo leer estas señales y hacer algo útil con ellas.

## Leyendo el estado del botón

La MB2 tiene dos botones físicos, el botón A y el botón B, conectados a pines GPIO configurados como entradas. Específicamente, el Botón A está conectado al pin P0.14, y el Botón B al pin P0.23. (Puedes verificar esto en la tabla de asignación de pines oficial [tabla de asignación de pines].)


[tabla de asignación de pines]: https://tech.microbit.org/hardware/schematic/#v2-pinmap

Leer el estado de una entrada GPIO implica verificar si el nivel de voltaje en el pin es alto (3.3V, nivel lógico 1) o bajo (0V, nivel lógico 0). Cada botón en el micro:bit está conectado a un pin. Cuando el botón *no* está presionado, ese pin se mantiene alto; cuando el botón está presionado, el pin no transmite corriente.



Vamos a configurar el pin del botón A como una entrada, y luego leer su estado.

```rust
{{#include examples/button-a-bsp.rs}}
```
Crearemos un bucle de espera activa para leer continuamente el estado, y avisaremos en cuanto cambie.
