# Generando el binario
El primer paso es construir el binario. Dado que el microcontrolador tiene una arquitectura diferente a la de nuestro ordenador, tendremos que realizar una compilación cruzada. 
Hacerlo en Rust es tan simple como pasar un flag extra `--target` a `rustc` o Cargo. 
La parte complicada es averiguar el argumento de ese flag: el *nombre* del destino.


Como ya hemos visto, el microcontrolador del micro:bit v2 tiene un procesador Cortex-M4F. `rustc` sabe cómo compilar de forma cruzada para la arquitectura Cortex-M y proporciona varios destinos que cubren las diferentes familias de procesadores dentro de esa arquitectura:

- `thumbv6m-none-eabi`, para los procesadores Cortex-M0 y Cortex-M1
- `thumbv7m-none-eabi`, para el procesador Cortex-M3 
- `thumbv7em-none-eabi`, para los procesadores Cortex-M4 y Cortex-M7 
- `thumbv7em-none-eabihf`, para los procesadores Cortex-M4**F** y Cortex-M7**F** 
- `thumbv8m.main-none-eabi`, para los procesadores Cortex-M33 y Cortex-M35P 
- `thumbv8m.main-none-eabihf`, para los procesadores Cortex-M33**F** y Cortex-M35P**F** 

"Thumb" aquí se refiere a una versión del conjunto de instrucciones Arm que tiene instrucciones más pequeñas para reducir el tamaño del código. La denominación `hf`/`F` significa que imolementa aceleración de punto flotante por hardware. Hará que los cálculos numéricos fraccionales ("punto decimal flotante") sean mucho más rápidos.

Para la MB2, micro:bit v2, queremos el destino `thumbv7em-none-eabihf`.

Antes de realizar la compilación cruzada, es necesario descargar una versión precompilada de la biblioteca estándar (en realidad, una versión reducida de ella) en el host local. Se hace usando `rustup`:

``` console
$ rustup target add thumbv7em-none-eabihf
```
Solo hay que hacerlo una vez; `rustup` actualizará este destino (reinstalando un nuevo componente de la biblioteca estándar `rust-std` que contiene la biblioteca `core` que usamos) cada vez que actualicemos la cadena de compilación. Por lo tanto, se puede omitir este paso si ya se agregó la toolchain necesaria anteriormente en [Verificación de la instalación].

[Verificación de la instalación]: ../03-setup/verify.md#Verificación-de-la-instalación

Con el componente `rust-std` en su lugar, ya es posible compilar el programa de forma cruzada usando Cargo. Nos aseguraremos de estar en el directorio `mdbook/src/05-meet-your-software`, luego lo construimos. Este código inicial es un ejemplo, así que lo compilamos como tal.

``` console
$ cargo build --example init
   Compiling semver-parser v0.7.0
   Compiling proc-macro2 v1.0.86
   ...

    Finished dev [unoptimized + debuginfo] target(s) in 33.67s
```

> **NOTA** Es importante que el programa se compile sin optimizaciones. El fichero `Cargo.toml` proporcionado y el comando de construcción anterior asegurarán que las optimizaciones estén desactivadas siempre que no pasemos a `cargo` el flag `--release`.

Ok, ya tenemos un ejecutable. No hará mucho, no parpadeará ningún LED: es solo una versión simplificada que construiremos más adelante en el capítulo. Para asegurarnos, vamos a verificar que el ejecutable producido es realmente un binario Arm. El comando a continuación 

    readelf -h ../../../target/thumbv7em-none-eabihf/debug/examples/init

Es equivalente al siguiente en sistemas que no tengan `readelf` instalado:

``` console
$ cargo readobj --example init -- --file-headers
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Arm
  Version:                           0x1
  Entry point address:               0x117
  Start of program headers:          52 (bytes into file)
  Start of section headers:          793112 (bytes into file)
  Flags:                             0x5000400
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         4
  Size of section headers:           40 (bytes)
  Number of section headers:         21
  Section header string table index: 19
```
Si los números no coinciden exactamente con estos, no es motivo de preocupación: gran parte depende del entorno de compilación actual.

A continuación, pasaremos el programa a la MB2.

