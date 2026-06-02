# Hello World
En el capítulo anterior, escribimos un programa tipo "Hola Mundo". Pero para los programadores de sistemas embebidos, el "verdadero Hola Mundo" es hacer parpadear un LED, cualquier LED, encendiéndolo y apagándolo una vez por segundo. Un programa que hace esto se conoce comúnmente como "blinky (parpadear)".


¿Por qué parpadear? Porque demuestra que tenemos suficiente control sobre la placa con la que estamos trabajando para realizar esta tarea simple. 
Podemos cargar un programa en la máquina y ejecutarlo, somos capaces de encontrar y encender el pin apropiado en el MCU, sabemos retrasar una cantidad fija de tiempo el parpadeo. Una vez que tenemos este nivel de control, el resto de tareas se vuelven mucho más sencillas.


En los capítulos anteriores, descubrimos varias formas de cargar un programa en la MB2. Ahora solo es cuestión de qué pin encendemos y apagamos, y cómo retrasar estas acciones.

Empecemos por aprender cómo trabajar con los pines necesarios. Hay una guía de programación si sabemos cómo leer diagramas de circuitos electrónicos. 
El esquema eléctrico se encuentra en [esquema de la MB2], aquí podemos buscar el LED que queramos encender y apagar, y determinar qué pines GPIO en el chip nRF52833 están conectados a ese LED. 
(La MB2 es un poco inusual en este sentido: normalmente un LED está conectado a solo un pin que lo enciende o apaga, en este caso cada LED está conectado a dos, una fila y una columna. La "pantalla" LED de la MB2 está conectada de una manera más compleja para permitir encender y apagar combinaciones de LEDs a la vez: una característica que usaremos pronto.)

[esquema de la MB2]: https://github.com/microbit-foundation/microbit-v2-hardware/blob/main/V2.21/MicroBit_V2.2.1_nRF52820%20schematic.PDF


Vamos a trabajar con el LED en la esquina superior izquierda de la matriz de LED en la MB2. Siguiendo las pistas `ROW1` y `COL1` a las que está conectado este LED, podemos ver que van a los pines del nRF52833 etiquetados como `AC17`/`P0.21` y `B11`/`AIN4`/`P0.28`. 
Al profundizar más en la documentación encontramos que `AC17` y `B11` son los índices de fila y columna de los pines físicos en la parte inferior del chip, lo cual no nos sirve. `AIN4` solo significa que este pin puede actuar como una "Entrada Analógica", lo cual tampoco nos sirve por ahora (pero sí lo hará más adelante).

Esto solo nos deja con los valores `P0.21` y `P0.28`. Estas etiquetas corresponden a bits en la memoria del nRF52833 que pueden ser encendidos y apagados para hacer que el LED se ilumine. Por razones electrónicas, si el pin `P0.21` está encendido (salida a 3.3V) y el pin `P0.28` está apagado (aceptando voltaje), entonces el LED se iluminará.

Pero ¿cómo podemos hacer esto por software? Trabajaremos a nivel del crate `nrf52833-hal`. La capa Hardware Abstraction Layer (HAL) es un software diseñado para facilitar el trabajo con el microcontrolador. Como se puede intuir por el nombre, este en concreto se utiliza con el microcontrolador de la MB2. En él se ha programado todo lo necesario para encender el LED seleccionado.

Echemos un vistazo a `examples/light-up.rs` en el directorio de este capítulo, luego intentaremos ejecutarlo.

```
cargo run --example light-up
```

Cargamos y ejecutamos nuestro programa. ¡El LED debería estar encendido!


``` rust
{{#include examples/light-up.rs}}
```


Resaltar que accedemos al Peripheral Access Crate (PAC) para este chip a través del crate HAL. Hay que seguir una serie de pasos difíciles para acceder a nuestros pines y la capa HAL abstrae esa complicación. Por último, dado que basta con inicializar los pines con los niveles adecuados, no es necesario configurarlos. La activación de los pines es un tema que trataremos en la siguiente sección.
