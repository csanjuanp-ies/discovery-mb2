# Solución de problemas generales

## Problemas con `cargo-embed`

La mayoría de los problemas con `cargo-embed` se deben a que las reglas de `udev` no se han instalado correctamente en
Linux, así que asegúrate de haberlo hecho bien.

Si te quedas atascado, puedes abrir una incidencia en el [sistema de seguimiento de incidencias de `discovery`] o visitar el [canal de Matrix de Rust Embedded] o el [canal de Matrix de probe-rs] y pedir ayuda allí.


[[sistema de seguimiento de incidencias de `discovery`]: https://github.com/rust-embedded/discovery-mb2/issues
[canal de Matrix de Rust Embedded]: https://matrix.to/#/#rust-embedded:matrix.org
[canal de Matrix de probe-rs]: https://matrix.to/#/#probe-rs:matrix.org

## Probles con Cargo

### "can't find crate for `core`"

*Síntomas:*

```
   Compiling volatile-register v0.1.2
   Compiling rlibc v1.0.0
   Compiling r0 v0.1.0
error[E0463]: can't find crate for `core`

error: aborting due to previous error

error[E0463]: can't find crate for `core`

error: aborting due to previous error

error[E0463]: can't find crate for `core`

error: aborting due to previous error

Build failed, waiting for other jobs to finish...
Build failed, waiting for other jobs to finish...
error: Could not compile `r0`.

To learn more, run the command again with --verbose.
```

*Causa:*

Te has olvidado de instalar el target adecuado para el microcontrolador: `thumbv7em-none-eabihf`.

*Solución:*

Instalar el target adecuado.

``` console
$ rustup target add thumbv7em-none-eabihf
```

### Unable to flash the device: `No loadable segments were found in the ELF file`

*Síntomas:*
```console
> cargo embed
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.04s
      Config default
      Target /home/user/embedded/target/thumbv7em-none-eabihf/debug/examples/init
 WARN probe_rs::flashing::loader: No loadable segments were found in the ELF file.
       Error No loadable segments were found in the ELF file.
```

*Causa:*

Cargo necesita saber cómo compilar y vincular el programa a los requisitos del dispositivo de destino.
Por lo tanto, se debe configurar los parámetros correctos en el archivo `.cargo/config.toml`.

*Solución:*

Añadir al fichero `.cargo/config.toml` con los parámetros correctos:
```toml
{{#include ../../05-meet-your-software/.cargo/config.toml}}
```

Leer [Configuración embebida](../../05-meet-your-software/embedded-setup.md) para más detalles.
