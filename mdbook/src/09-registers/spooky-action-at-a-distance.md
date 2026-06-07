# Acción fantasmagórica a distancia

`OUT` no es el único registro que puede controlar los pines del puerto `P0`. El registro `OUTSET` también te permite cambiar el valor de los pines, al igual que `OUTCLR`. Sin embargo, `OUTSET` y `OUTCLR` no  permiten recuperar el estado de salida del puerto `P0`.


`OUTSET` está documentado en [Especificación del producto]:

> Subsection 6.8.2.2. OUTSET - Page 145

Miremos el programa de abajo. La clave de este programa es `fn print_out`. Esta función imprime el valor actual en `OUT` a la consola `RTT` (`examples/spooky.rs`):


``` rust
{{#include examples/spooky.rs}}
```

Si ejecutas el programa, verás algo como esto en la consola `RTT`:

``` console
$ cargo embed
# cargo-embed's console
(..)
15:13:24.055: P0.OUT = 0x000000
15:13:24.055: P0.OUT = 0x200000
15:13:24.055: P0.OUT = 0x280000
15:13:24.055: P0.OUT = 0x080000
15:13:24.055: P0.OUT = 0x000000
```

Hay efectos secundarios, aunque estamos leyendo la misma dirección varias veces sin modificarla realmente, vemos su valor cambiar cada vez que se escribe en `OUTSET` o `OUTCLR`.


[Especificación del producto]: https://docs-be.nordicsemi.com/bundle/ps_nrf52833/attach/nRF52833_PS_v1.7.pdf
