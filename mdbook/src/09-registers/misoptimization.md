# (des)Optimización

Leer y escribir en registros es algo bastante especial. Incluso me atrevería a decir que es la encarnación de los efectos secundarios. En el ejemplo anterior escribimos cuatro valores diferentes en el mismo registro. Si no hubieras sabido que esa dirección era un registro, podrías haber simplificado la lógica para escribir solo el valor final `0x00000000` en el registro.

Por defecto, LLVM, el optimizador del compilador, no sabe que estamos tratando con un registro y fusionará las escrituras, cambiando así el comportamiento de nuestro programa. Comprobémoslo.

Primero, utilizaremos cargo objdump para obtener el ensamblado de los artefactos de construcción tanto de la versión optimizada como de la no optimizada.


```
# Non-optimized
cargo objdump -- --disassemble --no-show-raw-insn --source > debug.dump
# Optimized
cargo objdump --release -- --disassemble --no-show-raw-insn --source > release.dump
```

Vamos a ver qué hay aquí. En concreto, hay que tratar de encontrar el ensamblado que manipula el registro `OUT`.

Primero, echaremos un vistazo al contenido de `debug.dump`, el ensamblado de la construcción no optimizada.
Hemos saltado un montón de instrucciones, pero las que nos interesan están en la función `registers::__cortex_m_rt_main::h0b7888ca966441cf`, que es la función `main` de nuestro programa.

Además de saltar las instrucciones, se han añadido comentarios detrás de `; <--`, indicando el número de línea del fichero fuente que corresponde a la instrucción.

> Nota de tr.: Será dificil que coincida la numeración de líneas con la nuestra, pero buscando cerca de la dirección `0x160` deberíamos encontrar el mismo código ensamblador que el que se muestra a continuación, con las instrucciones que nos interesan.

```
$ cat debug.dump
[...]
00000158 <main>:
     158:      	push	{r7, lr}
     15a:      	mov	r7, sp
     15c:      	bl	0x160 <registers::__cortex_m_rt_main::h0b7888ca966441cf> @ imm = #0x0

00000160 <registers::__cortex_m_rt_main::h0b7888ca966441cf>:
     160:      	push	{r7, lr}
     162:      	mov	r7, sp
     164:      	sub	sp, #0x8
     166:      	bl	0x198 <registers::init::hb6346637538e8ec5> @ imm = #0x2e
     16a:      	movw	r1, #0x504        ; <-- Load lower half of `OUT` register address into register `r1`
     16e:      	movt	r1, #0x5000       ; <-- Load upper half of `OUT` register address into register `r1`
     172:      	str	r1, [sp, #0x4]
     174:      	ldr	r0, [r1]          ; <-- (16) Load value at the address in `r1` into `r0`.
     176:      	orr	r0, r0, #0x200000 ; <-- (16) Bitwise OR the value in `r0` with `0x200000`, and store in `r0`
     17a:      	str	r0, [r1]          ; <-- (16) Store contents of `r0` in memory at address from `r1`
     17c:      	ldr	r0, [r1]          ; <-- (19) Load value at the address in `r1` into `r0`.
     17e:      	orr	r0, r0, #0x80000  ; <-- (19) Bitwise OR the value in `r0` with `0x80000`, and store in `r0`
     182:      	str	r0, [r1]          ; <-- (19) Store contents of `r0` in memory at address from `r1`
     184:      	ldr	r0, [r1]          ; <-- (22) Load value at the address in `r1` into `r0`.
     186:      	bic	r0, r0, #0x200000 ; <-- (22) Bitwise AND the value in `r0` with bitwise complement of `0x200000`, and store in `r0`
     18a:      	str	r0, [r1]          ; <-- (22) Store contents of `r0` in memory at address from `r1`
     18c:      	ldr	r0, [r1]          ; <-- (25) Load value at the address in `r1` into `r0`.
     18e:      	bic	r0, r0, #0x80000  ; <-- (25) Bitwise AND the value in `r0` with bitwise complement of `0x80000`, and store in `r0`
     192:      	str	r0, [r1]          ; <-- (25) Store contents of `r0` in memory at address from `r1`
     194:      	b	0x196 <registers::__cortex_m_rt_main::h0b7888ca966441cf+0x36> @ imm = #-0x2
     196:      	b	0x196 <registers::__cortex_m_rt_main::h0b7888ca966441cf+0x36> @ imm = #-0x4
[...]
```
Como podemos ver, el ensamblado no optimizado contiene 4 instrucciones de carga, 4 de almacenamiento y 4 de manipulación de bits. Se corresponden perfectamente con el código que escribimos. Ahora, echemos un vistazo al ensamblado optimizado.


