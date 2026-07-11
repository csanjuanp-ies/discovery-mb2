# Conocer el software

En este capítulo aprenderemos a construir, ejecutar y depurar algunos programas *muy* simples. El objetivo aquí no es entrar en los detalles de la programación MB2 Rust (todavía), sino simplemente familiarizarse con la mecánica del proceso.

Primero, una nota rápida sobre las convenciones utilizadas en el resto de este libro. Esperamos que obtengas una copia de todo el libro en

```
git clone http://github.com/rust-embedded/discovery-mb2
```

El código fuente del libro está en `discovery-mb2/mdbook/src`. Se debería examinar la carpeta en profundidad. Cada capítulo está en un directorio diferente y tiene tanto el texto Markdown *como* el código fuente completo de todos los programas del capítulo. Cuando hablamos de alguna ruta como `src/main.rs`, nos referimos comenzando desde el capítulo en el que estamos trabajando. Por ejemplo, `discovery-mb2` tiene un archivo llamado `mdbook/src/05-meet-your-software/examples/init.rs`. Haremos referencia a ese archivo como simplemente `examples/init.rs` en el capítulo.

Existen dos tipos básicos de código Rust: programas ejecutables "binarios" y "bibliotecas". 
No desarrollaremos muchas bibliotecas a lo largo del libro. 
El código fuente de los programas reside en varios lugares:

* La aplicación `src/main.rs` se compilará y ejecutará directamente con `cargo embed` o `cargo
  run`. No hace falta ninguna configuración especial.

* Un fichero llamado `examples/foo.rs` tiene que ser ejecutado y compilado mediante `cargo embed --example foo` o
  `cargo run --example foo`.
  
* Un programa denominado `src/bin/bar.rs` puede ser ejecutado con `cargo embed --bin bar` o
  `cargo run --bin bar`.

Es un poco confuso, pero es la forma estándar de Cargo.

Vamos a avanzar en todo esto.
