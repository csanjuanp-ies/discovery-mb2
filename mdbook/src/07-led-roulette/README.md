# Ruleta LED

De acuerdo, vamos a crear una "aplicación real". El objetivo es llegar a esta pantalla de luces giratorias:

<p align="center">
<video src="../assets/roulette_fast.mp4" width="500" loop="true" autoplay="true"/>
</p>

Como encender los LED por separado es bastante molesto (especialmente si tenemos que usar todos) podemos usar el crate BSP `microbit-v2`, discutido anteriormente, para trabajar con el "display" de LED del MB2. 
Funciona así (`examples/light-it-all.rs`):


```rust
{{#include examples/light-it-all.rs}}
```

El Array `light_it_all` contiene un 1 en la posición del LED que se encenderá y 0 donde estará apagado. 
La llamada a `show()` utiliza un temporizador para controlar el tiempo de encendido, una *copia* del array, y la duración en milisegundos que estarán iluminados.

