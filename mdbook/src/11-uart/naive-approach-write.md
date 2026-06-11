# Programación novata y `write!`

## Programación novata

Probablemente, habrás escrito algo similar a (`examples/naive-send-single-byte.rs`):

```rs
{{#include examples/naive-send-string.rs}}
```
Aunque este programa es perfectamente válido, podríamos necesitar todas las ventajas de `print!` como el formateo de argumentos y demás. Si nos preguntamos cómo hacer eso, sigue leyendo.

## `write!` y `core::fmt::Write`

El crate `core::fmt::Write` de Rust, que es la parte de Rust que se puede usar sin un sistema operativo, tiene un módulo llamado `fmt` que contiene el código necesario para implementar el formateo de texto. Dentro de este módulo, hay un rasgo llamado `Write` que define una interfaz para escribir texto formateado. Cualquier tipo que implemente este rasgo puede ser utilizado con la macro `write!` para escribir texto formateado.

El trait `core::fmt::Write` nos permite usar cualquier struct que lo implemente de la misma manera que usamos `print!` en el mundo `std`. En este caso, el struct `Uart` del HAL de `nrf` implementa `core::fmt::Write`, por lo que podemos refactorizar el programa anterior quedando (`examples/send-string.rs`):

```rs
{{#include examples/send-string.rs}}
```
Si flasheamos el programa en la MB2, comprobaremos que es equivalente al basado en iteradores que se hicimos en el punto anterior.
