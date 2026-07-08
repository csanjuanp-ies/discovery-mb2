## Supresión de rebotes
Como mencionamos en la sección anterior, el hardware puede ser un poco… especial. Este es sin duda el caso
de los botones del MB2 y, en realidad, de casi cualquier pulsador o interruptor en casi cualquier sistema. Si
observas varias interrupciones por cada pulsación de tecla, probablemente se deba a lo que se conoce
como "rebote" del interruptor. Es, literalmente, lo que el nombre indica: cuando los contactos eléctricos del
interruptor entran en contacto, pueden separarse y volver a entrar en contacto varias veces con bastante rapidez antes de
establecer una conexión sólida. Por desgracia, nuestro microprocesador es *muy* rápido para los estándares mecánicos: cada uno de estos rebotes genera una nueva interrupción.


Para "eliminar el rebote" del interruptor, hay que *no* procesar las interrupciones por pulsación del botón durante un breve periodo de tiempo después de recibir una. Por lo general, un intervalo de 50-100 ms es suficiente para eliminar el rebote. Calcular el tiempo de eliminación del rebote parece complicado: sin duda, no conviene entrar en un bucle infinito en un controlador de interrupciones, pero, por otra parte, sería difícil gestionar esto en el programa principal.


La solución pasa por otra forma de concurrencia de hardware: el periférico `TIMER`, que ya hemos utilizado en numerosas ocasiones. Se puede configurar el temporizador cuando se reciba una interrupción "válida" de un botón y no responder a más interrupciones de ese botón hasta que el periférico del temporizador haya dejado transcurrir el tiempo suficiente. Los temporizadores de `nrf-hal` vienen configurados con un valor de recuento de 32 bits y una "frecuencia de tics" de 1 MHz: un millón de tics por segundo. Para un rebote de 100 ms, basta con dejar que el temporizador cuente 100 000 tics. Cada vez que el controlador de interrupciones del botón detecte que el temporizador está en marcha, simplemente no hará nada.


La implementación de todo esto se puede ver en el siguiente ejemplo (`examples/count-debounce.rs`). Al
ejecutar el ejemplo, deberíamos ver un recuento por cada pulsación del botón.

```rust
{{#include examples/count-debounce.rs}}
```

**NOTA** Los botones del MB2 son un poco delicados: es bastante fácil pulsar uno lo suficiente como para notar un "clic", pero no lo suficiente como para que entre realmente en contacto con el interruptor. Recomiendo utilizar la uña para pulsar el botón al realizar las pruebas.