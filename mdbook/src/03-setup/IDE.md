# Cómo sacar el máximo partido a tu IDE

Todo el código de este libro parte de la base de que utilizas un terminal sencillo para compilar tu código, ejecutarlo e interactuar con él. Tampoco se hace ninguna suposición sobre el editor de texto que utilices.

Sin embargo, es posible que tengas un IDE favorito, que te ofrezcan autocompletado, sugerencias de tipos, atajos de teclado preferidos y mucho más. En esta sección se explica cómo sacar el máximo partido al IDE utilizando el código obtenido del repositorio de este libro.

# Configuración del IDE

A continuación, te explicamos cómo configurar el IDE para sacar el máximo partido a este libro.
Si no aparece en la lista, te invitamos a mejorar este libro añadiendo una sección, para que el próximo lector pueda disfrutar de la mejor experiencia posible.

## Como configurar IntelliJ o RustRover

Al editar la configuración de compilación de IntelliJ, estos son algunos valores no predeterminados:
* Hay que modificar el comando. Cuando en este libro se indique que ejecutes `cargo embed FLAGS`,
  tendrás que cambiar el valor predeterminado `run` por el comando `embed FLAGS`,
* Se tiene que activar la opción "Emular terminal en la consola de salida". De lo contrario, el programa no podrá mostrar texto en un terminal.
* Debes asegurarte de que el directorio de trabajo sea `microbit/src/N-name`, siendo `N-name` el directorio del capítulo que estás leyendo. No se puede ejecutar desde el directorio `src`, ya que no contiene ningún archivo cargo.
