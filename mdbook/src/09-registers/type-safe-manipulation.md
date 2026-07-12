# Manipulación segura de tipos

Uno de los registros de `P0`, el registro `IN`, es un registro de solo lectura.

> 6.8.2.4 IN - Pages 145 and 146

Fijémonos en la columna 'Access' de la tabla, solo la letra 'R' está presente. Esto significa que se supone que no podemos escribir en este registro.

Cada registro tiene permisos diferentes de lectura/escritura. Algunos de ellos son solo de escritura, otros pueden ser leídos y escritos, que son solo leídos.

Trabajar directamente con las direcciones de memoria es propenso a errores. Ya vimos que intentar acceder a una dirección de memoria no válida generó una excepción que interrumpió la ejecución del programa.



¿No sería estupendo disponer de una API que permitiera manipular los registros de forma "segura"? Lo ideal sería que la API
tuviera en cuenta estos tres puntos que he mencionado: no manipular las direcciones reales,
respetar los permisos de lectura y escritura; y evitar la modificación de las partes reservadas de un registro.

Muy bien, la tenemos. `registers::init()` devuelve un valor que proporciona una API de tipo seguro para manipular los registros de los puertos `P0` y `P1`.

Como recordaremos: un grupo de registros asociados a un periférico se llama bloque de registros y se encuentra en una región contigua de memoria. En esta API de tipo seguro, cada bloque de registros se modela como una `struct` donde cada uno de sus campos representa un registro. 
Cada campo es un nuevo tipo diferente basado en `u32` que expone una combinación de los siguientes métodos: `read`, `write` o `modify` según sus permisos de lectura/escritura. Finalmente, estos métodos no toman valores primitivos como `u32`, sino que aceptan otro nuevo tipo que se puede construir usando el patrón *builder* y que evita la modificación de las partes reservadas del registro.

La mejor manera de familiarizarse con esta API es migrar el ejemplo anterior a la nueva API (`examples/type-safe.rs`).


```rust
{{#include examples/type-safe.rs}}
```


Lo primero que llama la atención es que no hay direcciones mágicas de por medio. En su lugar, utilizamos una forma más intuitiva, `p0.out`, para referirnos al registro `OUT` del bloque de registros del puerto `P0`.

El bloque de registros tiene un método [`modify`] que toma una closure. Antes de que se llame a esta, el valor del registro `OUT` se lee y se pasa como el parámetro `r`. Dado el valor de `r`, es posible manipular `w` para obtener el nuevo valor del registro utilizando sus métodos. El resultado se escribe en el registro una vez que la closure termina. En nuestro caso, el valor actual del registro también se pasa en el parámetro `w`, lo que nos permite simplemente manipular `w` cuando queramos mantener el resto de los bits del registro tal como están.


El método `modify` se define para los registros que permiten tanto el acceso de escritura como de lectura. Si solo deseamos leer el valor de un registro, pero no actualizarlo, se puede usar el método [`read`]. O, si simplemente queremos escribir un valor de registro sin leerlo, existe el método [`write`].

Los registros de solo lectura implementan el método `read`, y los registros de solo escritura implementan el método `write`. Esto evita que los usuarios accedan a un registro de una manera incorrecta y, por lo tanto, no es necesario envolver las llamadas en un bloque `unsafe`. ¡Y lo más importante, no necesitamos averiguar la dirección exacta del registro y las posiciones de los bits!


[`write`]: https://docs.rs/svd2rust/latest/svd2rust/#write
[`read`]: https://docs.rs/svd2rust/latest/svd2rust/#read
[`modify`]: https://docs.rs/svd2rust/latest/svd2rust/#modify

Si ejecutamos el programa, veremos algo interesante que podemos hacer *mientras* depuramos.

`p0` es una referencia al puerto `P0` del bloque de registros, `print p0` devolverá la dirección base del bloque de registros y `print *p0` imprimirá su valor.


