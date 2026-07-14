# Adenda: PWM

Un último comentario antes de continuar.

Las interrupciones son bastante costosas. El procesador debe finalizar o abortar la instrucción que se está ejecutando en ese momento, guardar después el estado necesario para reanudar la ejecución y, a continuación, llamar a un controlador de interrupciones. Todo esto requiere unos cuantos ciclos de CPU de un valioso tiempo de ejecución.

Tal y como está escrita la solución de la sección anterior, se necesitarán dos interrupciones por ciclo de salida del altavoz. Eso supone alrededor de 1000 interrupciones por segundo. En un procesador como nuestro nRF52833, eso funciona bien.

El nRF52833 cuenta con un periférico integrado que podría reducir considerablemente la frecuencia de interrupciones de nuestra sirena. La unidad de modulación por ancho de pulso (PWM) puede, entre otras cosas, generar ciclos en el pin del altavoz a una frecuencia controlada por un registro PWM. Esto podría utilizarse para generar la onda cuadrada básica que se utiliza en la sirena. Seguiríamos necesitando una interrupción cada vez que quisiéramos cambiar la frecuencia, pero esto podría suponer unas 10 interrupciones por segundo en lugar de 1000.

No utilicé la unidad PWM en mi solución. Esto se debió en parte a que quería centrarme en las interrupciones. Sin embargo, otra razón importante fue que la unidad PWM del nRF52833 es bastante complicada y difícil de entender. Conseguir que algo funcione de forma sencilla en el entorno limitado del "bare-metal" siempre resulta atractivo.

Si te apetece un reto, te animo a que intentes utilizar la unidad PWM para tu sirena.
