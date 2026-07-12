# RTRM: Manual de Referencia

Ya hemos visto los pins de la GPIO en el nRF52833. En este chip (y en muchos otros) los pines GPIO están agrupados en *puertos*. Hay dos puertos, el Puerto 0 y el Puerto 1, abreviados como `P0` y `P1` respectivamente. Los pines dentro de cada puerto se nombran con números que comienzan desde 0. El Puerto 0 tiene 32 pines, nombrados `P0.00` a `P0.31`, y el Puerto 1 tiene 10 pines, nombrados `P1.00` a `P1.09`.

Lo primero que tenemos que recordar es qué pin está conectado a cada LED. Anteriormente, hicimos esto siguiendo el esquema. Resulta que eso es difícil: la información requerida está en la tabla de asignación de pines del MB2 [pinmap table].

[pinmap table]: https://tech.microbit.org/hardware/schematic/#v2-pinmap

La tabla dice que:
- `ROW1`, la fila superior de LEDs, está conectada al pin `P0.21`. `P0.21` es la forma corta de: Pin 21 en el Puerto 0.
- `ROW5`, la fila inferior de LEDs, está conectada al pin `P0.19`.

Hasta este momento, sabemos que queremos cambiar el estado de los pines `P0.21` y `P0.19` para encender y apagar las filas superior e inferior respectivamente. Esos pines son parte del Puerto 0, así que usaremos el periférico `P0` para configurarlos.

Cada periférico tiene un bloque de registros asociado. Un bloque de registros es una colección de registros asignados en memoria contigua. La dirección en la que comienza el bloque de registros se conoce como su dirección base. Necesitamos averiguar cuál es la dirección base del periférico `P0`. Esa información está en la siguiente sección de la [Especificación del Producto] del microcontrolador o en la [Especificación del Producto (html)]:

> Section 4.2.4 Instantiation - Page 22

La tabla indica que la dirección base del bloque de registros `P0` es `0x5000_0000`. 

Cada periférico tiene su propia sección en la documentación. Cada una de ellas termina con una tabla de los registros que contiene. Para la familia de periféricos `GPIO`, esa tabla está en:

> Section 6.8.2 Registers - Page 144

`OUT` es el registro que debemos usar para establecer o borrar los pines. Su valor de desplazamiento es `0x504` desde la dirección base de `P0`. Podemos buscar `OUT` en la [Especificación del Producto].

La información de ese registro está en la tabla de registros `GPIO`:

> Subsection 6.8.2.1 OUT - Page 145

De todos modos, `0x5000_0000` + `0x504` = `0x50000504`, es la dirección en la que estamos escribiendo en el código.

Esa es la dirección del registro que vamos a utilizar. La documentación dice cosas interesantes: Primero, este registro se puede escribir y leer. Segundo, el registro ocupa 32 bits de memoria y cada bit representa el estado del pin correspondiente. 
Eso significa que el bit 19 coincide con el pin 19, por ejemplo. Establecer el bit en 1 habilitará la salida del pin, y establecerlo en 0 lo restablecerá. Además, podemos ver que todas las salidas de los pines están deshabilitadas por defecto, ya que el valor de reinicio de todos los bits es 0.

Vamos a usar el comando `examine` de GDB (`x`). Dependiendo de la configuración del servidor, GDB podría negarse a leer de memoria que no esté especificada. Deshabilitamos este comportamiento ejecutando:

```
set mem inaccessible-by-default off
```

Vamos allá, Primero desactivamos  `inaccessible-by-default`, luego establecemos un par de puntos de interrupción, reiniciamos el dispositivo y lo detenemos.

>**Nota del tr.**: He tenido que añadir en los puntos de interrupción, el nombre del fichero para que bajo windows los reconociera bien:  break **main.rs:**.

```
(gdb) set mem inaccessible-by-default off
(gdb) break main.rs:16
Breakpoint 1 at 0x172: file src/07-registers/src/main.rs, line 16.
Note: automatically using hardware breakpoints for read-only addresses.
(gdb) break main.rs:19
Breakpoint 2 at 0x17c: file src/07-registers/src/main.rs, line 19.
(gdb) break main.rs:22
Breakpoint 3 at 0x184: file src/07-registers/src/main.rs, line 22.
(gdb) break main.rs:25
Breakpoint 4 at 0x18c: file src/07-registers/src/main.rs, line 25.
(gdb) monitor reset halt
Resetting and halting target
Target halted
```

Perfecto, ejecutemos hasta el primer punto de ruptura, justo antes de la línea 16, y veamos el contenido del registro en la dirección `0x50000504`.

```
(gdb) c
Continuing.

Breakpoint 1, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:16
16              *(PORT_P0_OUT as *mut u32) |= 1 << 21;
(gdb) x 0x50000504
0x50000504:     0x00000000
```

