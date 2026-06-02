# Buble de espera activa

Para hacer parpadear el LED, necesitamos esperar aproximadamente medio segundo entre cada cambio. ¿Cómo hacemos eso?

La primera idea que se nos ocurre es usar un bucle de espera activa (forma tonta), o "spin wait". No es una muy buena aproximación, pero es un comienzo. Echemos un vistazo a `examples/spin-wait.rs`.

```rust
{{#include examples/spin-wait.rs}}
```

Ahora ejecutamos `cargo run --release --example spin-wait`, en donde la opción `--release` es muy importante. Deberíamos ver el LED de nuestro MB2 parpadear aproximadamente una vez por segundo.

Cosas que quizá te nos podemos preguntar:

* **¿Qué son los caracteres `_` en el número?** Rust permite estos caracteres en los números y los ignora. Es visualmente adecuado para hacer que los números grandes sean más legibles. Aquí los estamos usando como puntos (o lo que sea el separador para grupos de tres dígitos en el idioma correspondiente).

* **Si el microcontrolador nRF52833 tiene una frecuencia de 64MHz, ¿por qué el bucle de espera itera solo 4 millones veces? ¿No deberían ser 32?** El bucle de espera ejecuta varias instrucciones cada vez: el `nop` (ver la siguiente sección), suma algo de tiempo, y tenemos que regresar al inicio del bucle. El código generado es aproximadamente este para la primera llamada a `wait()`


  ```asm
  .LBB1_4:
      adds r3, #1
      nop
      cmp  r3, r2
      bne  .LBB1_4
  ```

  y este para la segunda

  ```asm
  .LBB1_6:
      subs	r3, #1
      nop
      bne	.LBB1_6
  ```

  Esto son solo tres o cuatro instrucciones, pero el salto hacia atrás también necesita tiempo. Observemos que la salida creada *no es igual:* el compilador elige generar diferentes instrucciones para el primer bucle y segundo bucle. Ver "Depende" más adelante.
  
  Aun así, estamos ejecutando alrededor de 4 instrucciones por iteración del bucle. Esto significa que en nuestra CPU de 64MHz un cambio de medio segundo debería tomar 64M/2/4 = 8M iteraciones para completarse. Así que algo nos está ralentizando por un factor de 2. ¿Qué lo causa? No se sabe. **Esto es terrible**.

* **Por qué `--release` es tan importante?** Prueba sin él. Notarás que el LED sigue parpadeando, pero con un período de *muchos* segundos. El bucle de espera ahora no está optimizado y necesita muchas más instrucciones cada vez que se ejecuta.

* **¿Qué significa la llamada `nop()` y por qué está ahí?** Responderemos a esta cuestión en la siguiente sección.

* **¿Por qué nos referimos a esto como  la forma tonta?**

  * **No es muy precisa.** Intentar ajustar el bucle para que dure exactamente los 0,5 segundos, no es posible.

  * **Depende.**  ¿Qué pasa si se ejecuta en una CPU diferente? ¿Y si utilizamos otros flags de compilación? En realidad no habría ninguna diferencia en la lógica, pero el resultado sería otro.

  * **Malgasta energía.** La CPU está ejecutando instrucciones tan rápido como puede, solo para perder tiempo. Si no hay nada más que hacer, tendría que esperar tranquilamente hasta que se le necesite de nuevo. Esto no importa mucho si está alimentado vía USB. Pero si conectamos la MB2 a baterías externas realmente se notará y mucho.


En la siguiente sección, hablaremos de `nop()`. Después de eso, trataremos más sobre el resto de elementos de nuestro programa blinky que son necesarios mejorar.

Para ser un simple programa, es un programa bastante complicado. Por eso empezamos con blinky.
