# Puerto Serie

Lo más cercano a un estándar universal de E/S para las placas embebidas modernas es el "puerto serie". Prácticamente, todos los microcontroladores tienen una forma de hacer que algunos de sus pines actúen como un puerto serie, y prácticamente todas las placas de microcontroladores hacen que estos pines sean fáciles de acceder. La MB2 no es una excepción.

En este capítulo, describiremos qué es un puerto serie. Luego mostraremos cómo configurar el ordenador con un "puerto serie virtual" usando USB y utilizar ese puerto virtual con "software de terminal" para hablar con el puerto serie en la MB2.

¿Qué es un [puerto serie]? De forma fácil, es un "mecanismo" por el que dos dispositivos intercambian datos bit a bit, uno en cada paso (*en serie*), usando una línea de datos en cada dirección (*full-duplex*) más una línea de tierra común. El puerto serie se denominó como "RS-232": más adelante veremos la historia de su desarrollo. Sin embargo, el protocolo "hablado" en las líneas de transmisión y recepción no tiene un nombre oficial que sepamos, es simplemente "serie" o tal vez "serie asíncrono" o "UART serie".

Para ser claros: la mayoría de los canales de comunicación en las computadoras modernas son serie. USB (el "Universal Serial Bus") es un canal serie; I2C (del que hablaremos más adelante) es un canal serie. Este capítulo y el siguiente *no* tratan sobre el concepto general de comunicación en serie: estos capítulos tratan sobre algo específico llamado "puerto serie" que tiene su propia implementación e historia.

El puerto de comunicaciones serie es *asíncrono* en el sentido de que ninguna de las líneas compartidas lleva una señal de reloj. En cambio, ambas partes deben acordar cómo de rápido se enviarán los datos a través del cable *antes* de que ocurra la comunicación. Un periférico llamado Universal Asynchronous Receiver/Transmitter (UART) envía bits a la velocidad especificada en su línea de salida y espera la llegada de los bits en su línea de entrada.

<p align="center">
<img class="white_bg" height="100" title="Serial Protocol" src="../assets/serial-proto.svg" alt="Serial Protocol"/>
</p>

El protocolo de comunicación serie funciona con tramas, cada una transporta un byte de datos. Cada trama tiene un *bit de inicio*, de 5 a 9 bits de datos de carga útil (enviados en formato lsb a msb; las aplicaciones actuales rara vez utilizan el tamaño de 9 bits; los bytes de 7 o menos bits en una trama se rellenarán a la izquierda hasta un byte de 8 bits con ceros) y 1 o 2 *bits de parada*. En el diagrama anterior, se envía un carácter ASCII 'E' utilizando 8 bits de datos y 1 bit de parada.

La velocidad de protocolo es conocida como *velocidad de transmisión (baud rate)* y se mide en bits por segundo (bps) (Si parece que esto suena mal, lo es. "Baud" se supone que es *símbolos* por segundo; un símbolo debería corresponder a una trama; incluso si un bit de datos se considera el "símbolo", no se envían a esta velocidad debido al resto del protocolo. Es una convención, y no tiene que tener sentido) Las velocidades en baudios históricamente comunes para la UART son 9.600bps, 19.200bps y 115.200bps, pero no es raro hoy en día enviar datos a 921.600bps.

Con una configuración "normal" de 1 bit de inicio, 8 bits de datos, 1 bit de parada y una velocidad de transmisión de 921,6 Kbps, podemos enviar y recibir 92.16 Kbytes por segundo, lo suficientemente rápido para transmitir audio CD sin comprimir en un solo canal. A la velocidad de 115.200 bps que usaremos, podemos enviar y recibir 11,52 Kbytes por segundo. Esto es suficiente para la mayoría de las aplicaciones.

Vamos a usar un puerto serie (indirectamente) para intercambiar datos entre la MB2 y el ordenador. Podríamos preguntarnos: ¿por qué no estamos usando RTT para comunicarnos como hicimos antes? RTT es un protocolo que está destinado a ser utilizado únicamente para depuración. No encontraremos dispositivos que usen RTT para intercambiar datos con otros dispositivos. Sin embargo, la transmisión serie se utiliza con bastante frecuencia. Por ejemplo, algunos receptores GPS envían la información de posición que reciben a través de líneas serie. Además, RTT, como muchos protocolos de depuración, es lento en comparación con las velocidades de transferencia serie.

<p align="center">
<img class="white_bg" height="500" title="Serial" src="../assets/serial.svg" alt="Serial"/>
</p>

Los ordenadores actuales no suelen tener un puerto serie, e incluso si lo tienen, el voltaje que utilizan (+5V en uno moderno, ±12V en un RS-232 antiguo) está fuera del rango que el hardware de la MB2 acepta y puede dañarlo. *No podemos conectar directamente el ordenador al microcontrolador*.

<a href="https://en.wikipedia.org/wiki/File:UART_to_USB_adapter.jpg">
<p align="center">
<img height="240" title="UART To USB Adapter" src="../assets/UART_to_USB_adapter.jpg" alt="UART to USB"/>
</p>
</a>

Una opción es comprar un conversor USB←→serie (por no más de 5€) que utiliza tensiones de +3.3V para la mayoría de las placas de microcontroladores actuales. El dispositivo que vemosa arriba es muy normal. Nosotros nos comunicaremos con el puerto serie de la MB2 a través del puerto USB incorporado en la placa microcontroladora. Sin embargo, si queremos conectarnos directamente a un puerto serie hardware, en el microcontrolador o en alguna otra placa, la única vía posible es el conversor serie.

Es posible configurar un canal del puerto USB de la MB2 para comunicarse con el convertidor USB←→serie incorporado del Microcontrolador (Esta es la parte derecha en la figura de anterior) Esta conversión USB←→serie se implementa utilizando el "[microcontrolador de comunicaciones]" de la placa: se genera una interfaz serie al microcontrolador y una interfaz serie USB virtual al ordenador. 
El ordenador reconoce una interfaz serie virtual a través de la clase de dispositivo USB CDC-ACM ("Communications Device Class - Abstract Control Model", ugh). 
El microcontrolador de la MB2 accede al ordenador como un dispositivo conectado a su puerto serie; el ordenador verá el puerto serie del microcontrador como un dispositivo serie virtual.

Ahora vamos a familiarizarnos con la interfaz de puerto serie USB que ofrece cada sistema operativo. Elige un SSOO:

- [Linux/UNIX](nix-tooling.md)
- [Windows](windows-tooling.md)
Para MacOS, no tenemos instrucciones específicas, pero se puede probar con las instrucciones de Linux. Sin embargo, ten en cuenta que tu experiencia puede diferir un poco.

[puerto serie]: https://en.wikipedia.org/wiki/Serial_port
[microcontrolador de comunicaciones]: ../05-meet-your-software/flash-it.md
