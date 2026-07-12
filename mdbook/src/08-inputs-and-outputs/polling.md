# Polling
Ya que sabemos leer entradas GPIO, consideremos cómo podríamos usar estas lecturas de manera práctica. Supongamos que queremos que nuestro programa encienda un LED cuando se presione el Botón A y lo apague cuando se presione el Botón B. 
Podemos hacer esto sondeando el estado de ambos botones en un bucle y respondiendo en consecuencia cuando se detecte que un botón está presionado. Aquí está cómo podríamos escribir este programa:


```rust
{{#include examples/polling-led-toggle.rs}}
```

Este método de comprobación indefinida de la entrada se denomina sondeo (polling). Cuando hacemos esto, decimos que estamos *sondeando* la entrada. En este caso, miramos ambos botones A y B.

El sondeo es simple, pero nos permite hacer cosas interesantes con el mundo exterior. Es posible "sondear" en un bucle todas las entradas del dispositivo y responder a los resultados de alguna manera, una por una. 
Este método es conceptualmente muy simple y es un buen punto de partida para muchos proyectos. Pero pronto descubriremos por qué este mecanismo podría no ser el mejor método para todos (o incluso la mayoría) de los casos, pero es el primero que se tiene que estudiar.

**Nota**: "Sondear" se utiliza a menudo con dos significados distintos. El primero, "sondeo" se usa para referirse a preguntar (una vez) cuál es el estado de una entrada. El segundo significado, "sondeo", o quizás "bucle de sondeo", es más para referirse a comprobar (indefinidamente) cuál es el estado de una entrada mediante repetición como el que usamos arriba. Esta segunda acepción se utiliza solo en los programas más simples, y rara vez lo implementaremos en producción (no es práctico como veremos pronto), así que generalmente cuando los ingenieros hablan sobre sondeo, se refieren al primero, es decir, preguntar (una vez) cuál es el estado de una entrada.