# El reto

Para mantener el software simple, asumiremos que se golpea con una tabla paralela al suelo. Para medir la magnitud del golpe, necesitaremos tener en cuenta tanto la aceleración X como Y (ignorando Z, ya que solo refleja la gravedad). Para hacerlo aún más fácil, asumiremos que se sostiene la tabla con el botón B cerca de nosotros y el botón A más lejos, Después lo movemos alejándolo de nosotros. Esto significa que estamos golpeando en la dirección positiva X.


<p align="center">
<img class="white_bg" title="Punch Direction" src="../assets/mb2-punch-axis.svg" width="500" alt="Puch direction" />
</p>

A continuación, se muestra un ejemplo de cómo podría ser el algoritmo de la aplicación:

- Por defecto, la aplicación no está "observando" la aceleración de la tabla.
- Cuando se detecta una aceleración X significativa (es decir, la aceleración supera algún umbral), la aplicación debe iniciar una nueva medición.
- Durante ese intervalo de medición, la aplicación debe realizar un seguimiento de la aceleración máxima observada.
- Después de que finalice el intervalo de medición, la aplicación debe informar la aceleración máxima observada. Se puede imprimir el valor utilizando la macro `rprintln!`.


Pruébalo y cuéntame con qué fuerza eres capaz de dar un puñetazo `;-)`.

> **NOTA** Hay un API adicional que debería ser útil para esta tarea que aún no hemos discutido: la de [`set_accel_scale`] que se necesita para medir valores altos de g.
>
> [`set_accel_scale`]: https://docs.rs/lsm303agr/1.1.0/lsm303agr/struct.Lsm303agr.html#method.set_accel_scale
