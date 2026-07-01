# LSM303AGR
Ambos sensores de movimiento en el micro:bit, el magnetómetro y el acelerómetro, están empaquetados en un solo componente: el circuito integrado LSM303AGR. Estos dos sensores se pueden acceder a través de un bus I2C. Cada sensor se comporta como un dispositivo I2C y tiene una *dirección* diferente.

Cada sensor tiene su propia memoria donde almacena los resultados de la detección de su entorno. Nuestra interacción con estos sensores consistirá principalmente en leer su memoria.


La memoria de estos sensores se modela como registros direccionables por bytes. Estos sensores también se pueden configurar; eso se hace escribiendo en sus registros. Entonces, en cierto sentido, estos sensores son muy similares a los periféricos de *dentro* del microcontrolador. La diferencia es que sus registros no están mapeados en la memoria del microcontrolador. En su lugar, sus registros deben accederse a través del bus I2C.

La fuente principal de documentación sobre el LSM303AGR es su [Hoja de Datos]. Léela para ver cómo se pueden acceder los registros de los sensores. Esa parte está en:


[Hoja de Datos]: https://www.st.com/resource/en/datasheet/lsm303agr.pdf

> Section 6.1.1 I2C Operation - Page 38 - LSM303AGR Data Sheet

La otra parte de la documentación importante para este libro es la descripción de los registros. La encontramos en:

> Section 8 Register description - Page 46 - LSM303AGR Data Sheet
