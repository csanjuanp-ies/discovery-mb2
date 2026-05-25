# Linux

Aquí se muestran instrucciones de instalación para algunas distribuciones:

## Ubuntu 20.04 o más reciente / Debian 10 o más reciente

> **NOTA** `gdb-multiarch` es el comando de GDB que utilizarás para depurar los programas de Arm Cortex-M.
``` console
$ sudo apt install gdb-multiarch minicom libunwind-dev
```

## Fedora 32 o posterior

> **NOTA** `gdb` es el comando de GDB que utilizarás para depurar los programas de Arm Cortex-M.
``` console
$ sudo dnf install gdb minicom libunwind-devel
```

## Arch Linux

> **NOTA** `gdb` es el comando de GDB que utilizarás para depurar los programas de Arm Cortex-M.
``` console
$ sudo pacman -S arm-none-eabi-gdb minicom libunwind
```

## Otras distros

> **NOTA** `arm-none-eabi-gdb` es el comando de GDB que utilizarás para depurar los programas de Arm Cortex-M.

Para aquellas distribuciones que no incorporan paquetes, usaremos [Arm's pre-built
toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads). Hay que descargar el fichero "Linux
64-bit" y poner los binarios en un directorio `bin` del PATH. A continuación, mostramos una manera de hacerlo:

``` console
$ mkdir -p ~/local
$ cd ~/local
$ tar xjf /path/to/downloaded/XXX.tar.bz2
```

Después, se añade la ruta a la variable `PATH` en el correspondiente fichero de configuración shell:
(e.j. `~/.zshrc` or `~/.bashrc`):

```
PATH=$PATH:$HOME/local/XXX/bin
```

## reglas udev

Las siguientes instrucciones tienen que tener permisos de administración para poder ejecutarse (son necesarias para usar el puerto USB): p. ej. `sudo`.

Crea el siguiente fichero en `/etc/udev/rules.d`.

``` console
$ cat /etc/udev/rules.d/69-microbit.rules
```

``` text
# CMSIS-DAP for microbit
ACTION!="add|change", GOTO="microbit_rules_end"
SUBSYSTEM=="usb", ATTR{idVendor}=="0d28", ATTR{idProduct}=="0204", TAG+="uaccess"
LABEL="microbit_rules_end"
```

A continuación, recarga las reglas de udev:
``` console
$ sudo udevadm control --reload
```

Si tienes alguna tarjeta conectada al ordenador, desconéctala y vuelve a conectarla, o ejecuta el siguiente comando.

``` console
$ sudo udevadm trigger
```

## Verificación de los permisos

Conecta la placa micro:bit al ordenador mediante un cable USB.

El micro:bit debería aparecer como un dispositivo USB (archivo) en `/dev/bus/usb`. Veamos cómo se ha identificado:

``` console
$ lsusb | grep -i "NXP Arm mbed"
Bus 001 Device 065: ID 0d28:0204 NXP Arm mbed
$ # ^^^        ^^^
```

En caso el microcontrolador esté conectado al bus #1 y se haya enumerado como el dispositivo #65. Esto significa que el fichero
`/dev/bus/usb/001/065` *es* la placa MB2. Vamos a comprobar los permisos:

``` console
$ ls -l /dev/bus/usb/001/065
crw-rw-r--+ 1 nobody nobody 189, 64 Sep  5 14:27 /dev/bus/usb/001/065
```

Los permisos deberían ser `crw-rw-r--+`, fíjate en el `+` al final, a continuación, comprueba los derechos de acceso ejecutando el siguiente comando.

``` console
$ getfacl /dev/bus/usb/001/065
getfacl: Removing leadin '/' from absolute path names
# file: dev/bus/usb/001/065
# owner: nobody
# group: nobody
user::rw-
user:<YOUR-USER-NAME>:rw-
group::rw-
mask::rw-
other::r-
```

Deberías ver tu nombre de usuario en la lista anterior junto con los permisos
`rw-`, si no es así ... Comprueba de nuevo las [reglas udev]
y prueba a recargarlas:

[reglas udev]: linux.md#reglas-udev

``` console
$ sudo udevadm control --reload
$ sudo udevadm trigger
```

Pasa ahora a la [siguiente sección].

[siguiente sección]: verify.md
