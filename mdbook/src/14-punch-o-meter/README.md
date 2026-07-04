# Medidor de puñetazos

En esta sección vamos a experimentar con el acelerómetro que lleva la placa.

¿Qué vamos a construir esta vez? ¡Un "medidor de puñetazos"! Mediremos la potencia de tus golpes. Bueno, en realidad, la aceleración máxima que puedas alcanzar, ya que la aceleración es lo que miden los acelerómetros.
Sin embargo, la fuerza y la aceleración son proporcionales, por lo que es una buena aproximación.

Como ya sabemos por capítulos anteriores, el acelerómetro está integrado en el paquete LSM303AGR.
Al igual que el magnetómetro, se puede acceder a él mediante el bus I2C. El acelerómetro también tiene el mismo sistema de coordenadas que el magnetómetro.