# Configuración embebida

Vamos a echar un vistazo a nuestro primer programa. Comprueba el fichero `examples/init.rs`:

``` rust
{{#include examples/init.rs}}
```

Los programas para microcontroladores son diferentes de los programas estándar en dos aspectos: `#![no_std]` y
`#![no_main]`.

El atributo `no_std` indica que este programa no usará la biblioteca estándar de Rust, la cual asume un sistema operativo subyacente; el programa utilizará en su lugar el crate `core`, un subconjunto de `std` que puede ejecutarse en sistemas hardware directamente (es decir, sistemas sin abstracciones de sistema operativo como archivos y sockets).

El atributo `no_main` indica que este programa no usará la interfaz estándar 'main', que está diseñada para aplicaciones de línea de comandos que reciben argumentos. En lugar del 'main' estándar, usaremos el atributo 'entry' del crate [`cortex-m-rt`] para definir un punto de entrada personalizado. 
En este programa hemos definido el punto de entrada como `main`, pero se podría haber usado cualquier otro nombre. 
La función del punto de entrada debe tener la firma `fn() -> !`; este tipo indica que la función no termina. 
Esto significa que el programa nunca finaliza: si el compilador detecta que esto sería posible, se negará a compilarlo.


[`cortex-m-rt`]: https://crates.io/crates/cortex-m-rt

Si eres un observador cuidadoso, habrás notado que hay un directorio `.cargo` posiblemente oculto en el proyecto de Cargo. Este directorio contiene un archivo de configuración de Cargo `.cargo/config.toml`.

```toml
{{#include .cargo/config.toml}}
```

Este fichero modifica el proceso de enlace para adaptar la disposición de memoria del programa a los requisitos del dispositivo de desarrollo. Este proceso de enlace es un requisito del crate `cortex-m-rt`. El archivo `.cargo/config.toml` también le dice a Cargo cómo construir y ejecutar el código en nuestro MB2.

Hay también un fichero `Embed.toml` aquí:

```toml
{{#include Embed.toml}}
```

Este fichero informa a `cargo-embed` que:

- Trabaja con un chip NRF52833.
- Queremos detener la ejecución en el chip después de flashearlo, por lo que nuestro programa se detiene antes de `main`.
- Queremos deshabilitar RTT. RTT es un protocolo que permite al chip enviar texto a un depurador. Ya has visto RTT en acción: fue el protocolo que envió "Hello World" en el capítulo 3.
- Queremos habilitar GDB. Este paso es necesario para procesos de depuración.

Vamos a ver que está pasando, empecemos por construir este programa.
