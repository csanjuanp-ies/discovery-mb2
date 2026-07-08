# El altavoz del MB2

La MB2 tiene un altavoz integrado: el dispositivo cuadrado grande de color negro con la etiqueta "SPEAKER" situado en el centro de la parte trasera de la placa.

El altavoz funciona moviendo el aire en respuesta a un pin GPIO: cuando el pin del altavoz está alto (3,3 V), un diafragma en su interior —el "cono del altavoz"— se empuja hasta el final hacia fuera; cuando el pin del altavoz está bajo (GND), se retrae completamente. A medida que el aire sale y vuelve a entrar, fluye hacia dentro y hacia fuera por el pequeño orificio rectangular —el "puerto del altavoz"— situado en el lateral del dispositivo.  Si esto se hace con la suficiente rapidez, los cambios de presión producirán un sonido.

<img class="white_bg" height="350" title="Speaker" src="../assets/speaker.svg" alt="Speaker"/>

Con el hardware adecuado que lo accione, este cono de altavoz podría, de hecho, desplazarse a cualquier posición dentro de su rango con una corriente adecuada. Esto permitiría una reproducción bastante buena de cualquier sonido, como un altavoz "normal". Por desgracia, las limitaciones del hardware del MB2 que controla el altavoz hacen que solo estén fácilmente disponibles las posiciones de máxima entrada y máxima salida.

Empujemos el cono del altavoz hacia fuera y luego hacia dentro 220 veces por segundo. Esto producirá una onda de presión "cuadrada" de 220 ciclos por segundo. La unidad "ciclos por segundo" es el hercio; produciremos un
tono de 220 Hz (un "La" musical), que no resulta desagradable en este altavoz de sonido agudo.

Haremos que nuestro tono suene durante cinco segundos y luego se detenga. Es importante recordar que el programa está almacenado en la memoria flash del MB2: el tono volverá a sonar cada vez que reiniciemos o incluso encendamos el MB2. Si dejamos que el tono suene indefinidamente, este comportamiento puede resultar bastante molesto.

Aquí está el código (`examples/square-wave.rs`).

```rust
{{#include examples/square-wave.rs}}
```