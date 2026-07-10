# El reto

El reto para este capítulo es construir una pequeña aplicación que se comunique con el mundo exterior a través de la interfaz serie introducida en el último capítulo. Debe esperar recibir los comandos "mag" para el magnetómetro así como "acc" para el acelerómetro desde el puerto serie. Luego, debería ser capaz de enviar los datos correspondientes del sensor al puerto serie en respuesta.

En esta ocasión no se proporcionará ningúna plantilla con código, ya que todo lo que se necesita ya se encuentra en el [UART](../11-uart/README.md) y en este capítulo. No obstante, aquí hay algunas pistas:

-   Quizá te interese utilizar `core::str::from_utf8` para convertir los bytes del búfer en un `&str`, ya que necesitamos compararlos con `"mag"` y `"acc"`.

-   TEndremos que leer la documentación sobre la API y el funcionamiento del magnetómetro. Aunque el crate `lsm303agr` proporciona la interfaz, la [ficha técnica del LSM303AGR](https://www.st.com/resource/en/datasheet/lsm303agr.pdf) detalla los parámetros de medición del campo magnético del sensor. En las páginas 13-15 están las características del sensor y lo que es más importante, las páginas 66-67 para ver el formato del registro de salida.

