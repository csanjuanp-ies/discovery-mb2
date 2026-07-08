## Compartiendo datos mediante Globales

> **NOTA* Este contenido se ha obtenido parcialmente (con permiso) de la publicación del blog
> *[Interrupts Is Threads]* de James Munns, que contiene más discusión sobre este tema.

Como hemos mencionado anteriormente, cuando ocurre una interrupción no se nos pasan argumentos y no podemos devolver ningún resultado. Esto hace que sea difícil para nuestro programa interactuar con periféricos y otro estado del programa principal. Antes de preocuparnos por este problema de programación a bajo nivel, probablemente valga la pena pensar en los hilos en el "std" Rust.

### Rust "std": Compartiendo datos con una hebra

En Rust "std", podemos crear hilos de ejecución (threads) que se corren en paralelo con la hebra principal. Esto nos permite realizar múltiples tareas al mismo tiempo, pero también introduce la necesidad de compartir datos entre hilos de manera segura.


En Rust "std", también tenemos que tener en cuenta el intercambio de datos cuando hacemos cosas como
crear un hilo.

Cuando queremos *compartir* algo con la hebra, debemos pasar la propiedad a través de un cierre (closure).

```rust
// Create a string in our current thread
let data = String::from("hello");

// Now spawn a new thread, and GIVE it ownership of the string
// that we just created
std::thread::spawn(move || {
    std::thread::sleep(std::time::Duration::from_millis(1000));
    println!("{data}");
});
```

Si queremos *compartir* algo, y aún tener acceso a ello en el hilo original, normalmente no podemos pasar una referencia. Si hacemos esto:

```rust
use std::{thread::{sleep, spawn}, time::Duration};

fn main() {
    // Create a string in our current thread
    let data = String::from("hello");
    
    // make a reference to pass along
    let data_ref = &data;
    
    // Now spawn a new thread, and GIVE it ownership of the string
    // that we just created
    spawn(|| {
        sleep(Duration::from_millis(1000));
        println!("{data_ref}");
    });
    
    println!("{data_ref}");
}
```

Obtendremos un error de compilación:

```text
error[E0597]: `data` does not live long enough
  --> src/main.rs:6:20
   |
3  |       let data = String::from("hello");
   |           ---- binding `data` declared here
...
6  |       let data_ref = &data;
   |                      ^^^^^ borrowed value does not live long enough
...
10 | /     spawn(|| {
11 | |         sleep(Duration::from_millis(1000));
12 | |         println!("{data_ref}");
13 | |     });
   | |______- argument requires that `data` is borrowed for `'static`
...
16 |   }
   |   - `data` dropped here while still borrowed
```

Tenemos que *asegurarnos de que los datos vivan lo suficiente* para que tanto el hilo actual como el nuevo hilo que estamos creando puedan acceder a ellos. Podemos hacer esto poniendo los datos en un `Arc` (Atomically Reference Counted heap allocation) de la siguiente manera:

```rust
use std::{sync::Arc, thread::{sleep, spawn}, time::Duration};

fn main() {
    // Create a string in our current thread
    let data = Arc::new(String::from("hello"));
    
    let handle = spawn({
        // Make a copy of the handle to GIVE to the new thread.
        // Both `data` and `new_thread_data` are pointing at the
        // same string!
        let new_thread_data = data.clone();
        move || {
            sleep(Duration::from_millis(1000));
            println!("{new_thread_data}");
        }
    });
    
    println!("{data}");
    // wait for the thread to stop
    let _ = handle.join();
}
```

Esto ya está bien. Podemos acceder a los datos en ambos hilos mientras queramos. Pero, ¿y si queremos *modificar* los datos en ambos lugares?

Para este caso, normalmente necesitaremos algún tipo de "mutabilidad interna": un tipo que no requiera un `&mut` para cambiarlo. En la programación estándar, normalmente recurriríamos a un tipo como `Mutex`, bloqueándolo (`lock()`) para obtener acceso mutable a los datos.

Se parecería algo como esto:

```rust
use std::{sync::{Arc, Mutex}, thread::{sleep, spawn}, time::Duration};

