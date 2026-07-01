# El protocolo
El protocolo I2C es más elaborado que el protocolo de comunicación en serie porque admite comunicación estructurada entre varios dispositivos. Veamos cómo funciona:

## Controlador → Dispositivo
Si el controlador quiere enviar datos al destino:

1. Controlador: Emite un Broadcast START
2. C: Emite la dirección del dispositivo destino (7 bits) + el bit de R/W (8th) establecido a WRITE
3. Dispositivo: Responde un ACK (ACKnowledgement)
4. C: Envía un byte
5. D: Responde con un ACK
6. Repite los pasos 4 y 5 cero o más veces
7. C: Emite un Broadcast STOP, o comienza otra transacción de escritura

> **NOTA** La dirección del destino puede tener 10 bits en vez de 7 bits. No cambiaría nada más.

## Controlador ← Dispositivo
Si el controlador quiere leer datos del destino:

1. Controlador: Emite un Broadcast START
2. C: Emite la dirección del dispositivo destino (7 bits) + el bit de R/W (8th) establecido a READ
3. Dispositivo: Responde un ACK (ACKnowledgement)
4. T: Envía un byte
5. C: Responde con un ACK
6. Repite los pasos 4 y 5 cero o más veces
7. C: Emite un Broadcast STOP, o comienza otra transacción de lectura

> **NOTA** La dirección del destino puede tener 10 bits en vez de 7 bits. No cambiaría nada más.

## "Registros de dispositivo"
Muchos dispositivos I2C están organizados internamente como si tuvieran "registros de dispositivo", cada uno con una dirección de 8 bits y un contenido de 8 bits. Normalmente, los registros se escriben con una escritura de dos bytes: el primer byte es la dirección del registro y el segundo el nuevo valor del registro.

Una operación denominada "combinada" o "dividida" puede consistir en una escritura en el destino seguida de una lectura inmediata del mismo, tal y como se muestra en el diagrama anterior. Normalmente, los registros del dispositivo se leen de esta manera: se escribe la dirección del registro y, a continuación, se lee inmediatamente el valor actual de dicho registro.

Algunos elementos I2C pueden leer y escribir múltiples registros con direcciones adyacentes mediante alguna forma de "auto-incremento de dirección", lo que permite enviar solo la primera dirección del registro del dispositivo y luego confiar en que el dispositivo incremente la dirección para lecturas o escrituras posteriores.

El protocolo I2C es un protocolo complejo, y existen muchas variaciones y características especiales. Lea el manual de su dispositivo cuidadosamente para ver qué se necesita hacer para comunicarse con él.
