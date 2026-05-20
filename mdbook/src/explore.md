# ¿Qué queda por descubrir?

¡Apenas hemos arañado la superficie! ¡Quedan muchas cosas por descubrir!

> **NOTA:** si estás leyendo esto, y te gustaría ayudar a agregar ejemplos o ejercicios
> para el libro en cualquiera de los temas referidos a continuación, o cualquier otro tema 
> relevante de aplicaciones embebidas, ¡nos encantaría tener tu ayuda!
> Por favor, [abre un tema] si deseas colaborar, pero necesitas ayuda u
> orientación sobre cómo contribuir al libro o cómo abrir una solicitud de 
> incorporación de cambios para añadir la información.

[abre un tema]: https://github.com/rust-embedded/discovery-mb2/issues/new

## Más allá de la MB2
Hemos cubierto mucho del hardware de la placa MB2 en este curso, pero quedan muchos otros elementos por explorar.

## Acceso directo a memoria (DMA).
Algunos periféricos tienen DMA, una especie de `memcpy` *asíncrono* que te permite mover datos hacia o desde la memoria sin que la CPU esté involucrada.

Si estás trabajando con un micro:bit v2, en realidad ya has utilizado DMA: el HAL lo hace por ti con los periféricos UARTE y TWIM. Un DMA se puede usar para realizar transferencias masivas de datos: ya sea de RAM a RAM, de un periférico como un UARTE a RAM, o de RAM a un periférico. Puedes programar una transferencia DMA, por ejemplo "leer 256 bytes de UARTE en este búfer" y dejarla ejecutándose en segundo plano. Posteriormente, habra que verificar el registro de estado para ver si la transferencia se ha completado o pedir que se reciba una interrupción cuando la transferencia se complete. Por lo tanto, puedes programar la comunicacióm vía DMA y hacer otra tarea mientras la transferencia está en curso.

Los detalles del DMA a bajo nivel pueden ser un poco complicados. Esperamos agregar un capítulo que cubra este tema en un futuro cercano.

Hay implementadas algunas abstraciones para trabajar con el cobntrolador de modulación en amplitud (PWM) en el crate `embedded-hal`, [módulo `pwm`], y existen implementaciones de estos traits en `nrf52833-hal`.

[módulo `pwm`]: https://docs.rs/embedded-hal/latest/embedded_hal/pwm/index.html


## Entradas y salidas digitales
Hemos usado los pines del microcontrolador como salidas digitales, para controlar los leds. Cuando construimos el juego de la serpiente, también vimos brevemente cómo estos pines pueden configurarse como entradas digitales. Como entradas digitales, estos pines pueden leer el estado binario de interruptores (encendido/apagado) o botones (presionado/no presionado).

La programación de las entradas y salidas se abstrae dentro del módulo `embedded-hal` en el [módulo `digital`] y en el [crate `nrf52833-hal`]. Ambos tienen implementación para las entradas - salidas digitales.

(*spoiler*: leer el estado binario de interruptores / botones no es tan sencillo como parece 
;-))

[módulo `digital`]: https://docs.rs/embedded-hal/latest/embedded_hal/digital/index.html
[crate `nrf52833-hal`]: https://docs.rs/nrf52833-hal/latest/nrf52833_hal/

## Conversión analógica - digital (ADC)
Se han creado multitud de sensores difgitales, que se pueden leer usando protocolos como I2C o SPI. Pero también existen sensores analógicos. Estos sensores simplemente envían una lectura a la CPU del voltaje que están midiendo en un pin de entrada ADC.

El periférico ADC puede medir un nivel de voltaje "analógico", por ejemplo, `1.25` Voltios, como un número "digital", por ejemplo, `24824`, que el procesador puede usar en sus cálculos.

