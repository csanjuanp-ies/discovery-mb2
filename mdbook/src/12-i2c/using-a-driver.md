# Usando un driver

Como ya hemos comentado en el capítulo 5, `embedded-hal` proporciona abstracciones
que se pueden usar para escribir código independiente de la plataforma que pueda interactuar con
el hardware. De hecho, todos los métodos que hemos utilizado para interactuar con el hardware
en el capítulo [Ruleta LED](../07-led-roulette/index.md) y hasta ahora en este capítulo fueron de traits, definidos por `embedded-hal`.
Ahora haremos uso real de los traits que `embedded-hal` proporciona por primera vez.

No tendría sentido implementar un driver para el LSM303AGR para cada plataforma que Rust embebido
soporta (y nuevas que podrían aparecer). Para evitar esto, se puede escribir un driver que utilice tipos genéricos que implementen traits de `embedded-hal` para proporcionar una versión de driver independiente de la plataforma. Afortunadamente, para nosotros esto ya se ha hecho en el crate [`lsm303agr`]. Por lo tanto, leer los valores reales del acelerómetro y del magnetómetro será básicamente una experiencia plug and play (además de leer un poco de documentación). De hecho, la página de `crates.io` ya nos proporciona todo lo que necesitamos saber para leer datos del acelerómetro, pero usando una Raspberry Pi. Solo tendremos que adaptarlo a nuestro chip:

[`lsm303agr`]: https://crates.io/crates/lsm303agr

Mira la página vinculada para ver el código de ejemplo de Linux para Raspberry Pi.

Como ya sabemos cómo crear una instancia de un objeto que implementa los traits [`embedded_hal::blocking::i2c`] de la [página anterior](read-a-single-register.md), adaptar el código de ejemplo es sencillo (`examples/show-accel.rs`):

[`embedded_hal::blocking::i2c`]: https://docs.rs/embedded-hal/0.2.6/embedded_hal/blocking/i2c/index.html

```rust
{{#include examples/show-accel.rs}}
```

De la misma manera que antes, podemos compilar y ejecutar este ejemplo en nuestro micro:bit.

```console
$ cargo embed --example show-accel
```
Es más, si te mueves un poco (físicamente) con la micro:bit deberíamos ver que los números de aceleración que se están imprimiendo cambian.
