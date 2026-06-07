# Dirección `0xBAAAAAAD`
No todas las direcciones de memoria pueden ser accedidas como periféricos. Mira este programa (`examples/bad.rs`).


```rust
{{#include examples/bad.rs}}
```
Esta dirección está muy cerca de la utilizada para `OUT` que usamos antes, pero es *inválida*, en el sentido de que no hay ningún registro en ella.

Intentemos ejecutar el programa.

``` console
$ cargo run
(..)
Resetting and halting target
Target halted
(gdb) continue
Continuing.

Breakpoint 1, registers::__cortex_m_rt_main_trampoline () at src/07-registers/src/main.rs:9
9	#[entry]
(gdb) continue
Continuing.

Program received signal SIGINT, Interrupt.
registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:10
10	fn main() -> ! {
(gdb) continue
Continuing.

Breakpoint 3, cortex_m_rt::HardFault_ (ef=0x2001ffb8) at src/lib.rs:1046
1046	    loop {}
(gdb) 
```
Hemos intentado hacer una operación inválida, leer de memoria que no existe, así que el procesador ha levantado una *excepción*: una excepción de *hardware*.

En la mayoría de los casos, las excepciones se elevan cuando el procesador intenta realizar una operación no posible. Las excepciones rompen el flujo normal de un programa y obligan al procesador a ejecutar un *manejador de excepciones*, que es simplemente una función o subrutina.

Hay diferentes tipos de excepciones. Cada tipo de excepción se genera por diferentes condiciones y cada una es gestionada por un manejador de excepciones diferente.

El crate `registers` que depende del `cortex-m-rt` define un manejador de excepciones *hard fault* por defecto, llamado `HardFault_`, que maneja la excepción de "dirección de memoria inválida". El fichero `embed.gdb` fijó un punto de interrupción en `HardFault` automáticamente; por eso el depurador detuvo el programa mientras se ejecutaba el manejador de excepciones. 

Podemos obtener más información sobre la excepción desde el depurador. Veamos:

```
(gdb) list
1040  #[allow(unused_variables)]
1041	#[doc(hidden)]
1042	#[cfg_attr(cortex_m, link_section = ".HardFault.default")]
1043	#[no_mangle]
1044	pub unsafe extern "C" fn HardFault_(ef: &ExceptionFrame) -> ! {
1045	    #[allow(clippy::empty_loop)]
1046	    loop {}
1047	}
1048	
1049	#[doc(hidden)]
1050	#[no_mangle]
```

El parámetro `ef` es una imagen del estado del programa justo antes de que se produjera la excepción. Inspeccionemos su contenido:

```
(gdb) print/x *ef
$1 = cortex_m_rt::ExceptionFrame {
  r0: 0x5000a784,
  r1: 0x3,
  r2: 0x2001ff24,
  r3: 0x0,
  r12: 0x1,
  lr: 0x4403,
  pc: 0x43ea,
  xpsr: 0x1000000
}
```

Hay muchos datos aquí, pero el más importante es `pc`, el registro del contador de programa. La dirección en este registro apunta a la instrucción que generó la excepción. Desensamblamos el programa alrededor del fallo.


```
(gdb) disassemble /m ef.pc
Dump of assembler code for function core::ptr::read_volatile<u32>:
1654	pub unsafe fn read_volatile<T>(src: *const T) -> T {
   0x000043d2 <+0>:	push	{r7, lr}
   0x000043d4 <+2>:	mov	r7, sp
   0x000043d6 <+4>:	sub	sp, #16
   0x000043d8 <+6>:	str	r0, [sp, #4]
   0x000043da <+8>:	str	r0, [sp, #8]

1655	    // SAFETY: the caller must uphold the safety contract for `volatile_load`.
1656	    unsafe {
1657	        assert_unsafe_precondition!(
   0x000043dc <+10>:	b.n	0x43de <core::ptr::read_volatile<u32>+12>
   0x000043de <+12>:	ldr	r0, [sp, #4]
   0x000043e0 <+14>:	movs	r1, #4
   0x000043e2 <+16>:	bl	0x43f4 <core::ptr::read_volatile::precondition_check>
   0x000043e6 <+20>:	b.n	0x43e8 <core::ptr::read_volatile<u32>+22>

1658	            check_language_ub,
1659	            "ptr::read_volatile requires that the pointer argument is aligned and non-null",
1660	            (
1661	                addr: *const () = src as *const (),
1662	                align: usize = align_of::<T>(),
1663	            ) => is_aligned_and_not_null(addr, align)
1664	        );
1665	        intrinsics::volatile_load(src)
   0x000043e8 <+22>:	ldr	r0, [sp, #4]
   0x000043ea <+24>:	ldr	r0, [r0, #0]          ; <-- That's the one!
   0x000043ec <+26>:	str	r0, [sp, #12]
   0x000043ee <+28>:	ldr	r0, [sp, #12]

1666	    }
1667	}
   0x000043f0 <+30>:	add	sp, #16
   0x000043f2 <+32>:	pop	{r7, pc}

End of assembler dump.

```

La excepción se produjo por la instrucción `ldr r0, [r0, #0]` al leer. Esta sentencia intentó leer la memoria en la dirección indicada por el registro `r0`. Por cierto, un registro de CPU (procesador), no es un registro mapeado en memoria; no tiene una dirección asociada como, por ejemplo, `OUT`.

¿No sería estupendo poder comprobar cuál era el valor del registro `r0` justo en el instante
en que se produjo la excepción? ¡Pues ya lo hemos hecho! El campo `r0` del valor `ef` que mostramos
antes es el valor que tenía el registro `r0` cuando se produjo la excepción. Aquí lo tenemos de nuevo:

```
(gdb) print/x *ef
$1 = cortex_m_rt::ExceptionFrame {
  r0: 0x5000a784,
  r1: 0x3,
  r2: 0x2001ff24,
  r3: 0x0,
  r12: 0x1,
  lr: 0x4403,
  pc: 0x43ea,
  xpsr: 0x1000000
}
```

`r0` contiene el valor `0x5000_A784` que es la dirección inválida con la que llamamos a la función `read_volatile`.
