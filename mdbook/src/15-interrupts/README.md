## Interrupciones

Hasta ahora, hemos trabajado con varios componentes de hardware de la MB2. Hemos leído los botones, hemos esperado a que terminaran los temporizadores, hemos realizado comunicaciones en serie y nos hemos comunicado con dispositivos mediante I2C. Cada una de estas tareas implicaba esperar a que uno o varios periféricos estuvieran listos. Hasta ahora, la espera se realizaba mediante "sondeo": preguntando repetidamente al periférico si ya había terminado, hasta que así fuera.


Como nuestra placa microcontroladora solo tiene una CPU, no puede hacer otra cosa mientras espera. Además, una CPU que sondea continuamente un periférico desperdicia energía, y en muchas aplicaciones, no podemos permitirnos eso. ¿Podemos hacerlo mejor?

Afortunadamente, ¡sí podemos! Aunque nuestro pequeño microcontrolador no puede realizar cálculos en paralelo, sí puede cambiar fácilmente entre diferentes tareas durante la ejecución, respondiendo a eventos del mundo exterior. Este cambio se realiza mediante una característica llamada "interrupciones".

Las Interrupciones tienen un nombre muy apropiado: permiten que los periféricos interrumpan la ejecución del programa principal en cualquier momento. En nuestro MB2 con nRF52833, los periféricos están conectados al Controlador de Interrupciones Anidadas y Vectorizadas (NVIC) del núcleo. El NVIC puede detener la CPU en su ejecución, indicarle que haga otra cosa y, una vez que haya terminado, devolver la CPU a lo que estaba haciendo antes de ser interrumpida. Cubriremos las partes Anidadas y Vectorizadas del controlador de interrupciones más adelante: primero centrémonos en cómo el núcleo cambia de tareas.


### Manejo de Interrupciones

El modelo computacional usado por el NRF52833 es el mismo que usan casi todas las CPU modernas. Dentro hay ubicaciones de almacenamiento conocidas como "registros de la CPU". (Confusamente, estos registros de CPU son diferentes de los "registros de dispositivo" que discutimos anteriormente en el capítulo [Registros]) Para llevar a cabo un cálculo, la CPU normalmente carga valores desde la memoria a los registros de CPU, realiza el cálculo usando los valores de los registros y luego almacena el resultado nuevamente en la memoria. (Esto se conoce como una "arquitectura de carga y almacenamiento").

Todo lo relacionado con el cálculo que la CPU está ejecutando se almacena en los registros de la CPU. Si el núcleo va a cambiar de tarea, debe almacenar el contenido de los registros de la CPU en algún lugar para que la nueva tarea pueda usarlos como su propio espacio de trabajo. Cuando la nueva tarea se completa, la CPU puede restaurar los valores de los registros y reiniciar el cálculo anterior.  Efectivamente, eso es exactamente lo primero que hace el núcleo en respuesta a una solicitud de interrupción: detiene lo que está haciendo inmediatamente y almacena el contenido de los registros de la CPU en la pila.

El siguiente paso es saltar al código que debe ejecutarse en respuesta a una interrupción. **Una Rutina de Servicio de Interrupción (ISR), a menudo denominada "manejador" de interrupción, es una función especial en el código de la aplicación que es llamada por el núcleo en respuesta a las interrupciones**. Una "tabla de interrupciones" en la memoria contiene un "vector de interrupción" para cada posible interrupción: el vector de interrupción indica qué ISR llamar cuando se recibe una interrupción específica. Describimos los detalles del direccionamiento de ISR en la sección [NVIC y Prioridad de Interrupción].

Una vez que se llama a la ISR, el núcleo ejecuta el código. Cuando la interrupción termina, el núcleo debe restaurar los registros de la CPU y reanudar la ejecución del cálculo que estaba realizando antes de la interrupción.

## Escribir en memoria en la MB2

Vamos a escribir un programa que use interrupciones para "interrumpir" la MB2 cuando se presiona el Botón A (`examples/poke.rs`). La placa responderá diciendo "ouch" y entrando en pánico.

```rust
{{#include examples/poke.rs}}
```

El manejador de la ISR es "especial". El nombre `GPIOTE` es obligatorio, indicando que esta ISR debe almacenarse en la entrada para la interrupción `GPIOTE` en la tabla de interrupciones.

