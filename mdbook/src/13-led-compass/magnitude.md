# Magnitud

¿Cómo es de fuerte el campo magnético del planeta? De acuerdo a la documentación que tenemos, el método [`magnetic_field()`] nos devuelve los valores `x` `y` `z` en nanoteslas. Eso significa que lo único que tenemos que calcular para obtener la magnitud del campo magnético es la magnitud del vector 3D que describen dichos valores. Como recordamos de la escuela, esto es simplemente:


``` rust
use libm::sqrtf;
let magnitude = sqrtf(x * x + y * y + z * z);
```

[`magnetic_field()`]: https://docs.rs/lsm303agr/1.1.0/lsm303agr/struct.Lsm303agr.html#method.magnetic_field

Rust no tiene funciones en coma flotante como `sqrtf()` en el crate `core`, por lo que nuestro programa `no_std` tiene que usar una implementación de algún lugar. Importamos el crate [libm] para esto. 

[libm]: https://crates.io/crates/libm

Juntándolo todo en un programa (`examples/magnitude.rs`):

``` rust
{{#include examples/magnitude.rs}}
```

Lo ejecutamos con `cargo run --example magnitude`.


Este programa debería mostrar la magnitud (fuerza) del campo magnético del planeta en nanoteslas (`nT`) y milligauss (`mG`, donde 1 `mG` = 100 `nT`). La magnitud del campo magnético de la Tierra está en el rango de `250 mG` a `650 mG` (la magnitud varía dependiendo de tu ubicación geográfica), así que idealmente deberíamos ver un valor en ese rango. El valor probablemente estará bastante alejado porque el sensor no ha sido calibrado: ver [apéndice 3] para calibración. Con calibración, veo una magnitud de alrededor de `340 mG`.

[apéndice 3]: ../appendix/3-mag-calibration/README.md

Algunas cuestiones:
- Sin mover el sensor, ¿qué valor se ve? ¿Aparece siempre el mismo valor?
- Si rotamos la placa, ¿Cómo cambia el valor? ¿Debería cambiar?
