# A la espera de que me interrumpan

Quizá te hayas preguntado por qué hemos estado utilizando `asm::wfi()` (espera a la instrucción) en nuestro bucle principal en lugar de algo como `asm::nop()`.

Como ya se ha comentado anteriormente, `asm::nop()` significa "no-op" (sin operación) y es una instrucción que en la CPU no ejecuta nada.  Sin duda, podríamos haber utilizado `asm::nop()` en nuestro bucle principal y el programa se habría comportado de la misma manera.  El microcontrolador, por su parte, se comportaría de forma diferente.

Al llamar a `asm::wfi()`, la CPU entra en modo "Wait For Interrupt" (WFI). Cuando la CPU está en modo WFI, permanece inactiva hasta que una interrupción la activa. Durante este estado de inactividad, la CPU deja de ejecutar instrucciones, desactiva algunos relojes y periféricos, y entra en un estado de bajo consumo, aunque mantiene el núcleo en funcionamiento.  Cuando se produce una interrupción, la CPU se activa y ejecuta las instrucciones con normalidad.

La principal diferencia entre `asm::wfi()` y `asm::nop()` es que la instrucción NOP se completa de inmediato y, por lo tanto, se ejecutará repetidamente en un bucle. La instrucción NOP sigue teniendo que recuperarse de la memoria del programa y ejecutarse, aunque su ejecución no realice ninguna acción. La mayoría de los microcontroladores que encontrarás en el mercado disponen de un modo de bajo consumo (algunos incluso tienen varios, cada uno con diferentes componentes activos y características de consumo de energía distintas) que puede (y, en muchos casos, debería) utilizarse para ahorrar energía. La instrucción WFI detiene la ejecución *en un modo de bajo consumo* hasta que se recibe una interrupción.

Encontrarás algunos programas controlados por interrupciones que no contienen más que `asm::wfi()` en el bucle principal, y en los que toda la lógica del programa se implementa en los controladores de interrupciones.
