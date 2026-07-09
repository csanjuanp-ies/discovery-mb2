# Cómo usar GDB

A continuación se muestran algunos comandos útiles de GDB que pueden ayudarnos a depurar nuestros programas. Se da por hecho que ya has [programado un programa](../../05-meet-your-software/flash-it.md) en tu microcontrolador y
has conectado GDB a una sesión de `cargo-embed`.

## Depuración general

> **NOTA:** Muchos de los comandos que aparecen a continuación se pueden ejecutar utilizando una forma abreviada. Por ejemplo,
> `continue` se puede escribir simplemente como `c`, o `break $location` se puede escribir como `b $location`. Una vez que
> tengamos experiencia con los comandos que aparecen a continuación, ¡intenta ver hasta qué punto puedes acortarlos
> antes de que GDB deje de reconocerlos!

### Uso de Breakpoints

* `break $location`: Establece un punto de interrupción en un lugar del código. El valor de `$location` puede ser:
  * `break *main`: Interrumpir en la dirección exacta de la función `main`
  * `break *0x080012f2`: Interrumpir en la ubicación exacta de memoria `0x080012f2`
  * `break 123`: Detener la ejecución en la línea 123 del archivo que se está visualizando actualmente
  * `break main.rs:123`: Detener la ejecución en la línea 123 del archivo `main.rs`
* `info break`: Mostrar los puntos de interrupción actuales
* `delete`: Eliminar todos los puntos de interrupción
  * `delete $n`: Eliminar el punto de interrupción `$n` (siendo `n` un número. Por ejemplo: `delete $2`)
* `clear`: Eliminar el punto de interrupción de la siguiente instrucción
  * `clear main.rs:$function`: Eliminar el punto de interrupción en la entrada de `$function` en `main.rs`
  * `clear main.rs:123`: Eliminar el punto de interrupción de la línea 123 de `main.rs`
* `enable`: Activar todos los puntos de interrupción establecidos
  * `enable $n`: Activar el punto de interrupción `$n`
* `disable`: Desactivar todos los puntos de interrupción establecidos
  * `disable $n`: Desactivar el punto de interrupción `$n`

### Cobntrol de la Ejecución 

* `continue`: Iniciar o continuar la ejecución del programa
* `next`: Ejecutat la siguiente línea del programa
  * `next $n`: Repetir `next` `$n` veces
* `nexti`: Igual que `next`, pero utilizando instrucciones de máquina
* `step`: Ejecutar la siguiente línea; si esta incluye una llamada a otra función, entra en ese código
  * `step $n`: Repetir `step` `$n` veces
* `stepi`: Igual que `step`, pero utilizando instrucciones de máquina
* `jump $location`: Reanudar la ejecución en la ubicación especificada:
  * `jump 123`: Reanudar la ejecución en la línea 123
  * `jump 0x080012f2`: Reanudar la ejecución en la dirección 0x080012f2

### Impresuion de información

* `print /$f $data`: Mostar el valor contenido en la variable `$data`. Opcionalmente, permite formatear la
  salida con `$f`, que puede incluir:
    ```txt
    x: hexadecimal
    d: decimal con signo
    u: decimal sin signo
    o: octal
    t: binario
    a: dirección
    c: carácter
    f: punto flotante
    ```
  * `print /t 0xA`: Imprimir el valor hexadecimal `0xA` en formato binario (0b1010)

* `x /$n$u$f $address`: Examinar la memoria en la dirección `$address`. Opcionalmente, `$n` define el número de unidades a
  mostrar, `$u` el tamaño de la unidad (bytes, semipalabras, palabras, etc.), y `$f` cualquier formato de `print` definido anteriormente
  * `x /5i 0x080012c4`: Mostrar 5 instrucciones de máquina a partir de la dirección `0x080012c4`
  * `x/4xb $pc`: Imprimir 4 bytes de memoria a partir de donde apunta actualmente `$pc`
* `disassemble $location`
  * `disassemble /r main`: Desensamblar la función `main`, utilizando `/r` para mostrar los bytes que componen
    cada instrucción

### Observando la Tabla de Símbolos

* * `info functions $regex`: Mostrar los nombres y los tipos de datos de las funciones que coinciden con `$regex`; omite
    `$regex` para mostrar todas las funciones
  * `info functions main`: Mostrar los nombres y los tipos de las funciones definidas que contienen la palabra `main`
* `info address $symbol`: Mostrar dónde está almacenado `$symbol` en la memoria
  * `info address GPIOC`: Mostrar la dirección de memoria de la variable `GPIOC`
* `info variables $regex`: Mostrar los nombres y tipos de las variables globales que coinciden con `$regex`; omite
  `$regex` para mostrar todas las variables globales
* `ptype $data`: Mostrar información más detallada sobre `$data`
  * `ptype cp`: Mostrar información detallada sobre el tipo de la variable `cp`

### Cambianbdo datos en la Pila de programa

* `backtrace $n`: Mostrar el seguimiento de `$n` marcos; si se omite `$n`, se muestran todos los marcos
  * `backtrace 2`: Mostrar el seguimiento de los dos primeros marcos
* `frame $n`: Seleccionar el marco con el número o la dirección `$n`; si se omite `$n`, se muestra el marco actual
* `up $n`: Seleccionar el marco `$n` marco más arriba
* `down $n`: Seleccionar el marco `$n` marcos más abajo
* `info frame $address`: Describir el marco en `$address`; omite `$address` para el marco seleccionado actualmente
* `info args`: Mostrar los argumentos del marco seleccionado
* `info registers $r`: Mostrar el valor del registro `$r` en el marco seleccionado; omite `$r` para todos los
  registros
  * `info registers $sp`: Mostrar el valor del registro del puntero de pila `$sp` en el marco actual


### Cobntrolando `cargo-embed` de forma remota

* `monitor reset`: Reiniciar la CPU y volver a iniciar la ejecución desde el principio
