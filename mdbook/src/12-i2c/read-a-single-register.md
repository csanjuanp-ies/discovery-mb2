# Leer un único registro

¡Vamos a poner toda esta teoría en práctica!

Lo primero que necesitamos saber son las direcciones de destino tanto del acelerómetro como del magnetómetro del chip, estas se pueden encontrar en la hoja de datos del LSM303AGR en la página 39 y son:

- 0011001 para el acelerómetro
- 0011110 para el magnetómetro

> **NOTA** Estos son los 7 bits principales de la dirección, el 8º bit será el bit que determina si estamos realizando una lectura o escritura.

A continuación, necesitamos saber qué registro queremos leer. Muchos de los chips I2C proporcionan algún tipo de registro de identificación del dispositivo para que sus controladores puedan leerlo. Considerando los miles (o incluso millones) de chips I2C que existen, es muy probable que en algún momento se construyan dos chips con la misma dirección (después de todo, la dirección tiene "solo" 7 bits de ancho). Con este registro de ID del dispositivo, un controlador puede asegurarse de que realmente está hablando con un LSM303AGR y no con algún otro chip que simplemente tenga la misma dirección. Como se puede leer en la hoja de datos del LSM303AGR (específicamente en las páginas 46 y 61), existen dos registros: `WHO_AM_I_A` en la dirección `0x0f` y `WHO_AM_I_M` en la dirección `0x4f`, que contienen un conjunto de bits únicos para el dispositivo. (La "A" es para "Acelerómetro" y la "M" es para "Magnetómetro".)

Lo único que nos falta es la parte de software: necesitamos determinar qué API del `microbit` o de un crate HAL deberíamos usar para esto. Si leemos la [Especificación del Producto nRF52833], descubrimos que en realidad no tiene un periférico específico para I2C. En su lugar, tiene periféricos más generales compatibles con I2C llamados TWI ("Interfaz de Dos Hilos"), TWIM ("Interfaz de Dos Hilos Maestro") y TWIS ("Interfaz de Dos Hilos Esclavo"). Normalmente, operaremos en modo controlador y utilizaremos el TWIM, que admite "Easy DMA". El dispositivo TWI se usa principalmente para compatibilidad con dispositivos más antiguos.

Si miramos la documentación del módulo [`twi(m)`] del crate `microbit`y lo ponemos junto con toda la  información que hemos recopilado hasta ahora, terminamos con el siguiente fragmento de código para leer e imprimir los dos IDs de dispositivo (`examples/chip-id.rs`):

``` rust
{{#include examples/chip-id.rs}}
```

Aparte de la inicialización, este fragmento de código debería ser bastante fácil si se entendió el protocolo I2C como se describió antes. La inicialización aquí funciona de manera similar a la del capítulo de la UART. Pasamos el periférico así como los pines que se utilizan para comunicar con el chip al constructor; y luego la frecuencia a la que deseamos que opere el bus, en este caso 100 kHz (`K100`, ya que los identificadores no pueden comenzar con un dígito).

[Especificación del Producto nRF52833]: https://docs-be.nordicsemi.com/bundle/ps_nrf52833/attach/nRF52833_PS_v1.7.pdf
[`twi(m)`]: https://docs.rs/microbit-v2/0.11.0/microbit/hal/twim/index.html

## Testeo
Como siempre

```console
$ cargo embed --example chip-id
```

para probar nuestro pequeño programa de ejemplo.
