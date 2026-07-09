# Juego de la serpiente

Ahora vamos a implementar un juego básico de [serpiente](https://en.wikipedia.org/wiki/Snake_(video_game_genre)) al que podrás jugar en un MB2 utilizando su matriz de LED de 5×5 como pantalla y sus dos botones como mandos. Para ello, nos basaremos en algunos de los conceptos tratados en los capítulos anteriores de este libro y también aprenderemos sobre algunos periféricos y conceptos nuevos.

## Modularidad

El código fuente que vemos aquí es más modular de lo que probablemente debería ser. Esta modularidad tan detallada nos permite analizar el código fuente poco a poco. Construiremos el código de abajo hacia arriba: primero crearemos tres módulos —`game`, `controls` y `display`— y, a continuación, los combinaremos para crear el programa final. Cada módulo tendrá un archivo fuente de nivel superior y uno o más archivos fuente incluidos: por ejemplo, el módulo `game` estará formado por `src/game.rs`, `src/game/coords.rs`, `src/game/movement.rs`, etc. La instrucción `mod` de Rust se utiliza para combinar los distintos componentes del módulo. *El lenguaje de programación Rust* ofrece una buena [descripción] del sistema de módulos de Rust.

[descripción]: https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html
