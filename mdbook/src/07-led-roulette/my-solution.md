# La solución

¿Qué solución se te ocurrió?

Aquí tienes la mía. Probablemente sea una de las formas más simples (pero por supuesto no la más bonita) de generar la matriz requerida:

``` rust
{{#include src/main.rs}}
```
Una cosa más, ¡comprueba que tu solución también funciona cuando se compila en modo "release"!

``` console
$ cargo embed --release
```
Si es necesesario depurar el programa en modo "release" el comando de GDB es un poco diferente:

``` console
$ gdb ../../../target/thumbv7em-none-eabihf/release/led-roulette
```

El compilador de Rust modifica las instrucciones generadas para una compilación release (a veces mucho) tratando de hacer el código más rápido y/o más pequeño. Desafortunadamente, GDB tiene dificultades para entender lo que está pasando después de la optimización. Como resultado, depurar compilaciones release con GDB puede ser difícil.

El tamaño del binario generado es algo a tener en cuenta siempre, hay que recordar que tenemos recursos limitados. ¿Qué tamaño tiene tu solución? Puedes comprobarlo usando el comando `size` sobre el binario de release:

``` console
$ cargo size --release -- -A
    Finished release [optimized + debuginfo] target(s) in 0.02s
led-roulette  :
section              size        addr
.vector_table         256         0x0
.text                6332       0x100
.rodata               648      0x19bc
.data                   0  0x20000000
.bss                 1076  0x20000000
.uninit                 0  0x20000434
.debug_loc           9036         0x0
.debug_abbrev        2754         0x0
.debug_info         96460         0x0
.debug_aranges       1120         0x0
.debug_ranges       11520         0x0
.debug_str          71325         0x0
.debug_pubnames     32316         0x0
.debug_pubtypes     29294         0x0
.Arm.attributes        58         0x0
.debug_frame         2108         0x0
.debug_line         19303         0x0
.comment              109         0x0
Total              283715
```

Los valores pueden variar dependiendo de cómo se compile el código: esto es normal.

¿Cómo se lee esta salida? La sección `text` contiene las instrucciones del programa. La de `rodata` los datos de solo lectura almacenados con las instrucciones del programa. Las partes `data` y `bss` almacenan variables asignadas estáticamente en RAM (variables `static`). 
Si recordamos la especificación del microcontrolador de la MB2, veremos que su memoria flash es menos del doble del tamaño de este binario extremadamente simple: ¿puede ser esto correcto? Como vemos en las estadísticas, la mayor parte del binario está realmente compuesto por secciones relacionadas con la depuración. Sin embargo, esas no se pasan al microcontrolador, después de todo no son necesarias para la ejecución.
