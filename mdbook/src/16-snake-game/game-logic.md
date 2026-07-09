# Lógica del Juego

El primer módulo que vamos a crear es la lógica del juego. Probablemente, ya conozcamos el juego de la [serpiente], pero
si no es así, la idea básica es que el jugador guíe a una serpiente por una cuadrícula en 2D. En todo momento, hay
alguna "comida" en una ubicación aleatoria de la cuadrícula y el objetivo del juego es conseguir que la serpiente
"coma" tanta como sea posible. Cada vez que la serpiente come, aumenta de longitud. El jugador pierde
si la serpiente se choca contra su propia cola.

[serpiente]: https://en.wikipedia.org/wiki/Snake_%28video_game_genre%29

En algunas variantes del juego, el jugador también pierde si la serpiente choca contra el borde del tablero,
pero, dado el reducido tamaño de nuestro tablero, vamos a aplicar una regla de "reenvío": si la serpiente
se sale por uno de los bordes del tablero, continuará desde el borde opuesto.

## El módulo `game` 
Construiremos la lógica del juego en el módulo `game`. 

### Coordenadas
Lo primero es definir el sistema de coordenadas para nuestro juego (`src/game/coords.rs`).

```rust
{{#include src/game/coords.rs}}
```

Usamos una estructura `Coords` para gestionar una posición en la cuadrícula. 
Dado que `Coords` solo contiene dos enteros, le indicamos al compilador que derive una implementación del trait `Copy` 
para que podamos pasar estructuras `Coords` sin tener que preocuparnos por la propiedad.


### Generación aleatoria de coordenadas

Definimos una función asociada, `Coords::random`, que nos dará una posición aleatoria en la cuadrícula. Usaremos esto más adelante para determinar dónde colocar la comida de la serpiente.

Para generar coordenadas aleatorias, necesitamos una fuente de números aleatorios. El nRF52833 tiene un periférico generador de números aleatorios por hardware (HWRNG), documentado en la sección 6.19 de la [especificación del nRF52833]. La HAL nos proporciona una interfaz sencilla para el HWRNG a través de la estructura `microbit::hal::rng::Rng`. El HWRNG puede no ser lo suficientemente rápido para un juego; también es conveniente para las pruebas poder replicar la secuencia de números aleatorios producida por el generador entre ejecuciones, lo cual es imposible para el HWRNG por diseño. Por lo tanto, también definimos un generador de números pseudoaleatorios (PRNG [pseudo-random]). El PRNG utiliza un algoritmo [xorshift] para generar valores pseudoaleatorios `u32`. El algoritmo es básico y no es criptográficamente seguro, pero es eficiente, fácil de implementar y lo suficientemente bueno para nuestro humilde juego de la serpiente. 
 La estructura `Prng` requiere un valor inicial de semilla, que obtenemos del periférico RNG.


[especificación del nRF52833]: https://docs-be.nordicsemi.com/bundle/ps_nrf52833/attach/nRF52833_PS_v1.7.pdf
[pseudo-random]: https://en.wikipedia.org/wiki/Pseudorandom_number_generator
[xorshift]: https://en.wikipedia.org/wiki/Xorshift

Todo esto se materializa en `src/game/rng.rs`.

```rust
{{#include src/game/rng.rs}}
```

### Movimiento
También necesitamos definir algunas `enum`s (enumeraciones) que nos ayuden a gestionar el estado del juego: dirección de movimiento, dirección de giro, el estado actual del juego y el resultado de un "paso" particular en el juego (es decir, un único movimiento de la serpiente). El fichero `src/game/movement.rs` contiene estas implementaciones.

```rust
{{#include src/game/movement.rs}}
```

### La Serpiente (*A Snaaake!*)

La serpiente la modelaremos con una estructura `Snake`. Esta mantiene un registro de las coordenadas ocupadas por la serpiente y su dirección de movimiento. Utilizamos una cola (`heapless::spsc::Queue`) para mantener el orden de las coordenadas y un conjunto hash (`heapless::FnvIndexSet`) para permitir una detección rápida de colisiones. La `Snake` tiene métodos que le permiten moverse. Esto se encuentra en `src/game/snake.rs`.


```rust
{{#include src/game/snake.rs}}
```

### El módulo Game superior

La estructura `Game` es la que mantiene el estado del juego. Contiene un objeto `Snake`, las coordenadas actuales de la comida, la velocidad del juego (que se utiliza para determinar el tiempo que transcurre entre cada movimiento de la serpiente), el estado del juego (si el juego está en curso o si el jugador ha ganado o perdido) y la puntuación del jugador.

Tiene asociados métodos para manejar cada paso del juego, determinando el siguiente movimiento de la serpiente y actualizando el estado del juego en consecuencia. También contiene dos métodos: `game_matrix` y `score_matrix`, que devuelven matrices 2D de valores que pueden usarse para mostrar el estado del juego o la puntuación del jugador en la matriz de LED (como veremos más adelante).

La estructura `Game` se encuentra en `src/game.rs`.

```rust
{{#include src/game.rs}}
```

Lo siguiente es dotar de la habilidad de controlar los movimientos de la serpiente.
