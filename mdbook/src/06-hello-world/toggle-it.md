# Parpadeándo

Vamos a encencer y apagar el LED repetidamente. Así es como se hace parpadear, ¿verdad?


En el fichero `examples/blink.rs` encontraremos el código para hacer parpadear el LED.
Hemos decidido usar siguiente LED en lugar del original (dejando encendido el primero), pero eso es un cambio fácil de hacer.

```rust
{{#include examples/fast-blink.rs}}
```
Estamos usando el crate `embedded-hal` para proporcionar los traits de Rust necesarios para encender y apagar el LED. Esto significa que esta parte del código es portable a cualquier HAL de Rust que implemente los traits de `embedded-hal`, como el nuestro.


Pero, espera: ¡ninguno de los LED está parpadeando! El segundo LED es un poco más tenue que el primero, pero ambos están encendidos siempre... 
o ¿lo están? De fábrica, el MB2 ejecuta 64 *millones* de instrucciones por segundo. Supongamos que se necesitan unas pocas docenas de instrucciones para encender o apagar el LED. (posiblemente en una compilación en modo debug, aunque muchas menos en modo release. Aunque los pines tardan un tiempo en cambiar de estado. No lo sé.) De todos modos, ese segundo LED en realidad se está encendiendo y apagando cientos de miles de veces — quizás millones de veces — cada segundo. Nuestro ojo simplemente no puede seguir el ritmo.

Tendríamos que ralentizar el programa para poder ver el LED parpadear. De hecho, hacer esperar al programa entre cambios es la parte más difícil.