fn main() {
    // Create a string in our current thread
    let data = Arc::new(Mutex::new(String::from("hello")));
    
    // lock it from the original thread
    {
        let guard = data.lock().unwrap();
        println!("{guard}");
        // the guard is dropped here at the end of the scope!
    }
    
    let handle = spawn({
        // Make a copy of the handle, that you GIVE to the new thread.
        // Both `data` and `new_thread_data` are pointing at the
        // same `Mutex<String>`!
        let new_thread_data = data.clone();
        move || {
            sleep(Duration::from_millis(1000));
            {
                let mut guard = new_thread_data.lock().unwrap();
                // we can modify the data!
                guard.push_str(" | thread was here! |");                
                // the guard is dropped here at the end of the scope!
            }
        }
    });
    
    // wait for the thread to stop
    let _ = handle.join();
    {
        let guard = data.lock().unwrap();
        println!("{guard}");
        // the guard is dropped here at the end of the scope!
    }
}
```

Si ejecutamos el código, veremos:


```text
hello
hello | thread was here! |
```

¿Por qué Rust "std" nos hace hacer esto? Rust nos está ayudando a pensar en dos cosas:
1. Los datos viven lo suficiente (¡potencialmente "para siempre"!)
2. Solo un fragmento de código puede acceder de manera mutable a los datos a la vez.

Si Rust nos permitiera acceder a datos que no pudieran vivir lo suficiente, como datos prestados de un hilo a otro, las cosas podrían salir mal. Sería factible obtener datos corruptos si el hilo original termina o encuentra un error y luego el segundo hilo intenta acceder a los datos que ahora son inválidos. Si Rust permitiera que dos fragmentos de código cambiaran los mismos datos al mismo tiempo, tendríamos una condición de carrera de datos, y los datos podrían terminar corruptos.

### Rust Embebido: Compartiendo datos con un ISR

En Rust embebido nos preocupamos por las mismas cosas cuando se trata de compartir datos con los manejadores de interrupciones. Al igual que con los hilos, las interrupciones pueden ocurrir en cualquier momento, como si un hilo se despertara y accediera a algunos datos compartidos. Esto significa que los datos que compartimos con una interrupción deben vivir lo suficiente, y hay que asegurarnos que nuestro código principal no esté usando algunos datos compartidos con un ISR cuando ese ISR se ejecute y *también* intente trabajar con ellos.

De hecho, en Rust embebido, modelamos las interrupciones de manera similar a como modelamos los hilos en Rust: se aplican las mismas reglas, por las mismas razones. Sin embargo, en Rust embebido, tenemos algunas diferencias cruciales:

* Las interrupciones no trabajan exactamente como hebras: las configuramos de antemano, y esperan hasta que ocurra algún evento (como presionar un botón o que expire un temporizador). En ese momento se ejecutan, pero sin acceso a ningún contexto pasado.

* Las interrupciones pueden ser llamadas múltiples veces, una por cada vez que ocurre el evento.

Como no podemos pasar contexto a las interrupciones como argumentos de función, necesitamos encontrar otro lugar para almacenar esos datos. En Rust embebido no tenemos acceso a asignaciones en el heap: por lo tanto, no se puede usar `Arc` y similares.

Sin esa posibilidad de pasar cosas por valor, y sin un heap para almacenar datos, solo nos deja con un lugar para poner nuestros datos compartidos a los que nuestro ISR puede acceder: `static` globales.

### Rust Embebido compartiendo datos con un ISR: El "método estándar" 

Las variables globales son ciudadanos de segunda clase en Rust, con muchas limitaciones en comparación con las variables locales. Podemos declarar una variable global así:

```rust
static COUNTER: usize = 0;
```

Desde luego, esto no es muy útil: queremos poder variar el `COUNTER`. Podemos decir 


```rust
static mut COUNTER: usize = 0;
```
Pero ahora el acceso no será seguro.

```rust
unsafe { COUNTER += 1 };
```

Esta inseguridad es por una razón: imagina que en medio de actualizar `COUNTER` se ejecuta un manejador de interrupción que también intenta actualizar `COUNTER`. Se producirá errores de carrera. Claramente, es necesario algún tipo de bloqueo.

El crate `critical-section` proporciona un tipo de `Mutex`, pero con una API y operaciones inusuales. Al examinar el `Cargo.toml` de este capítulo, se comprueba que la característica `critical-section-single-core` en el crate `cortex-m` está habilitada. Esta característica determina que solo hay un núcleo de procesador en este sistema, y que, por lo tanto, la sincronización se puede realizar simplemente *deshabilitando las interrupciones* durante la sección crítica. Si no estamos en una interrupción, esto asegurará que solo el programa principal tenga acceso a la varaible global. Si estamos en una interrupción, esto asegurará que el programa principal no pueda acceder a ella (el control del programa está en el manejador de interrupciones) y que ningún otro manejador de interrupciones de mayor prioridad pueda activarse.

`critical_section::Mutex` es un poco extraño en el hecho que proporciona exclusión mutua, pero no proporciona mutabilidad por sí mismo. Para hacer que los datos sean mutables, necesitaremos proteger un tipo de mutabilidad interior, generalmente `RefCell`, con el mutex. Este `Mutex` también es un poco extraño en que no se bloquea con `.lock()`. En su lugar, inicia una sección crítica con una closure que recibe un "token de sección crítica" que previene la ejecución de otros programas. Este token puede pasarse al método `borrow()` del `Mutex` para permitir el acceso.

Si ahora lo juntamos todo, obtenemos la capacidad de compartir estado entre los manejadores de interrupciones y el programa principal
(`examples/count-once.rs`).

```rust
{{#include examples/count-once.rs}}
```

Todavía no podemos regresar de manera segura desde nuestra ISR, pero ahora tenemos todas las herramientas para hacer algo al respecto: compartir el `GPIOTE` con la ISR para que la ISR pueda borrar la interrupción.

### Compartir periféricos (etc.) con variables globales

Queda un problema más por resolver: las variables globales de Rust deben inicializarse de manera estática, antes de que comience el programa. Para el contador eso fue fácil: simplemente se inicializó en 0. Si queremos compartir el periférico `GPIOTE`, sin embargo, eso no funcionará. El periférico debe recuperarse de la estructura `Board` y configurarse una vez que el programa haya comenzado: no hay un inicializador `const` para esto (ni puede haberlo).

Vamos a rescribir el contador de botones un poco. Primero, movemos la cuenta real para que sea un `AtomicUsize`. Este es un tipo más natural para esta variable global de todos modos. A continuación, agregamos otra variable global `GPIOTE_PERIPHERAL` usando el tipo `LockMut` del crate `critical-section-lock-mut`. Este crate es un envoltorio para el patrón de la última sección.

Ahora, en el programa principal, podemos inicializar el `GPIOTE` y luego ponerlo a disposición de la ISR, ya podemos eliminar la sentencia de pánico y dejar que el contador aumente en cada pulsación del botón. Movemos la visualización del contador al bucle principal, para mostrar que el contador se comparte entre el manejador de interrupciones y el resto del programa.


Ejecuta este ejemplo (`examples/count.rs`) y fíjate en que el recuento aumenta en 1 cada vez que
se pulsa el botón A del MB2.


```rust
{{#include examples/count.rs}}
```

> **NOTA** Siempre es una buena idea que al compilar ejemplos que involucren el manejo de interrupciones se use `--release`. Los manejadores de interrupciones largos pueden llevar a mucha confusión.

Pero, en realidad, ese `rprintln!()` en el controlador de interrupciones es una mala práctica: mientras el controlador de interrupciones está ejecutando el código de impresión, no se puede ejecutar nada más. Cambiemos la visualización al bucle principal, justo después del `wfi()` "esperar a la interrupción". La cuenta se mostrará cada vez que termine un controlador de interrupciones (`examples/count-bounce.rs`).


```rust
{{#include examples/count-bounce.rs}}
```
En este ejemplo, la cuenta aumenta en 1 cada vez que se pulsa el botón A del MB2. Tal vez, especialmente si la MB2 es vieja (!), es posible que una sola pulsación aumente el contador en varios valores. *Esto no es un error de software*, generalmente. En la siguiente sección, hablaremos sobre lo que podría estar sucediendo y cómo deberíamos manejarlo.

[Interrupts Is Threads]: https://onevariable.com/blog/interrupts-is-threads