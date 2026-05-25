# Configurando el entorno de desarrollo

La programación de microcontroladores utilizan un conjunto de herramientas diferente al que se utiliza en los sistemas de escritorio. En particular, es necesario ejecutar y depurar programas en un dispositivo "remoto".

## Documentación

Sin embargo, las herramientas no lo son todo. Sin documentación, es prácticamente imposible trabajar con microcontroladores. La documentación técnica oficial de MB2 está en <https://tech.microbit.org>. Haremos referencia a otra documentación a lo largo del libro.

## Herramientas

Haremos uso a lo largo de curso de las herramientas listadas a continuación. Cuando no se especifica una versión mínima, cualquier versión reciente debería funcionar, pero hemos mostrado la versión que hemos probado.

- Rust 1.79.0 o una toolchain más reciente.

- `gdb-multiarch`. Herramienta de depuración. La versión más antigua que hemos probado es la 10.2, pero es probable que otras versiones también funcionen. Si la distribución/plataforma no tiene `gdb-multiarch` disponible, podemos usar `arm-none-eabi-gdb` para depurar. Además, algunos binarios de `gdb` están construidos con capacidades multiplataforma: se puede encontrar más información sobre esto en el capítulo de depuración de este libro.

- [`cargo-binutils`]. Versión 0.3.6 o posterior.

  [`cargo-binutils`]: https://github.com/rust-embedded/cargo-binutils

- [`probe-rs-tools`]. Versión 0.24.0 o más reciente.

  [`probe-rs-tools`]: https://probe.rs/docs/overview/about-probe-rs/

- `minicom` en Linux y macOS. Se ha probado la versión: 2.7.1. Esperamos que versiones posteriores también funcionen.

- `PuTTY` en Windows.

A continuación, mostramos instrucciones de instalación independientes del SSOO usado, para algunas de las herramientas especificadas:

### `rustc` & Cargo

Para instalar rustup se deben seguir las indicaciones de [https://rustup.rs](https://rustup.rs).

Si ya estuviera instalado rustup, hay que asegurarse que estás en el canal estable y que la toolchain seleccionada es la estable debiendo estar actualizada. `rustc -V` debería devolver una fecha y versión no más antigua que la mostrada a continuación:

``` console
$ rustc -V
rustc 1.79.0 (129f3b996 2024-06-10)
```

En concreto yo he usado:
``` console
$ rustc -V
rustc 1.94.0 (4a4ef493e 2026-03-02)
```

### `cargo-binutils`

``` console
$ rustup component add llvm-tools
$ cargo install cargo-binutils --vers '^0.3'
$ cargo size --version
cargo-size 0.3.6
```

En mi caso:
``` console
$ rustup component add llvm-tools
$ cargo install cargo-binutils --vers '^0.4'
$ cargo size --version
cargo-size 0.4.0
```

### `probe-rs-tools`

**NOTA** si existen en el sistema versiones anteriores de `probe-run`, `probe-rs` o `cargo-embed` instaladas, hay que eliminarlas antes de seguir con este punto,  ya que podrían causar problemas en el futuro. En particular, `probe-run` ya no existe oficialmente. Prueba a eliminarlas si es necesario:

```console
$ cargo uninstall cargo-embed
$ cargo uninstall probe-run
$ cargo uninstall probe-rs
$ cargo uninstall probe-rs-cli
```

Para instalar `probe-rs-tools`, hay que seguir las instrucciones de la página oficial en <https://probe.rs>.

* **NOTA** Si se prefiere instalar `probe-rs-tools` mediante `cargo install`, puedes probar los siguientes pasos.  La gente ha tenido muchos problemas con este método, pero si quieres, puedes intentarlo.

*  **NOTA del Tr.** En mi caso la instalación mediante cargo no me ha dado ningún tipo de problema.

```console
$ cargo install probe-rs-tools

Finished `release` profile [optimized] target(s) in 3m 34s
Installing C:\Users\arcipreste\.cargo\bin\cargo-embed.exe
Installing C:\Users\arcipreste\.cargo\bin\cargo-flash.exe
Installing C:\Users\arcipreste\.cargo\bin\probe-rs.exe
Installed package `probe-rs-tools v0.31.0` (executables `cargo-embed.exe`, `cargo-flash.exe`, `probe-rs.exe`)

C:\Users\...>probe-rs
The probe-rs CLI
```

  1. Actualizar a la versión más estable de Rust.

  2. Instalar el binario `probe-rs-tools` desde los [prerequisitos](https://probe.rs/docs/getting-started/installation/).  (Las instrucciones
     a las que se remite el enlace forman parte de la documentación más general del kit de herramientas de depuración integrado [`probe-rs`](https://probe.rs/))

  3. Probar la instalación

     ```console
     $ cargo install --locked probe-rs-tools
     ```

Instalar la herramienta `probe-rs-tools` dejará en nuestro ordenador diversas herramientas muy útiles, incluyendo `probe-rs` y `cargo-embed` (que normalmente se ejecutan como un comando de Cargo). Comprueba que todo funciona correctamente antes de continuar.

```
$ cargo embed --version
cargo-embed 0.24.0 (git commit: crates.io)
```


Para mí:
```
$ cargo embed --version
cargo embed 0.31.0 (git commit: crates.io)
```
### Este repositorio

Este libro contiene algunos pequeños proyectos de Rust usados en varios capítulos: la forma más fácil de utilizarlos es descargar el código fuente del libro. Puedes hacerlo de una de las siguientes maneras:

- Visitando el [repositorio original](https://github.com/rust-embedded/discovery-mb2/), Pulsa en el botón verde "Code" y después en "Download Zip".

- Clonando el repositorio usando `git` (si sabes usar `git` probablemente ya lo tengas instalado) desde el mismo repositorio en el mismo lugar que el descargable zip, se encuentra la URL para clonarlo.

### Instrucciones específicas para cada Sistema Operativo

A continuación, están las instrucciones específicas para cada sistema operativo, sigue las del tuyo: 

- [Linux](linux.md)
- [Windows](windows.md)
- [macOS](macos.md)
