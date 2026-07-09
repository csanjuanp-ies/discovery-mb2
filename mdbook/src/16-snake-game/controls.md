# Controles

Nuestro protagonista se controlará mediante los dos botones situados en la parte frontal del micro:bit. El botón A hará
que la serpiente gire a la izquierda, y el botón B hará que gire a la derecha.

Utilizaremos la macro `microbit::pac::interrupt` para gestionar las pulsaciones de los botones de forma concurrente. La
interrupción la generará el periférico GPIOTE (Tareas y eventos de entrada/salida de propósito general)
del MB2.


## El módulo `controls` 

Tendremos que llevar un control de dos elementos distintos del estado global mutable: una referencia al periférico `GPIOTE` y un registro de la dirección seleccionada para el siguiente giro.

Los datos compartidos se envuelven en un `RefCell` para permitir la mutabilidad interna y el bloqueo. Se Puede obtener más información sobre `RefCell` leyendo la [documentación de RefCell] y el [capítulo sobre mutabilidad interna] del libro de Rust. `RefCell`, a su vez, se envuelve en un `cortex_m::interrupt::Mutex` para permitir un acceso seguro.  El Mutex proporcionado por el crate `cortex_m` utiliza el concepto de [sección crítica].  Solo se puede acceder a los datos de un Mutex desde el interior de una función o una closure pasada a `cortex_m::interrupt::free` (renombrado aquí como `interrupt_free` para mayor claridad), lo que garantiza que el código de la función o del cierre no pueda ser interrumpido a su vez.

[documentación de RefCell]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[capítulo sobre mutabilidad interna]: https://doc.rust-lang.org/book/ch15-05-interior-mutability.html
[sección crítica]: https://en.wikipedia.org/wiki/Critical_section

### Inicialización

Primero, vamos a configurar los botones (`src/controls/init.rs`).

```rust
{{#include src/controls/init.rs}}
```
El periférico `GPIOTE` del nRF52 cuenta con 8 "canales", cada uno de los cuales puede conectarse a un pin `GPIO` y configurarse para responder a determinados eventos, entre ellos al flanco ascendente (transición de señal baja a alta) y al flanco descendente (de señal alta a baja). Un botón es un pin `GPIO` que tiene señal alta cuando no está pulsado y señal baja en caso contrario. Por lo tanto, pulsar un botón equivale a usar un flanco descendente.

Hay que Fijarse en el uso un tanto torpe de la función `init_channel()` en la inicialización para evitar copiar y pegar el código de inicialización de los botones. Los tipos que los distintos crates integrados para el MB2 han estado ocultos, ya que a veces dan un poco de miedo. Sería bueno explorar la estructura de tipos de los crates HAL y PAC en algún momento, es un poco extraña y cuesta un poco acostumbrarse a ella. En concreto, hay que tener en cuenta que cada pin del Microbit tiene *su propio tipo único*. El propósito de la función `degrade()` en la inicialización es convertirlos a un tipo común que pueda utilizarse razonablemente como argumento de `init_channel()` y, a partir de ahí, de `input_pin()`.

Conectamos `channel0` al `button_a` y `channel1` al `button_b`. En cada caso, configuramos el botón para que genere eventos en el flanco descendente (`hi_to_lo`). Almacenamos una referencia al periférico `GPIOTE` en el mutex de `GPIO`. A continuación, "desenmascaramos" (`unmask`) las interrupciones de `GPIOTE`, lo que permite que se propaguen a través del hardware, y llamamos a `unpend` para borrar cualquier interrupción con estado pendiente (que pueda haberse generado antes de que se desenmascararan las interrupciones).

### Manejador de interrupciones (ISR)

A continuación, escribimos el código que gestiona la interrupción. Utilizamos la macro `interrupt`, reexportada desde el crate `nrf52833_hal`. Definimos una función con el mismo nombre que la interrupción que queremos gestionar (puedes verlas todas [aquí](https://docs.rs/nrf52833-hal/latest/nrf52833_hal/pac/enum.Interrupt.html)) y la anotamos con `#[interrupt]` (`src/controls/interrupt.rs`).

```rust
{{#include src/controls/interrupt.rs}}
```

Cuando se genera una interrupción `GPIOTE`, comprobamos cada botón para ver si se ha pulsado. Si solo se ha pulsado el botón A, registramos que la serpiente debe girar a la izquierda. Si solo se ha pulsado el botón B, registramos que la serpiente debe girar a la derecha. En cualquier otro caso, registramos que la serpiente no debe girar. (Que se pulsen ambos botones "al mismo tiempo" es extremadamente improbable: las pulsaciones de los botones se registran casi al instante, y este controlador de interrupciones se ejecuta muy rápido; sería difícil mantener pulsados ambos botones el tiempo suficiente para que esto ocurriera. Del mismo modo, sería difícil pulsar un botón durante un tiempo lo suficientemente breve como para que este código no lo detectara y notificara que no hay ningún botón pulsado. Aun así, Rust exige que se tengan en cuenta estos casos inesperados: el código no se compilará a menos que se comprueben todas las posibilidades.) El giro correspondiente se almacena en el mutex `TURN`. Todo esto ocurre dentro de un bloque `interrupt_free`, para garantizar que no podamos ser interrumpidos por ningún otro evento mientras gestionamos esta interrupción.

Por último, exponemos una función sencilla para obtener el siguiente giro (`src/controls.rs`).

```rust
{{#include src/controls.rs}}
```

Esta función simplemente devuelve el valor actual del mutex `TURN`. Toma un único argumento booleano,
`reset`. Si `reset` es `true`, el valor de `TURN` se restablece, es decir, se establece en `Turn::None`.

A continuación, la visualización del juego.