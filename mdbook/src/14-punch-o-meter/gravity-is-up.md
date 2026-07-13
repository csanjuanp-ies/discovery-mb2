# ¿Ha subido la gravedad?

¿Qué es lo primero que haremos?

¡Hacer una comprobación de coherencia!

A estas alturas ya deberíamos ser capaces de escribir un programa que muestre continuamente los datos del acelerómetro en la
consola RTT, tal y como se explica en el [capítulo sobre I2C](../12-i2c/README.md). El mío se encuentra en `examples/show-accel.rs`, dentro de ese capítulo. ¿Observas algo interesante incluso cuando se sostiene la placa en paralelo al suelo con la parte trasera hacia arriba? (Recuerda que el acelerómetro está montado en la parte trasera de la MB2, por lo que, al sostenerla boca abajo de esta forma, el eje Z apunta hacia arriba).

Lo que se debería ver al sujetarla de esta forma es que tanto los valores de X como los de Y están bastante cerca de 0, mientras que el valor de Z ronda los 1000. Lo cual es extraño: la placa no se está moviendo, pero su aceleración es distinta de cero. ¿Qué está pasando? Esto debe tener que ver con la gravedad, ¿no? Porque la aceleración de la gravedad es de `1 g` (ajá, `1 g` = -1000 según el acelerómetro). Pero la gravedad atrae los objetos hacia abajo, por lo que la aceleración a lo largo del eje Z debería ser positiva, no negativa.

¿El programa ha invertido el eje Z? No, podemos probar a girar la placa para alinear la gravedad
con el eje X o Y, pero la aceleración medida por el acelerómetro siempre apunta hacia arriba.

Lo que ocurre es que el acelerómetro mide la *aceleración propia*, no
la aceleración que *se* observa. Esta aceleración propia es la aceleración de la placa tal y como
la ve un observador que se encuentra en caída libre. Un observador en caída libre se desplaza hacia el centro de la Tierra con una aceleración de `1 g`; desde su punto de vista, la Micro:bit se está moviendo en realidad hacia arriba (alejándose del centro de la Tierra) con una aceleración de `1 g`. Y por eso la aceleración propia apunta hacia arriba. Esto también significa que, si el chip estuviera en caída libre, el acelerómetro indicaría una aceleración propia de cero. Por favor, no intentes esto en casa. O hazlo, si estás dispuesto a arriesgar la MB2 dejándola caer.

Si, la física es difícil. Vamos a ello.