Ok, vemos que el valor del registro es `0x00000000` o `0` en este punto. Esto corresponde con los datos de la [Especificación del Producto], que dice que `0` es el 'valor de reinicio' de este registro. Eso significa que vez que el MCU se reinicia, el registro tendrá `0` como su valor.

Sigamos. Esta línea consta de varias instrucciones (lectura, OR bit a bit y escritura), así que necesitamos indicarle al depurador que continúe la ejecución más de una vez, hasta que lleguemos al siguiente punto de interrupción.


```
(gdb) c
Continuing.

Program received signal SIGINT, Interrupt.
0x00000174 in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:16
16              *(PORT_P0_OUT as *mut u32) |= 1 << 21;
(gdb) c
Continuing.

Breakpoint 2, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:19
19              *(PORT_P0_OUT as *mut u32) |= 1 << 19;
```

Ahora hemos parado justo antes de la línea 19, lo que significa que la línea 16 se ha ejecutado completamente en este punto. Echemos un vistazo al contenido del registro `OUT` nuevamente:


```
(gdb) x 0x50000504
0x50000504:     0x00200000
```

El valor ha cambiado a `0x00200000` lo que es `2097152` en decimal, o `2^21`. Eso significa que el bit 21 está establecido en 1, y el resto de los bits están fijados en 0. Eso corresponde con el código de la línea 16, que escribe (`1 << 21`) un 1 desplazado a la izquierda 21 posiciones, bit a bit OR con el valor actual de `OUT` (que era 0), y lo graba al registro `OUT`.


Escribir el valor  `1 << 21` (`OUT[21]= 1`) en `OUT` fija `P0.21` *alto*. Eso enciende la fila superior de LED. Comprueba que está encendida.

Sí, eso iba a decir. Ahora, pulsa "c" otra vez para continuar la ejecución hasta el siguiente
punto de ruptura e imprimamos su valor.

```
(gdb) c
Continuing.
```

Paramos antes de la línea 19.

```
Program received signal SIGINT, Interrupt.
0x0000017e in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:19
19              *(PORT_P0_OUT as *mut u32) |= 1 << 19;
(gdb) c
Continuing.

Breakpoint 3, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:22
22              *(PORT_P0_OUT as *mut u32) &= !(1 << 21);
(gdb) x 0x50000504
0x50000504:     0x00280000
```

En la línea 19, hemos puesto el bit 19 de `OUT` a 1, manteniendo el bit 21 como está. El resultado es `0x00280000`, que es `2621440` en decimal, o `2^19 + 2^21`, lo que significa que tanto el bit 19 como el bit 21 tiene valor 1.

Fijar el valor `1 << 19` (`OUT[19]= 1`) en `OUT` establece `P0.19` *alto*. Eso enciende la fila inferior de LED. Comprueba que está encendida.

Las líneas siguientes apagan las filas de nuevo. Primero la fila superior, luego la fila inferior. Esta vez, haciendo una operación bit a bit AND, combinada con un bit a bit NOT. Calculamos `!(1 << 21)`, que es todos los bits escritos a 1, excepto el bit 21. Luego, hacemos un AND bit a bit de eso con el valor actual de `OUT`, asegurándonos de que solo el bit 21 se establezca en 0, manteniendo el valor de los otros bits intacto.

Continuemos la ejecución y comprobemos que los valores existentes en el registro `OUT` coinciden con lo que esperamos. Para apusar la ejecución se puede pulsar `CTRL+C` una vez que el dispositivo entre en el bucle infinito al final de la función `main`.


```
(gdb) c
Continuing.

Program received signal SIGINT, Interrupt.
0x00000186 in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:22
22              *(PORT_P0_OUT as *mut u32) &= !(1 << 21);
(gdb) c
Continuing.

Breakpoint 4, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:25
25              *(PORT_P0_OUT as *mut u32) &= !(1 << 19);
(gdb) x 0x50000504
0x50000504:     0x00080000
(gdb) c
Continuing.

Program received signal SIGINT, Interrupt.
0x0000018e in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:25
25              *(PORT_P0_OUT as *mut u32) &= !(1 << 19);
(gdb) c
Continuing.
^C
Program received signal SIGINT, Interrupt.
0x00000196 in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:28
28          loop {}
(gdb) x 0x50000504
0x50000504:     0x00000000
```
En este punto todos los LED deberían estar apagados.

[Especificación del Producto]: https://docs-be.nordicsemi.com/bundle/ps_nrf52833/attach/nRF52833_PS_v1.7.pdf

[Especificación del Producto (html)]:https://docs.nordicsemi.com/r/bundle/ps_nrf52833/page/keyfeatures_html5.html
