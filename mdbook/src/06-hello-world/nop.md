# NOP




Quizás nos preguntemos qué hace la llamada a `nop()` dentro del bucle en la función `wait()` en el fichero `src/bin/spin-wait.rs`.

La respuesta es que literalmente no hace nada. La función `nop()` hace que el compilador ponga una
instrucción de máquina `NOP` Arm en ese punto del programa. `NOP` es una instrucción especial que hace que la CPU la ignore. Literalmente no hace Ninguna OPeración con ella (de ahí el nombre), simplemente consume un ciclo de reloj.

Eliminemos la llamada a `nop()` dentro del bucle `wait()` y recompilemos el programa. No podemos olvidar el modo `--release`. Luego lo ejecutamos.

Volvemos a conseguir un LED que no parpadea. Sin la llamada a `nop()`, el compilador optimizó el bucle de espera, detectó que no hace nada, y lo eliminó completamente del código final.  Hemos hecho que el bucle de espera sea infinitamente rápido.

¿Cuál es la función de `nop()`? Bueno, si miramos la implementación de `nop()` encontraremos (después de un montón de búsquedas) que se implementa así:

```rust
asm!("nop", options(nomem, nostack, preserves_flags));
```

La función `nop()` es una función en línea, lo que significa que no es realmente una función en el sentido tradicional. En su lugar, cuando llamamos a `nop()`, el código de la función se inserta directamente en el programa en ese punto. En este caso, el código de la función es una sola instrucción máquina `NOP` Arm.
Por ese motivo, `NOP` no será eliminada o desplazada por el compilador: se quedará justo ahí.

La capacidad de insertar código ensamblador en el programa donde sea necesario es bastante importante en la programación embebida. A veces la CPU tendrá instrucciones que el compilador no conoce, pero que aún necesitamos para usar la CPU de manera efectiva. La directiva `asm!()` de Rust crea un mecanismo para hacer eso.

El bucle de espera activa sigue siendo terrible, pero al menos ahora no es infinitamente rápido.