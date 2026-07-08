## NVIC y Prioridad de Interrupción

Hemos visto que las interrupciones hacen que el procesador salte inmediatamente a otra función en el código, pero ¿qué está pasando por debajo para permitir que esto suceda? En esta sección cubriremos algunos detalles técnicos que no serán necesarios para el resto del libro, así que, si no estás interesado, se puede saltar esta sección.

### El controlador de interrupciones

Las interrupciones permiten que el procesador responda a eventos producidos en los periféricos, como un pin de entrada GPIO cambiando de estado, un temporizador completando su ciclo o una UART recibiendo un nuevo byte. El periférico contiene circuitos que detectan el evento e informan a un controlador dedicado al manejo de interrupciones. En los procesadores Arm, el encargado de esta gestión de interrupciones se llama NVIC: Controlador de Interrupciones Anidadas y Vectorizadas (Nested Vector Interrupt Controller).


> **NOTA** En otras arquitecturas de microcontroladores como RISC-V, los nombres y detalles explicados aquí serán diferentes, pero los principios subyacentes son muy similares.

El NVIC puede recibir peticiones para elevar una interrupción desde muchos periféricos. Incluso es común que un periférico tenga múltiples posibles interrupciones por ejemplo, un puerto GPIO posee una interrupción para cada pin, o una UART usando tanto una interrupción de "datos recibidos" como una de "transmisión de datos finalizada". La tarea del NVIC es priorizar estas interrupciones, recordar cuáles aún necesitan ser procesadas y luego hacer que el procesador ejecute el código del manejador de interrupciones asociado.

### Prioridad en las interrupciones

El controlador de interrupciones NVIC tiene un "nivel de prioridad" configurable para cada interrupción. Dependiendo de su configuración, el NVIC puede asegurarse de que la interrupción actual se procese completamente antes de ejecutar una nueva, o puede "interrumpir" (preemptive) el procesador en medio de una interrupción para manejar otra que tenga mayor prioridad.

Esta apropiación del procesador permite que los procesadores respondan muy rápidamente a eventos críticos. Por ejemplo, un controlador de robot podría usar interrupciones de baja prioridad para gestionar el envío de información de estado al operador, pero también utilizar una interrupción de alta prioridad cuando un sensor detecta una colisión inminente para que pueda detener inmediatamente el movimiento de los motores. ¡No querríamos que el robot espere hasta haber terminado de enviar datos para detenerse!

Si aparece una interrupción de menor prioridad o similar mientras se está ejecutando una ISR, el NVIC estableced como "pendiente" la nueva interrupción y ejecutará su ISR en algún momento después de que la actual haya terminado. Cuando una función ISR regresa, el NVIC verifica si, mientras la ejecución, ocurrieron otras interrupciones que necesitan ser manejadas. Si es así, el NVIC revisa la tabla de interrupciones y llama a la ISR de mayor prioridad allí vectorizada. De lo contrario, la CPU regresa al programa en ejecución.

Hay que tener en cuenta que, si se desactivan por completo las interrupciones, todas las que se generen quedarán en espera. Las interrupciones se gestionarán una vez que se vuelvan a activar las interrupciones.

En Rust, podemos programar el NVIC usando el crate [`cortex-m`], que proporciona métodos para habilitar y deshabilitar (llamados `unmask` y `mask`) interrupciones, establecer prioridades de interrupción y activar interrupciones desde software. Frameworks como [RTIC] pueden manejar la configuración del NVIC, aprovechando la flexibilidad del NVIC para proporcionar manejo de recursos y tareas adecuado.


Podemos obtener más información sobre el NVIC en la [documentación de Arm].

[`cortex-m`]: https://docs.rs/cortex-m/latest/cortex_m/peripheral/struct.NVIC.html
[RTIC]: https://rtic.rs/
[documentación de Arm]: https://developer.arm.com/documentation/ddi0337/e/Nested-Vectored-Interrupt-Controller/About-the-NVIC

### La tabla de vectores de interrupción

Cuando describimos el NVIC, se dijo que podía "hacer que el procesador ejecute el código del manejador de interrupciones relevante". Pero, ¿cómo funciona eso realmente?

Primero, necesitamos algún mecanismo para que el procesador conozca qué código ejecutar para cada interrupción. En los micros Cortex-M, esto implica una parte de la memoria llamada tabla de vectores. Normalmente, se encuentra al principio de la memoria flash que contiene el código, la cual se reprograma cada vez que subimos un nuevo programa, y contiene una lista de direcciones: las ubicaciones en memoria de cada función de interrupción. La disposición específica del inicio de la memoria está definida por Arm en el [Manual de Referencia de Arquitectura]; para nuestros propósitos, la parte importante es que los bytes 64 hasta 256 contienen las direcciones de todos los 48 manejadores de interrupciones para el procesador nRF que usamos, cuatro bytes por dirección. Cada interrupción tiene un número, del 0 al 47. Por ejemplo, `TIMER0` es la interrupción número 8, y por lo tanto los bytes 96 a 100 contienen la dirección de cuatro bytes de su manejador de interrupciones. Cuando el NVIC le dice al procesador que maneje la interrupción número 8, la CPU lee la dirección almacenada en esos bytes y salta a ella.

¿Cómo se genera esta tabla de vectores en el programa? Usamos el crate [`cortex-m-rt`] que se encarga de esto por nosotros. Proporciona una interrupción predeterminada para cada posición no utilizada (ya que cada posición debe estar llena) y permite que el código anule este valor predeterminado siempre que queramos especificar nuestro propio manejador de interrupciones. Hacemos esto usando la macro `#[interrupt]`, que requiere que la función tenga un nombre específico relacionado con la interrupción que maneja. Luego, el crate `cortex-m-rt` utiliza su script de enlace para organizar que la dirección de esa función se coloque en la parte correcta de la memoria.

Para obtener más detalles sobre cómo se gestionan estos manejadores de interrupciones en Rust, consulte los capítulos de Excepciones e Interrupciones en el [Libro de Rust Embebido].

[Manual de Referencia de Arquitectura]: https://developer.arm.com/documentation/ddi0403/latest
[`cortex-m-rt`]: https://docs.rs/cortex-m-rt
[Libro de Rust Embebido]: https://docs.rust-embedded.org/book/
