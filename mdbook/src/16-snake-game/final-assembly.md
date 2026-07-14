# Juego de la serpiente: Paso final

El código del archivo `src/main.rs` reúne todos los mecanismos que hemos visto anteriormente para crear el juego definitivo.

```rust
{{#include src/main.rs}}
```

Tras inicializar la placa, el temporizador y el generador aleatorio (RNG), creamos una estructura `Game` y un `Display` del módulo `microbit::display::blocking`.

En el "bucle de juego" (que se ejecuta dentro del "bucle principal" que colocamos en nuestra función `main`), realizamos repetidamente los siguientes pasos:

1. Obtenemos una matriz de 5×5 bytes que represente la cuadrícula. El método `Game::get_matrix` toma tres argumentos enteros (que deben estar comprendidos entre 0 y 9, ambos inclusive) que, en última instancia, representarán el nivel de brillo con el que deben mostrarse la cabeza, la cola y la comida.

2. Mostramos la matriz durante un tiempo determinado por el método `Game::step_len_ms`. Tal y como está implementado actualmente, este método establece básicamente un intervalo de 1 segundo entre pasos, que se reduce en 200 ms cada vez que el jugador suma 5 puntos (comer 1 trozo de comida = 1 punto), con un mínimo de 200 ms.

3. Comprobamos el estado del juego. Si es `Ongoing` (que es su valor inicial), ejecuta un paso del juego y actualiza el estado del juego (incluida la propiedad `status`). En caso contrario, el juego ha terminado, por lo que hacemos parpadear la imagen actual tres veces, a continuación, mostramos la puntuación del jugador (representada como un número de LEDs iluminados que se corresponden con la puntuación) y salimos del bucle del juego.

El bucle principal se limita a ejecutar el bucle del juego repetidamente, restableciendo el estado del juego tras cada iteración.