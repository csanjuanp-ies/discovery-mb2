# Registros
Este capítulo contine un análisis técnico. Se puede [saltar este capítulo] por ahora y volver a él más tarde. Dicho esto, hay muchas cosas interesantes en él, así que recomendaría que se leyera.

[saltar este capítulo]: ../10-serial-port/readme.md

-----

Es el momento de explorar qué es lo que ocurre cuando llamamos a `display_pins.row1.set_high()` o `button_a_pin.is_high()`.

En pocas palabas, la llamada a `display_pins.row1.set_high()` simplemente escribe en algunas regiones especiales de memoria. Vayamos al directorio `09-registers` y ejecutemos el código en el fichero (`src/main.rs`).

``` rust
{{#include src/main.rs}}
```

¿Qué está pasando?

La dirección `0x50000504` apunta a un *registro*. Un registro es una región especial de memoria que controla un *periférico*. Un periférico es una pieza de electrónica que se encuentra justo al lado del procesador dentro del sock del microcontrolador y proporciona al procesador funcionalidad adicional. Después de todo, el procesador, por sí solo, solo puede hacer matemáticas y lógica.

Este registro en particular controla los pines de General Purpose Input/Output (GPIO) (GPIO es un periférico) y se puede usar para *alimentar* cada uno de esos pines a *bajo* o *alto*.

(En el nRF52833 hay más de 32 GPIOs, pero la CPU es de 32 bits. 
Por lo tanto, los pines GPIO están organizados en dos grupos "P0" y "P1", 
con un conjunto de registros para leer, escribir y configurar cada grupo. 
La dirección anterior es la dirección del registro de salida para los pines P0.)

## Un apunte: LED, salidas digitales y niveles de tensión

¿Alimentar? ¿Pin? ¿Bajo? ¿Alto?

Un pin es un contacto eléctrico. Nuestro microcontrolador tiene varios de ellos y algunos están conectados a Diodos Emisores de Luz (LED). Un LED emitirá luz cuando se le aplique tensión. 
Como su nombre indica, un LED también actúa como un "diodo". Un diodo solo dejará pasar la electricidad en una dirección. Conecta un LED "hacia adelante" y saldrá luz. Conéctalo "hacia atrás" y no pasará nada.


<a href="https://commons.wikimedia.org/wiki/File:LED_circuit.svg">
<p align="center">
<img class="white_bg" height="180" title="LED circuit" src="../assets/LED_circuit.svg" alt="Led circuit"/>
</p>
</a>

Afortunadamente, los pines del microcontrolador están conectados de tal manera que podemos alimentar los LEDs en la dirección correcta. Todo lo que tenemos que hacer es aplicar suficiente tensión a través de los pines para encender el LED. 
Los pines conectados a los LED normalmente están configurados como *salidas digitales* y pueden emitir dos niveles de tensión diferentes: "bajo", 0 Voltios, o "alto", 3 Voltios. Un nivel "alto" (tensión) encenderá el LED, mientras que un nivel "bajo" (tensión) lo apagará.

Esos estados "bajo" y "alto" se mapean directamente al concepto de lógica digital. Un valor "bajo" es `0` o `false` y otro "alto" es `1` o `true`. Por eso esta configuración de pines se conoce como salida digital.

Lo contrario a la salida digital es la entrada digital. De la misma manera que una salida digital puede ser `0` o `1`, una entrada digital puede ser `0` o `1`. La diferencia es que las salidas digitales pueden emitir tensiones, pero las entradas digitales *leen* una tensión. Cuando el microcontrolador lee un nivel de tensión por encima de un umbral alto, lo interpretará como un `1` y cuando lee un nivel de tensión por debajo de un umbral bajo, lo interpretará como un `0`.

-----
Ok, pero ¿cómo se puede saber qué hace este registro? ¡Es hora de RTRM (Leer el Manual de Referencia)!