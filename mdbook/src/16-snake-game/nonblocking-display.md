# Visualización no bloqueante

A continuación, mostraremos la serpiente y la comida en los LEDs de la pantalla MB2. Hasta ahora, hemos utilizado funciones de visualización bloqueantes, que permite que los LEDs estén al máximo de brillo o apagados. Con esto, sería posible crear un juego de la serpiente que funcionara a nivel básico. Pero quizá nos demos cuenta que, cuando la serpiente se alargue un poco más, resulta difícil distinguirla de la comida y saber en qué dirección se dirige. Averigüemos cómo hacer que el brillo de los LEDs varíe: podemos hacer que el cuerpo de la serpiente sea un poco más tenue, lo que ayudará a aclarar la imagen.

La biblioteca `microbit` ofrece dos interfaces diferentes para la matriz de LEDs. Está la
interfaz con bloqueo que ya hemos visto en capítulos anteriores. También hay una interfaz no bloqueante que permite personalizar el brillo de cada LED. A nivel de hardware, cada LED está o bien "encendido" o bien "apagado", pero el módulo `microbit::display::nonblocking` simula diez niveles de brillo para cada LED encendiéndolo y apagándolo rápidamente.

(No hay ninguna razón de peso por la que los dos modos de visualización de la biblioteca `microbit` tengan que estar separados y utilizar código distinto. Un diseño más completo permitiría el uso, ya sea no bloqueante o bloqueante, de una única API de visualización con niveles de brillo y frecuencias de actualización variables especificados por el usuario. Nunca se debe dar por sentado que lo que te han entregado es perfecto, ni siquiera que se acerque a la perfección. Piensa siempre en lo que se podría hacer de otra manera. Por ahora, sin embargo, trabajaremos con lo que tenemos, que es suficiente para nuestro objetivo inmediato.)

El código para interactuar con la interfaz no bloqueante (`src/display.rs`) es bastante sencillo y seguirá una estructura similar a la del código que utilizamos para interactuar con los botones. Esta vez empezaremos por el nivel superior.

## Módulo Display

```rust
{{#include src/display.rs}}
```

En primer lugar, inicializamos una estructura `microbit::display::nonblocking::Display` que representa la pantalla LED, pasándole los periféricos `TIMER1` y `DisplayPins` de la placa. A continuación, almacenamos la pantalla en un mutex. Por último, desactivamos la máscara de la interrupción `TIMER1`.

## API Display

A continuación, definimos un par de funciones que nos permiten activar (o desactivar) fácilmente la imagen que se va a mostrar (`src/display/show.rs`).

```rust
{{#include src/display/show.rs}}
```

`display_image` toma una imagen y le indica a la pantalla que la muestre. Al igual que el método `Display::show` al que llama, esta función toma una estructura que implementa el trait `tiny_led_matrix::Render`. Ese trait garantiza que la estructura contenga los datos y métodos necesarios para que `Display` la represente en la matriz de LEDs. Las dos implementaciones de `Render` proporcionadas por el módulo `microbit::display::nonblocking` son `BitImage` y `GreyscaleImage`. En un `BitImage`, cada "píxel" (o LED) está iluminado o no (como cuando utilizábamos la interfaz bloqueante), mientras que en un `GreyscaleImage` cada "píxel" puede tener un brillo diferente.

`clear_display` hace exactamente lo que su nombre sugiere.

## Gestionando la interrupción de Display

Por último, utilizamos la macro `interrupt` para definir un manejadior para la interrupción `TIMER1`. Esta interrupción se activa muchas veces por segundo, y es lo que permite a `Display` encender y apagar rápidamente los diferentes LED para dar la impresión de que los niveles de brillo varían. Lo único que hace nuestro handler es llamar al método `Display::handle_display_event`, que se encarga de ello (`src/display/interrupt.rs`).

```rust
{{#include src/display/interrupt.rs}}
```

Ahora podemos entender cómo la función `main` se encargará de la visualización: llamamos a `init_display` y utilizaremos las nuevas funciones que hemos definido para interactuar con ella.
