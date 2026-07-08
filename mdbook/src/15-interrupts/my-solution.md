# My Solution

Me costó un poco averiguar cómo debía calcular el controlador de interrupciones
el momento de la siguiente interrupción para mantener
la sirena en marcha. Al final utilicé un par de variables de estado
para controlar si el pin del altavoz estaba activado o desactivado
(podría haberlo comprobado directamente en el hardware) y para llevar un registro de en qué
momento del ciclo de encendido y apagado se encontraba la sirena.

Aquí está una posible solución (`src/main.rs`).


```rust
{{#include src/main.rs}}
```