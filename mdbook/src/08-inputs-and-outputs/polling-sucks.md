# El sondeo apesta, todavía

Efectivamente, los intermitentes generalmente parpadean, ¿verdad? ¿Cómo podríamos extender nuestro programa para hacer parpadear el LED del intermitente cuando se presiona un botón? Sabemos cómo hacer parpadear un LED desde nuestro programa de Hola Mundo; encendemos el LED, esperamos un tiempo y luego lo apagamos. Pero, ¿cómo podemos hacer esto en nuestro bucle mientras también comprobamos las pulsaciones de los botones? Podríamos intentar algo como esto:


```rust
    loop {
        if button_a.is_low().unwrap() {
            // Blink left arrow
            display.show(&LEFT_ARROW);
            timer.delay_ms(500_u32);
            display.show(&BLANK);
            timer.delay_ms(500_u32);
        } else if button_b.is_low().unwrap() {
            // Blink right arrow
            display.show(&RIGHT_ARROW);
            timer.delay_ms(500_u32);
            display.show(&BLANK);
            timer.delay_ms(500_u32);
        } else {
            display.show(&BLANK);
        }
        timer.delay_ms(10_u32);
    }
```

¿Puedes observar el problema? Estamos intentando hacer dos cosas al mismo tiempo:

1. Comprobar los botones
2. Parpadear el LED

Pero el microcontrolador solo es capaz de realizar una. Si presionamos un botón durante el retraso de parpadeo, el procesador no podrá responder hasta que el retraso haya terminado y el bucle comience de nuevo. Como resultado, obtenemos un programa apenas interactivo (pruébalo y verás lo lento que es el botón).

Un programa un poco más inteligente sabría que el procesador no está haciendo nada mientras se ejecuta el retraso del parpadeo. El programa podría hacer otras cosas mientras espera a que termine el retraso, como verificar las pulsaciones de los botones.

## Superbucles

El término de *superbucle* en sistemas embebidos se utiliza para referirse a un bucle de control principal que hace varias cosas en secuencia. Es la extensión natural del bucle de espera activa que hemos estado usando hasta ahora. 
Para gestionar la lógica que simule la ejecución de múltiples tareas sucediendo al mismo tiempo, necesitamos ser un poco más inteligentes a la hora de estructurar el programa para que podamos gestionar razonablemente rápido los eventos.

En el caso del programa de los intermitentes, donde queremos hacer parpadear los LED cuando se presiona un botón, y dejar de parpadear cuando se suelta el botón, podemos crear una "máquina de estados" para representar los diversos estados del programa. Tenemos tres estados para los botones:

1. Ningún botón está presionado.
2. El botón A está presionado.
3. El botón B está presionado.

También tenemos tres estados para la pantalla:

1. No se enciende nada.
2. Estamos en el estado de parpadeo activo para la pantalla (los LED están encendidos).
3. Estamos en el estado de parpadeo inactivo para la pantalla (los LED están apagados y esperando a ser encendidos una vez que termine el período de parpadeo).

Dado que queremos garantizar la respueta del sistema, tenemos que combinar estos diferentes estados. Para representar de forma completa la máquina de estado de nuestro programa, tendríamos lo siguiente:

1. Ningún botón está presionado.
2. El botón A está presionado y estamos en el estado de parpadeo activo (la flecha izquierda se muestra en la pantalla).
3. El botón A está presionado y estamos en el estado de parpadeo inactivo (nada se muestra en la pantalla).
4. El botón B está presionado y estamos en el estado de parpadeo activo (la flecha derecha se muestra en la pantalla).
5. El botón B está presionado y estamos en el estado de parpadeo inactivo (nada se muestra en la pantalla).

Cuando se pulse cualquiera de los botones por primera vez y pasemos del estado (1) al estado (2) o al (4), inicializaremos un temporizador que contará a partir del momento en que se pulse el botón. Cuando alcance un valor umbral (por ejemplo, medio segundo) y los botones sigan pulsados, pasaremos al estado (3) o al (5), respectivamente, y reinicializaremos el contador de tiempo.  Cuando el temporizador vuelva a alcanzar el valor umbral, volveremos al estado (2) o (4), de nuevo.  Si en cualquier momento durante los estados (2), (3), (4) o (5) vemos que el botón ya no está pulsado, volveremos al estado (1).

El superbucle se encargará de gestionar el flujo y muestreará continuamente el estado de los botones, comparando el tiempo actual con el umbral, y cambiará al estado correspondiente si se cumplen las condiciones anteriores.

Hemos implementado este superbucle como una demostración (en  `examples/blink-held.rs`), pero con la máquina de estados simplificada solo para hacer parpadear un LED cuando se mantiene presionado el botón A.

```rust
{{#include examples/blink-held.rs}}
```

Sigue siendo un poco complicado. El retardo de 10ms es más que adecuado para detectar los cambios en la pulsación de los botones por experiencia previa.

Los Superbucles funcionan y son muy utilizados en los sistemas embebidos, pero el programador debe tener cuidado para mantener un alto grado de respuesta a los eventos. 
Obserevamos cómo el programa de superbucle es diferente del ejemplo de sondeo simple anterior. Cualquier transición de estado en el superbucle, tal como se escribió anteriormente, debería tomar una cantidad de tiempo bastante pequeña (por ejemplo, ya no se producen retrasos que puedan bloquear el procesador durante largos periodos de tiempo y hacernos perder algún evento). 
No siempre es fácil transformar un programa de sondeo simple en un superbucle donde todas las transiciones de estado sean rápidas y relativamente no bloqueantes. En estos casos, tendremos que confiar en técnicas alternativas para manejar los diferentes eventos que se ejecutan al mismo tiempo.


## Concurrencia

Hacer varias cosas a la vez se llama programación *concurrente*. La concurrencia aparece en muchos lugares en la programación, pero especialmente en los sistemas embebidos. Hay toda una serie de técnicas para implementar sistemas que interactúan con periféricos mientras mantienen un alto grado de respuesta (por ejemplo, manejo de interrupciones, multitarea cooperativa, colas de eventos, etc.). Exploraremos algunas de estas técnicas en capítulos posteriores.

Hay una buena introducción a la concurrencia [aquí] que podríamos leer antes de continuar.

[aquí]: https://docs.rust-embedded.org/book/concurrency/index.html

Por ahora, echemos un vistazo en profundidad a lo que sucede cuando llamamos a `button_a.is_low()` o `display_pins.row1.set_high()`.