```
$ cat release.dump
[...]
00000158 <main>:
     158:      	push	{r7, lr}
     15a:      	mov	r7, sp
     15c:      	bl	0x160 <registers::__cortex_m_rt_main::h1f38525e07b97485> @ imm = #0x0

00000160 <registers::__cortex_m_rt_main::h1f38525e07b97485>:
     160:      	push	{r7, lr}
     162:      	mov	r7, sp
     164:      	bl	0x17a <registers::init::h4390f1d4f8a071f7> @ imm = #0x12
     168:      	movw	r0, #0x504          ; <-- Load lower half of `OUT` register address into register `r0`
     16c:      	movt	r0, #0x5000         ; <-- Load upper half of `OUT` register address into register `r0`
     170:      	ldr	r1, [r0]                ; <-- (?) Load value at the address in `r0` into `r1`.
     172:      	bic	r1, r1, #0x280000       ; <-- (?) Bitwise AND the value in `r1` with bitwise complement of `0x280000`, and store in `r1`
     176:      	str	r1, [r0]                ; <-- (?) Store contents of `r0` in memory at address from `r0`
     178:      	b	0x178 <registers::__cortex_m_rt_main::h1f38525e07b97485+0x18> @ imm = #-0x4
[...]
```

¿Ups? Solo una carga, una manipulación de bits y una escritura. El estado de los LED no va a cambiar ya que solo escribe el resultado final, todo apagado. 
La instrucción `str` es la que almacena un valor en el registro. Nuestro programa *debug* (no optimizado) tenía cuatro de ellas, una para cada escritura en el registro, pero el programa *release* (optimizado) solo tiene una.

¿Cómo podemos evitar que el LLVM optimice nuestro programa? Usamos operaciones *volatile* en lugar de lecturas/escrituras normales (`examples/volatile.rs`):


``` rust
{{#include examples/volatile.rs}}
```
Veamos qué pasa ahora si volvemos a generar el ensamblado de la versión optimizada, pero esta vez con operaciones *volatile*.

```
cargo objdump -q --release --example volatile -- --disassemble --no-show-raw-insn  > release.volatile.dump
```

Echemos un vistazo al resultado:

```
$ cat release.volatile.dump
[...]
00000158 <main>:
     158:      	push	{r7, lr}
     15a:      	mov	r7, sp
     15c:      	bl	0x160 <registers::__cortex_m_rt_main::h1f38525e07b97485> @ imm = #0x0

00000160 <registers::__cortex_m_rt_main::h1f38525e07b97485>:
     160:      	push	{r7, lr}
     162:      	mov	r7, sp
     164:      	bl	0x192 <registers::init::h4390f1d4f8a071f7> @ imm = #0x2a
     168:      	movw	r0, #0x504
     16c:      	movt	r0, #0x5000
     170:      	ldr	r1, [r0]
     172:      	orr	r1, r1, #0x200000
     176:      	str	r1, [r0]
     178:      	ldr	r1, [r0]
     17a:      	orr	r1, r1, #0x80000
     17e:      	str	r1, [r0]
     180:      	ldr	r1, [r0]
     182:      	bic	r1, r1, #0x200000
     186:      	str	r1, [r0]
     188:      	ldr	r1, [r0]
     18a:      	bic	r1, r1, #0x80000
     18e:      	str	r1, [r0]
     190:      	b	0x190 <registers::__cortex_m_rt_main::h1f38525e07b97485+0x30> @ imm = #-0x4
[...]
```

Perfecto, hemos conseguido las cuatro instrucciones de carga de nuevo. Revisa el código de nuevo con el GDB para verlo en acción.