Había implementaciones genéricas de traits para ADC en `embedded-hal`, pero fueron eliminadas para la versión 1.0 de la misma (ver [issue #377]). El crate `nrf52833-hal` proporciona una interfaz fácil de usar para el ADC incorporado en el nRF52833.

[issue #377]: https://github.com/rust-embedded/embedded-hal/issues/377

## Conversor digital - analógica (DAC)
Como puedes haber adivinado, un DAC es exactamente lo opuesto a un ADC. Puedes escribir un número digital en un registro para producir un voltaje específico en algún pin de salida analógica. Cuando este pin de salida analógica está conectado a la electrónica adecuada y el registro se escribe rápidamente con los valores correctos, puedes hacer cosas como producir sonidos o música.

Ni el chip nRF52833 ni la placa MB2 tienen un DAC dedicado. Normalmente, se obtiene una especie de efecto DAC al emitir PWM y usar un poco de electrónica en la salida (filtro RC) para "suavizar" la forma de onda PWM.


## Reloj de tiempo real (RTC)
Un RTC es un periférico que mantiene un seguimiento del tiempo bajo su propia alimentación, generalmente en "formato humano": segundos, minutos, horas, días, meses y años. Algunos incluso manejan automáticamente los años bisiestos y el horario de verano.

Ni el chip nRF52833 ni la placa MB2 contienen un reloj RTC. El nRF52833 sí contiene un "Contador de tiempo real" (RTC), un reloj de baja frecuencia que es compatible con el módulo `nrf52833-hal`. Este contador puede dedicarse a servir como un reloj de tiempo real simulado. El requisito clave, por supuesto, es mantener el periférico RTC alimentado incluso cuando la MB2 no esté en uso. Si bien la MB2 no tiene una batería a acoplada, el RTC debería poder funcionar durante mucho tiempo (posiblemente años) con una batería conectada al puerto correspondiente en la MB2 (por ejemplo, el paquete de baterías proporcionado con el kit micro::bit Go).


## Otros protocolos de comunicación
- SPI: El "Serial Peripheral Interface" es una interfaz de comunicación de alta velocidad similar en algunos aspectos a I2C. SPI está abstraído dentro del [módulo `spi` de `embedded-hal`] y es implementado por [`nrf52-hal`].

- I2S: El protocolo "Inter-IC Sound" es una variante de I2C personalizada para la transmisión de audio. I2S actualmente no está desarrollado dentro de `embedded-hal`, pero está implementado por el módulo [`nrf52-hal`].

- Ethernet: Exite una implementación de una pequeña pila TCP/IP llamada [`smoltcp`] que está implementada para algunos chips. La MB2 no tiene un periférico Ethernet.

- USB: hay trabajo experimental sobre esto, por ejemplo con el crate [`usb-device`]. Para la MB2, el puerto USB es gestionado por el cliente MCU en lugar del host MCU, lo que dificulta hacer cosas personalizadas con USB.
- Bluetooth: el wrapper `nrf-softdevice` proporcionado por el runtime [Embassy] es probablemente la forma más fácil de acceder al Bluetooth con MB2. Embassy también cuenta con el crate de host BLE nativo de Rust:[`TrouBLE`].

- CAN, SMBUS, IrDA, etc.: Existen muchos tipos de interfaces especializadas en el mundo; Rust a veces tiene soporte para ellas. Por favor, investiga la situación actual para la interfaz que necesitas.

[`nrf52-hal`]: https://github.com/nrf-rs/nrf-hal
[módulo `spi` de `embedded-hal`]: https://docs.rs/embedded-hal/0.2.6/embedded_hal/spi/index.html
[`smoltcp`]: https://github.com/smoltcp-rs/smoltcp
[`usb-device`]: https://github.com/mvirkkunen/usb-device
[`TrouBLE`]: https://crates.io/crates/trouble-host

Aplicaciones diferentes usan protocolos de comunicación diferentes. Las aplicaciones orientadas al usuario generalmente tienen un conector USB porque USB es un protocolo ubicuo en PC y teléfonos inteligentes. Mientras que dentro de los automóviles encontrarás muchos buses CAN. Algunos sensores digitales usan SPI, I2C o SMBUS.

Si estás interesado en desarrollar para la `embedded-hal` o implementaciones de periféricos en general, no dudes en abrir un tema en los repositorios de HAL. Alternativamente, también podrías unirte al [canal de Rust Embedded en matrix] y ponerte en contacto con la mayoría de las personas que construyeron lo que se mencionó anteriormente.

## Elementos relevantes para aplicaciones embebidas
Los siguientes temas no son específicos del dispositivo, ni del hardware en él. En cambio, discuten técnicas útiles que podrían usarse en sistemas embebidos.

La mayoría del hardware del que hablaremos aquí no está disponible en la MB2, pero mucho de él podría agregarse fácilmente uniéndolo al conector en el borde de la MB2, ya sea controlándolo directamente o usando algo como SPI o I2C.

### Multitarea
La mayoría de nuestros programas se ejecutan una sola tarea. ¿Cómo podríamos lograr multitarea en un sistema sin sistema operativo, y por lo tanto sin hilos? Hay dos enfoques para la multitarea: multitarea preventiva y multitarea cooperativa.

En la multitarea preventiva, una tarea que se está ejecutando puede, en cualquier momento, ser interrumpida por otra tarea. En el proceso, la primera tarea se suspenderá y el procesador ejecutará la segunda tarea. En algún momento, la primera tarea se reanudará.
Los microcontroladores proporcionan soporte de hardware para este tipo de multitarea en forma de *interrupciones*. Introdujimos las interrupciones cuando construimos nuestro juego de la serpiente en el [capítulo 16](16-snake-game/README.md).


En la multitarea cooperativa, una tarea que se está ejecutando seguirá así hasta que alcance un *punto de suspensión*. Cuando el procesador alcanza ese punto de suspensión, dejará de ejecutar la tarea actual y en su lugar pasará a una tarea diferente. En algún momento, la primera tarea se reanudará. La principal diferencia entre estos dos enfoques para la multitarea es que en la multitarea cooperativa *cede (yields)* el control de ejecución en *puntos de suspensión conocidos* en lugar de ser suspendida a la fuerza en cualquier parte de su código.

### Giroscopios
Como parte del ejercicio de medición de golpes (Punch-o-meter), usamos el acelerómetro para medir cambios en la aceleración en tres dimensiones. Pero hay otros sensores de movimiento como los giroscopios, que nos permiten medir cambios en el "giro" en tres dimensiones.

Puede ser miy útil para la creación de ciertos sistemas, como un robot que quiere evitar volcarse. Además, los datos de un sensor como un giroscopio también se pueden combinar con los datos de un acelerómetro utilizando una técnica llamada Sensor Fusion (ver más abajo para obtener más información).

### Motor Servo y paso a paso
Mientras que algunos motores se usan principalmente para girar en una dirección u otra, por ejemplo, conduciendo un coche de control remoto hacia adelante o hacia atrás, a veces es útil medir con más precisión cómo gira un motor.

Un microcontrolador puede utilizarse para controlar un motor Servo o paso a paso, que permiten un control más preciso de cuántas vueltas está dando el motor, o incluso pueden posicionar el motor en un lugar específico, por ejemplo, si queremos fijar las manecillas de un reloj en una posición particular.

### Fusión de sensores (Sensor fusion)
La placa micro:bit contiene dos sensores de movimiento: un acelerómetro y un magnetómetro. Por sí solos, estos miden la aceleración (propia) y el campo magnético (de la Tierra). Pero estas magnitudes pueden "fusionarse" en algo más útil: una medición "robusta" de la orientación de la placa, con menos error de medición que la de cualquier sensor individual.

Esta idea de derivar datos más confiables a partir de diferentes fuentes se conoce como fusión de sensores (sensor fusion).

---

¿Qué es lo siguiente? 

Lo primero y más importante, únete al [canal de Rust Embedded en matrix]. Muchas personas que contribuyen o trabajan en software embebido se reúnen allí, incluyendo, por ejemplo, a las personas que escribieron el BSP `microbit`, el crate `nrf52-hal`, los crates `embedded-hal`, etc. ¡Estaremos felices de ayudarte a comenzar o avanzar con la programación embebida en Rust!

[canal de Rust Embedded en matrix]: https://matrix.to/#/#rust-embedded:matrix.org

Hay muchas más opciones:
- Podrías revisar los ejemplos en el crate de soporte de la placa [`microbit-v2`]. Todos esos ejemplos funcionan para micro:bit.

- Si estás buscando una descripción general de lo que está disponible en Rust Embedded en este momento, consulta la lista de [Awesome Rust Embedded].

- Podrías revisar el sitio [Embassy]. Embassy es un marco de multitarea cooperativa moderno y eficiente que admite la ejecución concurrente utilizando Rust `async/await`.

- Podrías echar un vistazo a la concurrencia basada en interrupciones en tiempo real [RTIC]. RTIC es un marco de multitarea preemptiva muy eficiente que admite la priorización de tareas y la ejecución sin bloqueos.

- Podrías visitar las abstracciones que se han creado en el proyecto [`embedded-hal`] y quizá incluso intentar escribir tu propio controlador independiente de la plataforma basándote en él.

- Podrías probar a ejecutar Rust en otra placa de desarrollo. Placas populares como la ESP-32,
  la Raspberry Pi o Arduino cuentan con sus propias comunidades activas de desarrolladores de Rust.

- Podrías revisar el sitio [Tock]. Tock es un sistema operativo de código abierto para microcontroladores que se centra en la seguridad y la confiabilidad. Tock está escrito en Rust y proporciona una plataforma para ejecutar aplicaciones embebidas de manera segura.

[`embedded-hal`]: https://github.com/rust-embedded/embedded-hal
[RTIC]: https://rtic.rs
[Embassy]: https://embassy.dev
[Awesome Rust Embedded]: https://github.com/rust-embedded/awesome-embedded-rust/
[`microbit-v2`]: https://github.com/nrf-rs/microbit/
[Tock]: https://www.tockos.org