El decorador `#[interrupt]` se utiliza durante la compilación para marcar una función para que se trate de manera especial como una ISR. (Esto es un "proc macro": se puede leer más sobre ello en el [libro de Rust] si lo deseas.)

La finalidad de una "proc macro" es convertir el código fuente en otro código fuente. Si se tiene conocimientos sobre como una macro se traduce, se podría cambiar la invocación de macro. Se puede hacer esto usando las Herramientas en el [Rust Playground] o el comando "rust-analyzer: Expand macro" en el IDE.

Convertir una función `#[interrupt]` en una ISR implica varias cosas importantes:

* El compilador comprobará que la función no toma argumentos y no devuelve ningún valor (o nunca devuelve). La CPU no tiene argumentos para proporcionar a una ISR, y ningún lugar para poner un valor de retorno del gestor de interrupción. Esto se debe a que los manejadores de interrupciones tienen su propia pila de llamadas (al menos *conceptualmente*, si no siempre en la práctica).
* Se situará un punto de acceso a esta función (vector) en la tabla de interrupciones, de modo que cuando se produzca la interrupción, el núcleo pueda saltar a esta función.
* El compilador evitará la ejecución directa de la ISR desde el código de nuestra aplicación.

Son necesarios dos pasos para configurar la interrupción. Primero, se debe establecer el GPIOTE para generar una interrupción cuando el pin conectado al Botón A pase de un voltaje alto a uno bajo. Segundo, hay que programar el NVIC para permitir la interrupción. El orden importa un poco: hacer las cosas en el "orden incorrecto" puede generar una interrupción antes de que se esté listo para manejarla.

**Nota** Como con muchos microcontroladores, hay mucha flexibilidad en cuándo el GPIOTE puede generar una interrupción. Las interrupciones pueden generarse en la transición de bajo a alto del pin, de alto a bajo (como aquí), en cualquier cambio ("frontera"), cuando está bajo o cuando está alto. En el nRF52833, las interrupciones generan un evento que debe borrarse manualmente en la ISR para asegurarse de que el manejador no se llame una segunda vez para la misma interrupción. Otros microcontroladores pueden funcionar de manera diferente: se debe leer la documentación del crate de Rust y del microcontrolador para entender los detalles en otra placa.

Cuando pulsamos el botón A, vemos un mensaje de "ouch" y luego un mensaje de pánico. ¿Por qué el manejador de interrupciones llama a `panic!()`? Intenta comentar la llamada a `panic!()` y ver qué sucede cuando presionamos el botón. Los mensajes de "ouch" se desplazan por la pantalla. El NVIC registra cuando se ha emitido una interrupción: ese "evento" se mantiene hasta que el programa en ejecución lo borra explícitamente. 
Sin la llamada a `panic!()`, cuando el manejador de interrupciones regresa, el NVIC (en este caso) volverá a habilitar la interrupción, notará que todavía hay un evento de interrupción pendiente y ejecutará el manejador nuevamente. Esto continuará para siempre: cada vez que el manejador de interrupciones regrese, será llamado nuevamente. Como veremos en un momento, la indicación de interrupción puede borrarse desde dentro del manejador de interrupciones usando el método `reset_event()`.

Podemos definir manejadores de interrupciones para muchas fuentes de interrupción diferentes: cuando I2C está listo, cuando un temporizador expira, y así sucesivamente. Dentro de un manejador de interrupciones se puede hacer prácticamente cualquier cosa que se desee, pero es una buena práctica mantener los manejadores de interrupciones cortos y rápidos.

Generalmente, una vez la gestión está completa, el programa principal continúa ejecutándose como si la interrupción no hubiera ocurrido. Esto es un poco problemático: ¿cómo nota tu aplicación que la ISR se ha ejecutado y ha hecho cosas? Dado que una ISR no tiene parámetros de entrada ni resultados, ¿cómo puede el código de la ISR interactuar con el código de la aplicación?

[NVIC y Prioridad de Interrupción]: nvic-and-interrupt-priority.md
[Registros]: ../09-registers/README.md
[Rust Playground]: https://play.rust-lang.org/?version=stable&mode=debug&edition=2024
[libro de Rust]: https://doc.rust-lang.org/book/ch20-05-macros.html
