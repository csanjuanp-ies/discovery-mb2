# Terminología de Rust Embebido

Antes de adentrarnos en la programación del micro:bit, echemos un vistazo rápido a las bibliotecas y la terminología que serán necesarias para el resto de capítulos.

## Niveles de abstracción

Para tener una interfaz de programación completa de un microcontrolador o placa con un microcontrolador, normalmente se utilizarán los siguientes términos en diferentes niveles de abstracción:


### Crate de acceso a periféricos (PAC)

El trabajo del PAC es proporcionar una interfaz directa (más o menos segura) a los periféricos del chip, permitiéndote configurar cada bit como quieras (por supuesto, también de forma incorrecta). Por lo general, solo tendrás que lidiar con el PAC si las capas superiores no recogen todas tus necesidades o cuando estés desarrollando código de nivel superior para ellas. No es sorprendente que el PAC que vamos a usar (en su mayoría implícitamente) sea para el [nRF52].



### Capa de abstracción del hardware (HAL)

La función del HAL es construir sobre el PAC del chip una capa y proporcionar una abstracción superior que sea realmente utilizable para alguien que no conoce todo el comportamiento del chip. Normalmente, la capa HAL abstrae los periféricos completos en estructuras individuales que pueden, por ejemplo, usarse para enviar datos a través del periférico. Vamos a usar el [nRF52-hal].


### Crate para soporte de placa (BSP)

(En situaciones no relacionadas con Rust, esto suele llamarse el Board Support Package, de ahí el acrónimo.)

La tarea del BSP es abstraer toda una placa (como el micro:bit) de una vez. Eso significa que tiene que proporcionar utilidades para usar tanto el microcontrolador como los sensores, LED, etc. que puedan estar presentes en la placa. Con bastante frecuencia (especialmente con placas hechas a medida) no existirá ningún BSP. En su lugar, trabajaremos con un HAL para el chip y construiremos los controladores para los sensores nosotros o los buscaremos en `crates.io`. Afortunadamente, el micro:bit sí tiene un [BSP], así que vamos a usarlo junto con el HAL.


[nrF52]: https://crates.io/crates/nrf52833-pac
[nrF52-hal]: https://crates.io/crates/nrf52833-hal
[BSP]: https://crates.io/crates/microbit-v2

## Unificación de las capas

Lo siguiente que vamos a examinar es una pieza central de software en el mundo de Rust Embebido: [`embedded-hal`]. Como su nombre sugiere, se relaciona con el segundo nivel de abstracción que acabamos de conocer: los HALs. La idea detrás de [`embedded-hal`] es proporcionar un conjunto de traits que describen funciones compartidas entre todas las implementaciones de un periférico en todos los HALs. Por ejemplo, podríamos esperar tener capacidad de encender o apagar un pin: para encender y apagar un LED en la placa que sea.

[`embedded-hal`] nos permite escribir controladores independientes para componentes de hardware, por ejemplo, un sensor de temperatura, pero que pueda ser utilizado en cualquier otro chip en el que exista una implementación de los traits de [`embedded-hal`]. Esto se logra escribiendo el controlador de tal manera que solo dependa de los traits de [`embedded-hal`]. Los controladores escritos de esta manera se llaman *agnósticos a la plataforma.* Afortunadamente, los controladores que obtendremos de `crates.io` son casi todos agnósticos a la plataforma.

[`embedded-hal`]: https://crates.io/crates/embedded-hal


## Lecturas posteriores

Si quieres aprender más sobre estas capas de abstracción, Franz Skarman (a.k.a. [TheZoq2]) dio una charla sobre este tema durante la Oxidize 2020: [An Overview of the Embedded Rust Ecosystem].

[TheZoq2]: https://github.com/TheZoq2/
[An Overview of the Embedded Rust Ecosystem]: https://www.youtube.com/watch?v=vLYit_HHPaY
