# Windows

## `arm-none-eabi-gdb`


> Hemos descargado e instalado el siguiente:
> [Aquí](https://developer.arm.com/-/media/Files/downloads/gnu/15.2.rel1/binrel/arm-gnu-toolchain-15.2.rel1-mingw-w64-x86_64-arm-none-eabi.msi)


Arm proporciona ejecutables (`.exe`) para Windows. Se pueden encontrar en [gcc](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads), solo hay que seguir las instrucciones.
Justo antes de que finalice el proceso de instalación, hay que marcar/seleccionar la opción "Añadir ruta a la variable de entorno". Si no aparece dicha opción se tendrá que añadir desde el diálogo del sistema y añadir la siguiente ruta para la instalación por defecto:

> C:\Program Files\Arm\GNU Toolchain mingw-w64-x86_64-arm-none-eabi\bin

A continuación, comprobaremos que las herramientas se encuentran en el `%PATH%`:

``` console
$ arm-none-eabi-gcc -v
(..)
gcc version 5.4.1 20160919 (release) (..)
```

En mi caso: 

``` console
$ arm-none-eabi-gcc -v
(..)
gcc version 15.2.1 20251203 (Arm GNU Toolchain 15.2.Rel1 (Build arm-15.86))
```


## PuTTY

Se puede descargar `putty.exe` desde [aquí](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) y añadirlo al `%PATH%`.



Pasa ahora a la [siguiente sección].

[siguiente sección]: verify.md
