# Nordic nRF52833 (el micro "nRF52", micro:bit v2)

El MCU tiene 73 contactos metálicos diminutos justo debajo de él (es un chip llamado [aQFN73]).
Estos contactos están conectados a **pistas**, las pequeñas "carreteras" que actúan como los cables que conectan los componentes entre sí en la placa. El MCU puede alterar dinámicamente las propiedades eléctricas de los pines. Esto funciona de manera similar a un interruptor de luz, alterando cómo fluye la corriente eléctrica a través de un circuito. Al habilitar o deshabilitar el flujo de corriente eléctrica a través de un pin específico, se puede encender o apagar un LED conectado a ese pin (a través de las pistas).

Cada fabricante utiliza un esquema de numeración de piezas diferente, pero muchos permitirán determinar información sobre un componente simplemente mirando el número de pieza. Al observar el número de pieza de nuestro MCU encontramos `N52833 QIAAA0 2024AL`: probablemente no puedas verlo a simple vista, pero está en el chip. (Si tienes una revisión posterior de MB2, el número puede variar un poco. Esto no es un problema. El número `N52833` no debería faltar.) La `N` al principio nos indica que esta es una pieza fabricada por [Nordic Semiconductor]. Buscando el número de pieza en su sitio web encontramos rápidamente la [página del producto]. Allí vemos que el principal componente de comunicaciones de nuestro chip es un "Bluetooth Low Energy and 2.4 GHz SoC" (SoC es la abreviatura de "System on a Chip"), lo que explica el RF en el nombre del producto, ya que RF es la abreviatura de radiofrecuencia. Si buscamos un poco en la documentación del chip vinculada en la [página del producto] encontramos la [especificación del producto] que contiene el capítulo 10 "Información de pedido" dedicado a explicar la extraña nomenclatura del chip. Aquí encontramos que:


[aQFN73]: https://en.wikipedia.org/wiki/Flat_no-leads_package
[Nordic Semiconductor]: https://www.nordicsemi.com/
[página del producto]: https://www.nordicsemi.com/products/nrf52833
[especificación del producto]: https://docs-be.nordicsemi.com/bundle/ps_nrf52833/attach/nRF52833_PS_v1.7.pdf

- El valor `N52` es el número de serie del MCU, indica que es de la famila `nRF52` de MCUs
- El valor `833` es el código de pieza
- El valor `QI` es el código de paquete, abreviatura de `aQFN73`
- El valor `AA` es el código de variante, que indica la cantidad de RAM y memoria flash que tiene la MCU, en nuestro caso, 512 kilobytes de memoria flash y 128 kilobytes de RAM
- El valor `A0` es el código de compilación, que indica la versión del hardware (`A`) y la configuración del producto (`0`)
- El valor `2024AL` es un código de seguimiento, por lo que puede variar en el chip

La [especificación del producto] contiene mucha más información útil sobre el chip: por ejemplo, que se trata de un procesador Arm® Cortex™-M4 de 32 bits.


## ¿Arm? ¿Cortex-M4?

Si nuestro chip lo fabrica Nordic, ¿quién es Arm? Y si nuestro chip es el
nRF52833, ¿qué es el Cortex-M4?

Te puede sorprender al enterar de que, aunque los chips "basados en Arm" son bastante populares, la empresa detrás de la marca "Arm" ([Arm Holdings]) no fabrica realmente chips. En cambio, su modelo de negocio es simplemente *diseñar* partes de chips. Luego, licencian estos diseños a los fabricantes, quienes a su vez implementan los diseños (quizás con algún ajuste propio) en forma de hardware que pueden ser vendidos. La estrategia de Arm aquí es diferente a la de empresas como Intel, que tanto diseña *como* fabrica sus chips.

Arm licencia una gran variedad de diseños diferentes. Su familia de "Cortex-M" se utiliza principalmente como núcleo en microcontroladores. Por ejemplo, el Cortex-M4 (el núcleo en el que se basa nuestro chip) está diseñado para bajo costo y consumo de energía reducido. El Cortex-M7 es de mayor coste, pero con más características y rendimiento.

Afortunadamente, no se necesita saber demasiado sobre los diferentes tipos de procesadores o diseños de Núcleos para el propósito de este libro. Sin embargo, ahora conoces un poco más la terminología del dispositivo. Mientras se trabaja específicamente con un nRF52833, es posible que sea necesario leer documentación y utilizar herramientas para chips basados en Cortex-M, ya que el nRF52833 tie en su diseño un Cortex-M.


[Arm Holdings]: https://www.arm.com/