```
$ cargo run
(..)
Target halted
(gdb) set mem inaccessible-by-default off
(gdb) break main.rs:12
Breakpoint 4 at 0x162: main.rs:12. (2 locations)
(gdb) continue
Continuing.

Program received signal SIGINT, Interrupt.
cortex_m_rt::DefaultPreInit () at src/lib.rs:1058
1058	pub unsafe extern "C" fn DefaultPreInit() {}
(gdb) continue
Continuing.

Breakpoint 1, registers::__cortex_m_rt_main_trampoline () at src/07-registers/src/main.rs:7
7	#[entry]
(gdb) continue
Continuing.

Program received signal SIGINT, Interrupt.
registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:8
8	fn main() -> ! {
(gdb) continue
Continuing.

Breakpoint 4.2, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:12
12	    p0.out.modify(|_, w| w.pin21().set_bit());
(gdb) print *p0                                               ; ⬅️ Printing `*p0` here!
$1 = nrf52833_pac::p0::RegisterBlock {
  _reserved0: [0 <repeats 1284 times>],
  out: nrf52833_pac::generic::Reg<nrf52833_pac::p0::out::OUT_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::out::OUT_SPEC>
  },
  outset: nrf52833_pac::generic::Reg<nrf52833_pac::p0::outset::OUTSET_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::outset::OUTSET_SPEC>
  },
  outclr: nrf52833_pac::generic::Reg<nrf52833_pac::p0::outclr::OUTCLR_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::outclr::OUTCLR_SPEC>
  },
  in_: nrf52833_pac::generic::Reg<nrf52833_pac::p0::in_::IN_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::in_::IN_SPEC>
  },
  dir: nrf52833_pac::generic::Reg<nrf52833_pac::p0::dir::DIR_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 3513288704
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::dir::DIR_SPEC>
  },
  dirset: nrf52833_pac::generic::Reg<nrf52833_pac::p0::dirset::DIRSET_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 3513288704
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::dirset::DIRSET_SPEC>
  },
  dirclr: nrf52833_pac::generic::Reg<nrf52833_pac::p0::dirclr::DIRCLR_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 3513288704
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::dirclr::DIRCLR_SPEC>
  },
  latch: nrf52833_pac::generic::Reg<nrf52833_pac::p0::latch::LATCH_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::latch::LATCH_SPEC>
  },
  detectmode: nrf52833_pac::generic::Reg<nrf52833_pac::p0::detectmode::DETECTMODE_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::detectmode::DETECTMODE_SPEC>
  },
  _reserved9: [0 <repeats 472 times>],
  pin_cnf: [nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    } <repeats 11 times>, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
--Type <RET> for more, q to quit, c to continue without paging--c
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }]
}


```
Todos estos tipos nuevos y closures para nada generarán programas más grandes. Si compilamos el programa con la optimización [IOP] habilitada, veremos exactamente las mismas instrucciones que la versión "insegura" que usaba `write_volatile` y las mismas direcciones hexadecimales que tenía!


[IOP]: https://en.wikipedia.org/wiki/Interprocedural_optimization

Utilizamos `cargo objdump` para crear el desemsamblado del código `release.type-safe.dump`:

``` console
cargo objdump -q --release --example type-safe -- --disassemble --no-show-raw-insn  > release.type-safe.dump
```

Buscamos la función `main` en el fichero `release.type-safe.dump`
```
00000158 <main>:
     158:      	push	{r7, lr}
     15a:      	mov	r7, sp
     15c:      	bl	0x160 <registers::__cortex_m_rt_main::h0e9b57c6799332fd> @ imm = #0x0

00000160 <registers::__cortex_m_rt_main::h0e9b57c6799332fd>:
     160:      	push	{r7, lr}
     162:      	mov	r7, sp
     164:      	bl	0x192 <registers::init::hec71dddc40be11b5> @ imm = #0x2a
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
     190:      	b	0x190 <registers::__cortex_m_rt_main::h0e9b57c6799332fd+0x30> @ imm = #-0x4
```
comprobamos que el programa generado es exactamente el mismo que la versión que llamaba a `ptr::read_volatile` y `ptr::write_volatile`.

La mejor parte de esto es que nadie ha tenido que escribir una sola línea de código para implementar la API de GPIO. Todo el código se generó automáticamente a partir de un archivo de descripción de vista del sistema (SVD) utilizando la herramienta [svd2rust]. 
Este archivo SVD es en realidad un archivo XML que los fabricantes de microcontroladores proporcionan y que contiene los mapas de registros de sus microcontroladores. El archivo contiene el diseño de los bloques de registros, las direcciones base, los permisos de lectura/escritura de cada registro, el diseño de los registros, si un registro tiene bits reservados y mucha otra información útil.

[svd2rust]: https://crates.io/crates/svd2rust
