# Flashearlo
Flashear es el proceso de mover nuestro programa a la memoria del microcontrolador. Una vez hecho, el microcontrolador ejecutará el programa cada vez que se encienda.

Nuestro programa será el único existente en la memoria. Por esto me refiero a que no hay nada más ejecutándose en el microcontrolador: ni un sistema operativo, ni un "daemon", nada. Nuestro programa tiene control total sobre el dispositivo.


Pasarlo al microcontrolador es muy simple, gracias a `cargo embed`.

Antes de ejecutar nada, vamos a ver lo que realmente sucede con él. Si miras el lado de tu micro:bit con el conector USB hacia arriba, notarás que en realidad hay tres cuadrados. El más grande es un altavoz. Otro es nuestro MCU del que ya hablamos... pero ¿qué propósito tiene el tercero? Este chip es *otro* MCU, un NRF52820 casi tan potente como el NRF52833 que estamos programando. Este chip tiene tres propósitos principales:

1. Permitir el control de energía y reset de nuestro MCU NRF52833 desde el conector USB.
2. Proporcionar un [puente serial a USB] para nuestro MCU.
3. Proporcionar una interfaz para programar y depurar nuestro NRF52833 (este es el propósito más importante para nosotros por ahora).

Este chip actual como un pequeño puente entre nuestra computadora (a la que está conectado a través de USB) y el MCU (al que está conectado a mediante pistas y se comunica usando el protocolo SWD). Este puente nos permite flashear nuevos binarios en el MCU, inspeccionar el estado de un programa a través de un depurador y hacer otras cosas útiles.

Vamos a Flashearlo.

```console
$ cargo embed --example init
  (...)
     Erasing sectors ✔ [00:00:00] [####################################################################################################################################################]  2.00KiB/ 2.00KiB @  4.21KiB/s (eta 0s )
 Programming pages   ✔ [00:00:00] [####################################################################################################################################################]  2.00KiB/ 2.00KiB @  2.71KiB/s (eta 0s )
    Finished flashing in 0.608s
```

Te habrás fijado que `cargo embed` no termina después de mostrar la última línea. Esto es intencionado: no deberías cerrar `cargo embed`, ya que lo necesitamos en este estado para el siguiente paso: ¡depurarlo! Además, habrás notado que `cargo build` y `cargo embed` reciben los mismos flags. Esto se debe a que `cargo embed` realmente ejecuta la compilación y luego flashea el binario resultante en el chip. Esto significa que se puede omitir el paso de `cargo build` de aquí en adelante si queremos flashear el código de inmediato.


[puente serial a USB]: ../10-serial-port/README.md
