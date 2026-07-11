# Crate de soporte de placa

Trabajar directamente con el PAC y el HAL es bastante agradable. La mayoría de los MCUs de Arm y muchos otros MCUs para los que Rust puede compilar tienen una crate PAC. Si estamos trabajando con un microcontrolador en el que no existe, programar una crate PAC puede ser tedioso, pero es bastante sencillo. Muchos MCUs que tienen una crate PAC también tienen una crate HAL; nuevamente, es principalmente un trabajo farragoso construir uno si no se ha desarrollado. El código escrito a nivel de PAC y HAL da acceso a los detalles más internos del MCU.

Como hemos visto, sin embargo, se vuelve bastante molesto mantener un seguimiento de lo que está sucediendo en la interfaz entre nuestro nRF52833 y el resto de la MB2. Hemos tenido que leer esquemas y cosas por el estilo para ver cómo usar el hardware de la placa.

Un "crate de soporte de placa", conocido en la comunidad de sistemas embebidos no Rust como un Paquete de Soporte de Placa (BSP - Board Support Package), es un crate construido sobre el HAL y el PAC de una placa, para abstraer la mayoría de los detalles y proporcionar una interfaz amigable. El crate de soporte de placa con el que hemos estado trabajando es el crate `microbit-v2`.

Vamos a usar `microbit-v2` para obtener un blinky final y limpio (`src/main.rs`).


```rust
{{#include src/main.rs}}
```
En este caso, no hemos cambiado mucho. Nuestro crate BSP ha ocultado el PAC (por ahora). Más importante aún, lo ha hecho permitiéndonos usar nombres razonables para los pines GPIO de fila y columna para el LED.

El crate `microbit-v2` también proporciona un soporte elegante para esos LED de "pantalla". Pronto lo veremos, programando cosas más divertidas que parpadear.