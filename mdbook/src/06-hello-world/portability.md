# Portabilidad

(Esta sección es opcional. Eres libre de saltarla y pasar a la [siguiente sección], donde limpiamos un poco nuestro código y damos por terminado el día.)

[siguiente sección]: board-support-crate.md

Quizá nos preguntemos si todo este sofisticado ecosistema vale la pena. La configuración de nuestro "blinky" es
bastante compleja y utiliza un montón de crates y funciones de Rust para una tarea tan sencilla.

Una gran ventaja, sin embargo, es que nuestro código se vuelve realmente portátil. En otro tipo de placa, la configuración puede ser diferente, pero el bucle de ejecución es idéntico.

Echemos un vistazo al mismo programa para el Sipeed Longan Nano. Es una pequeña placa de $5 que, al igual que la MB2, trae embebida una MCU. En el resto, es completamente distinto: otro procesador (el GD32VF103, con un conjunto de instrucciones RISC-V completamente diferente al conjunto de instrucciones Arm que estamos usando), periféricos distintos, otra placa. Pero tiene un LED conectado a un pin GPIO, así que podemos hacerlo parpadear igualmente.

```rust
{{#include nanoblinky.rs}}
```

Las diferencias en la configuración son porque el hardware es diferente, y también porque este código usa una versión más antigua del crate HAL que aún no se ha actualizado para `embedded-hal` 1.0. Sin embargo, el bucle principal es idéntico, y el resto del código es bastante reconocible. Gracias a la portabilidad proporcionada por la compilación cruzada de Rust y el ecosistema de Rust embebido, blinky es simplemente blinky.

Podemos encontrar un ejemplo completo de trabajo de [nanoblinky] en GitHub, por si preferimos ver todos los detalles o incluso conseguir otra placa y probarlo.

[nanoblinky]: https://github.com/pdx-cs-rust/nanoblinky
