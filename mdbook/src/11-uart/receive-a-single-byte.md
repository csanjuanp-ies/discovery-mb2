# Recibir un byte

De forma similar a como enviamos un byte, también podemos recibirlo. Vamos a intentarlo(`examples/receive-byte.rs`).

``` rust
{{#include examples/receive-byte.rs}}
```
La única parte que cambia, comparada con el programa de envío , es el bucle al final de `main()`. Aquí usamos la función `serial.read()` para esperar hasta que un byte esté disponible y leerlo. Luego imprimimos ese byte en la consola de depuración RTT para ver si realmente está llegando algo.

Comprobaremos que si flasheamos este programa y escribimos algo en el terminal minicom/PuTTY, veremos los números correspondientes a los bytes que hemos enviado en la consola RTT en vez de los caracteres en formato `u8` que hemos escrito.
Como la conversión de `u8` a `char` es bastante sencilla, dejaremos esta tarea como práctica si realmente queremos ver los caracteres dentro de la consola RTT.
