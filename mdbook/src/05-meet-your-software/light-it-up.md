# ¡Enciéndelo!
Vamos a terminar el capítulo haciendo que uno de los muchos LEDs del MB2 se encienda. Para conseguir esta tarea utilizaremos uno de los traits proporcionados por `embedded-hal`, específicamente el trait [`OutputPin`] que nos permite encender o apagar un pin.


[`OutputPin`]: https://docs.rs/embedded-hal/0.2.6/embedded_hal/digital/v2/trait.OutputPin.html

## Los Led de la micro:bit 

En la parte de atrás de la micro:bit encontramos un cuadrado de 5x5 LED, normalmente llamado matriz de LED. Esta estructura de matriz se utiliza para que en lugar de tener que usar 25 pines separados para controlar cada uno, podamos usar solo 10 (5+5) pines para establecer qué columna y qué fila se enciende.

Pasemos a usar el crate `microbit-v2` para manipular los LED. En el [siguiente capítulo] entraremos en detalle sobre todas las opciones disponibles.

[siguiente capítulo]: ../06-hello-world/README.md

## Encenderlo de verdad

El código necesario para encender un LED en la matriz es realmente bastante simple, pero requiere un poco de configuración. Primero echemos un vistazo a `src/main.rs`.


```rust
{{#include src/main.rs}}
```

Las primeras líneas hasta la función `main` solo hacen algunas importaciones básicas y configuraciones que ya hemos visto antes. Sin embargo, la función `main` parece bastante diferente a lo que hemos visto hasta ahora.

La primera línea hace referencia a cómo la mayoría de los HAL escritos en Rust funcionan internamente. Como se ha discutido antes, están construidos sobre los PAC crates que poseen (en el sentido de Rust) todos los periféricos de un chip. Cuando decimos

    let mut board = Board::take().unwrap();


Tomamos todos estos periféricos del PAC y los asignamos a una variable. En este caso concreto,
no solo estamos trabajando con un HAL, sino con un BSP completo, por lo que también se hace cargo de la representación en Rust
de los demás chips de la placa.


> **NOTA**: Si nos parece extraño el hecho de llamar a `unwrap()`, ya que en teoría es posible usar
> `take()` más de una vez, es porque esto daría lugar a que los periféricos quedaran concectados a dos
> variables distintas y, por lo tanto, muchos comportamientos resultarían confusos, porque dos variables podrían modificar el
> mismo recurso. Para evitarlo, los PAC se han implementado de tal manera que se produciría un error si se 
> adquiriere la posesión de los periféricos dos veces.

(Una vez más, si todo esto es un poco confuso, en el [siguiente capítulo] se volverá a explicar con mayor detalle.)

Ahora podemos encender el LED conectado a `row1`, `col1` poniendo el pin `row1` en estado alto
(es decir, activándolo).  La razón por la que podemos dejar `col1` en estado bajo se debe al funcionamiento del circuito de la matriz de LED. 

Además, `embedded-hal` está diseñado de tal manera que cualquier operación en el hardware pueda devolver un error, incluso simplemente al activar o desactivar un pin. 
En este caso es muy improbable, por lo que podemos usar simplemente `unwrap()` el resultado.

## Probarlo

Probar nuestro pequeño programa es muy sencillo. Ejecutamos `cargo embed` y dejamos que se compile como antes. A continuación, ejecutamos GDB y nos conectamos a la sesión de GDB.


```
$ gdb ../../../target/thumbv7em-none-eabihf/debug/meet-your-software
(gdb) target remote :1337
Remote debugging using :1337
cortex_m_rt::Reset () at /home/nix/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.12/src/lib.rs:489
489     pub unsafe extern "C" fn Reset() -> ! {
(gdb)
```


Ahora dejamos que el programa se ejecute mediante el comando `continue` de GDB:
uno de los LED de la parte frontal del micro:bit debería
encenderse.

