# Temporizadores

Una de las mayores ventajas de un sistema embebido "bare-metal" es que controlamos todo lo que sucede en la máquina. Esto permite tener un control realmente preciso del tiempo: nada ralentizará la ejecución a menos que lo permitamos.

Sin embargo, hemos visto que, si realmente queremos acertar con el tiempo, probablemente necesitemos ayuda. Los MCUs como el nRF52833 proporcionan este tipo de apoyo en forma de "temporizadores". Un temporizador es un periférico que, como su nombre indica, actúa como un pequeño reloj que lleva un seguimiento muy preciso del tiempo.

El nRF52833 contiene cuatro temporizadores. Si accedemos a la documentación del chip, veremos que son bastante complicados de configurar y usar. Afortunadamente, el crate HAL ha creado funciones alrededor de los temporizadores para que las situaciones más comunes sean fáciles de programar. El uso más normal de un temporizador es retrasar por una cantidad precisa de tiempo: justo lo que nuestra función `wait()` de las secciones anteriores necesitaba realizar.

Echemos un vistazo a `examples/timer-blinky.rs`. Este código configura un temporizador y lo usa para retrasar durante 500 ms (0,5 s) entre cada cambio de estado.

```rust
{{#include examples/timer-blinky.rs}}
```

Ejecutemos el código con `cargo run --release --example timer-blinky`, si lo medimos con un cronómetro. comprobaremos que es exactamente un segundo para cada ciclo de encendido-apagado.

Cosas a tener en cuenta:

* Hemos necesitado usar el trait `embedded_hal::Delay` para obtener el método `delay_ms()` que estamos usando.

* Al igual que antes, recogemos el periférico de la estructura de periféricos del PAC y se lo pasamos al HAL.

Por fin tenemos un programa válido para producción. Vamos a hablar un poco sobre las implicaciones de todo esto.
