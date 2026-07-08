# El reto

¡Convirtamos el MB2 en una sirena! Pero no en una sirena cualquiera, sino en una
sirena controlada por interrupciones. De esa forma, podremos activar la sirena
y el resto de nuestro programa seguirá ejecutándose, sin tenerla en cuenta.

Haz que tu sirena varíe el tono desde 440 Hz hasta 660 Hz y viceversa
durante un segundo. El programa principal debe poner en marcha la
sirena, a continuación mostrar una cuenta atrás de diez segundos del 10 al 1, y luego
detener la sirena y mostrar "¡lanzamiento!". El programa principal no debe
interferir con la sirena durante la cuenta atrás; simplemente debe
funcionar por interrupciones.

*Pista:* A mí me resultó más fácil utilizar una estructura global bloqueada `Siren`
que gestionara el estado de la sirena y los periféricos
necesarios para su funcionamiento.

Este es un programa sofisticado que introduce muchas
ideas nuevas. No te sorprendas si te lleva un rato entenderlo
del todo.