# Enviar un solo byte

Lo primero que haremos será enviar un solo byte desde el microcontrolador al ordenador a través de la conexión serie.


Para ello, utilizaremos el siguiente fragmento de código (este ya se encuentra en
`11-uart/examples/send-byte.rs`):

``` rust
{{#include examples/send-byte.rs}}
```
Como se puede ver, una de las librerías que hemos usado, el módulo `serial_setup`, no es de `crates.io`, sino que la desarrollamos para este proyecto. El propósito es proporcionar un buen envoltorio alrededor del periférico UARTE. Si quieres, puedes echar un vistazo a lo que hace exactamente el módulo, pero no es necesario para entender este capítulo en general.

Profundicemos en la inicialización de la UARTE. La UARTE se configura con este fragmento de código:

```rs
uarte::Uarte::new(
    board.UARTE0,
    board.uart.into(),
    Parity::EXCLUDED,
    Baudrate::BAUD115200,
);
```
Se coge la propiedad del periférico UARTE en Rust (`board.UARTE0`) y los pines TX/RX de la placa (`board.uart.into()`), para que nadie más pueda utilizar ni el periférico UARTE ni los pines mientras los necesitemos. Después de eso, pasamos dos opciones de configuración al constructor: la velocidad de baudios (que ya deberías conocer) y una opción llamada "parity". La Paridad es una forma de permitir que las líneas de comunicación serie verifiquen si los datos que recibieron se corrompieron durante la transmisión. No es necesario para esta práctica, así que simplemente lo ignoramos. Luego lo envolvemos todo en el tipo `UartePort` para poder usarlo.


Tras la inicialización, enviamos la letra `X` (como valor de byte ASCII 88) a través de la instancia de UART recién creada. Estas funciones serie son "bloqueantes": esperan a que los datos se envíen antes de devolver el control. Esto no siempre es lo que se desea: el microcontrolador puede seguir realizando otras tareas mientras espera a que el byte se envíe. Sin embargo, en nuestro caso no teníamos nada más que hacer.

Al final, pero no menos importante, llamamos a `flush()` en el puerto. Esto se debe a que la UARTE puede decidir almacenar en un búfer la salida hasta que haya recibido cierto número de bytes para enviar. Llamar a `flush()` fuerza a escribir los bytes que tiene actualmente en este momento en lugar de esperar a que lleguen más.

## Probémoslo

Antes de flashear este programa, hay que iniciar minicom/PuTTY ya que los datos que recibimos a través de la comunicación serie no se almacenan: tenemos que verlo en vivo. Una vez que el monitor serie esté activo, podemos flashear el programa como en el capítulo 5:

```
$ cargo embed --example send-byte
  (...)
```

Y en cuanto el proceso de flasheo termine, deberíamos ver el carácter `X` aparecer en el terminal minicom/PuTTY, ¡felicidades!

Si nos lo hemos perdido, se puede pulsar el botón de reinicio situado en la parte trasera del MB2. Esto hará que el programa se vuelva a ejecutar y envíe una "X" otra vez.