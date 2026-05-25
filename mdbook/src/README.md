# `micro::bit v2 Embedded Discovery Book`

> ¡Descubriendo el mundo de los microcontraldores a través de [Rust]!

[Rust]: https://www.rust-lang.org/

Este libro es un curso introductorio sobre sistemas embebidos basados en microcontroladores que utiliza Rust como lenguaje de enseñanza en lugar del habitual C/C++.

## Ámbito
Se desarrollarán los siguientes temas:
- Cómo escribir, compilar, flashear y depurar un programa "embebido" (en Rust).

- Fundamentos ("periféricos") básicos de microcontroladores: Entrada y salida digital, temporizadores, modulación por ancho de pulso (PWM), convertidor analógico - digital (ADC), protocolos de comunicación comunes como: Serie, I2C y SPI, etc.

- Conceptos de multitarea: multitarea cooperativa vs. preemptiva, interrupciones, planificadores, etc.

- Conceptos de sistemas de control: sensores, calibración, filtros digitales, actuadores, control de bucle abierto, control de bucle cerrado, etc.


## Enfoque

- Para principiantes. No se requiere experiencia previa con microcontroladores o sistemas embebidos.

- Práctico. Se proporcionan multitud de ejercicios para poner en práctica la teoría. *Tú* serás quien haga la mayor parte del trabajo.

- Centrado en las herramientas. Haremos un amplio uso de herramientas para facilitar el desarrollo. La "depuración real", con GDB, y el registro se introducirán desde el comienzo. No abordaremos el uso de LED como mecanismo de depuración en este texto.

## No son objetivos

Está fuera del ámbito de este libro:

- Enseñanza del lenguaje Rust. Ya hay mucho material sobre ese tema. Nos centraremos en microcontroladores y sistemas embebidos.

- Ser un texto exhaustivo sobre teoría de circuitos eléctricos o electrónica. Solo cubriremos lo mínimo necesario para entender cómo funcionan algunos dispositivos.

- Desarrollar detalles tales como scripts de linter y el proceso de carga. Por ejemplo, usaremos herramientas existentes para ayudar a cargar el código en la placa, pero no entraremos en detalle sobre cómo funcionan esas herramientas.

Tampoco se pretende portar este material a otras placas de desarrollo; este libro hará uso exclusivo de la placa de desarrollo micro:bit v2 (MB2).

## Informando de problemas

La fuente en **inglés** de este libro está en [este repositorio]. Si hay algún tipo de errata o problema se puede comunicar mediante el [registro de incidencias].

[este repositorio]: https://github.com/rust-embedded/discovery-mb2
[registro de incidencias]: https://github.com/rust-embedded/discovery-mb2/issues

## Otros recursos para Rust embebido

Este libro es solo uno de varios recursos sobre Rust embebido, proporcionados por el [Grupo de Trabajo de Sistemas Embebidos]. La selección completa se puede encontrar en [La Estantería de Libros de Rust Embebido]. Esto incluye la lista de [Preguntas Frecuentes].



[Grupo de Trabajo de Sistemas Embebidos]: https://github.com/rust-embedded/wg
[La Estantería de Libros de Rust Embebido]: https://docs.rust-embedded.org
[Preguntas Frecuentes]: https://docs.rust-embedded.org/faq.html
