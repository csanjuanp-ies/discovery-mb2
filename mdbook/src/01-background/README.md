# Antecedentes
Estás a punto de escribir código Rust "a nivel de sistemas" (bare-metal) para un microcontrolador. 
Quizá nunca hayas hecho nada parecido antes. Eso es *fantástico*: ¡bienvenido a una aventura increíble!


Lo primero que debemos hacer es responder algunas preguntas.
* **¿Qué es un microcontrolador?**

  Un microcontrolador es un *sistema* en un chip. Mientras que el ordenador está compuesto por varios
  componentes: un procesador, RAM, almacenamiento, puertos Ethernet, etc.; un
  microcontrolador tiene todos esos tipos de componentes integrados en un solo "chip" o paquete. Esto
  hace posible construir sistemas más compactos.

* **¿Qué puedo hacer con un controlador?**

  Muchas cosas. Los microcontroladores son la parte central de lo que se conoce como "sistemas *embebidos*".
  Estos sistemas están en todas partes, pero normalmente no los vemos. Controlan las máquinas que
  lavan la ropa, imprimen documentos y cocinan la comida. Los sistemas embebidos mantienen los edificios
  en los que se vive y trabaja a una temperatura cómoda, y controlan los componentes que hacen que los vehículos en los que viajamos funcionen.

  La mayor parte de los sistemas embebidos hacen su trabajo sin intervención del usuario. Incluso si implementan una interfaz de usuario como lo hace una lavadora; la mayor parte de su operación se realiza por sí sola.

  Los sistemas embebidos se usan para *controlar* procesos físicos. Para hacer esto posible, tienen uno o más dispositivos para indicar el estado del mundo ("sensores"), y uno o más dispositivos que permiten cambiar las cosas ("actuadores"). Por ejemplo, un sistema de control climático de un edificio podría tener:

    - Sensores que miden la temperatura y la humedad en varias ubicaciones.
    - Actuadores que controlan la velocidad de los ventiladores.
    - Actuadores que hacen que aumentan o disminuyen la temperatura del edificio.

* **¿Cuándo tenemos que usar un microcontrolador?**

  Muchos de los sistemas embebidos mencionados anteriormente podrían implementarse con una computadora que ejecute Linux (por ejemplo, una "Raspberry Pi"). ¿Por qué usar un microcontrolador en su lugar? Parece que podría ser más difícil desarrollar un programa.

  Algunas de las razones son:

    * *Coste:* Un microcontrolador es mucho más barato que una computadora de propósito general. No solo el microcontrolador es más barato; también requiere muchos menos componentes eléctricos externos para funcionar. Esto hace que el circuito impreso (PCB) sea más pequeño y barato de diseñar y fabricar.

    * *Consumo energético:* La mayoría de los microcontroladores consumen una fracción de la energía de un procesador completo. Para aplicaciones que funcionan con baterías, marca una gran diferencia.
  
    * *Capacidad de respuesta:* Para cumplir su propósito, algunos sistemas embebidos deben reaccionar dentro de un intervalo de tiempo limitado (por ejemplo, el sistema de frenado "abs" de un automóvil). Si no se cumple con este tipo de *plazo*, podría ocurrir un fallo catastrófico. Tal plazo se llama requisito de "tiempo real duro". Un sistema embebido que está sujeto a tal plazo se conoce como "sistema de tiempo real duro". Una computadora y un sistema operativo de propósito general generalmente tienen muchos componentes de software que comparten los recursos de procesamiento de la computadora. Esto hace que sea más difícil garantizar la ejecución de un programa dentro de límites de tiempo estrictos.

    * *Fiabilidad:* En sistemas con menos componentes (tanto de hardware como de software), ¡hay menos cosas que pueden salir mal!

* **¿Cuándo *no* debo usar controladores?**

  Los microcontrolasdores no son adecuados para procesos computacionalmente intensos. Para mantener el costo y el consumo de energía bajos, los microcontroladores tienen recursos disponibles limitados.

  Los microcontroladores pueden ejecutar generalmente menos instrucciones por segundo que sus hermanos mayores. Las partes más lentas podrían ejecutarse a "solo" unos pocos millones de instrucciones por segundo. Además, la cantidad de trabajo por instrucción normalmente es menor. Los elementos internos de los microcontroladores suelen ser "de 32 bits", pero siguen existiendo componentes "de 16 bits": esto puede significar más instrucciones para trabajar con los tipos de datos típicos de Rust. La mayoría de los microcontroladores no tienen o tienen poca "caché", lo que implica que las instrucciones se ejecutarán tan rápido como sea el acceso a la memoria principal.

  Algunos microcontroladores no tienen soporte de hardware para operaciones de coma flotante. En esos dispositivos, realizar una suma de números decimales puede llevar cientos de ciclos de CPU.

  Finalmente, los microcontroladores suelen tener en conjunto, una cantidad limitada de memoria. Los tamaños pueden ser tan pequeños como 16 KB para instrucciones de programa y 4 KB para datos, lo que hace que programar para estos sistemas sea bastante desafiante. Si bien el tamaño de la memoria interna por unidad de costo y consumo de energía está aumentando constantemente, el procesador con el que trabajaremos todavía tiene "solo" 512 KB para instrucciones de programa y 256 KB para datos, mucho menos que el de una "computadora real".

* **¿Por qué no usar C?**
 
  Seguramente no necesite convencerte, ya que probablemente estés familiarizado con las diferencias de lenguaje entre Rust y C. Algo que sí quiero mencionar es la gestión de paquetes. C carece de una solución de gestión de paquetes oficial y ampliamente aceptada, mientras que Rust usa Cargo. Esto hace que el desarrollo sea *mucho* más eficiente. Y, en mi opinión, una gestión de paquetes fomenta la reutilización del código porque las bibliotecas se pueden integrar fácilmente en una aplicación, lo cual también es algo bueno, ya que suelen realizarse pruebas "más exhaustivas" sobre ellas.

* **¿Por qué no usar Rust?**

  O ¿Por qúe preferir C sobre Rust?

  El ecosistema de C es más maduro. Existen soluciones listas para usar para varios problemas. Si se necesita controlar un proceso dependiente del tiempo, puedes comprar uno de los sistemas operativos de tiempo real (RTOS) comerciales existentes y resolver el problema. No hay RTOS comerciales en grado de producción en Rust (al momento de escribir esto), por lo que se tendría que crear uno casi desde cero o probar uno de los que están en desarrollo. Puedes encontrar una lista de estos en el repositorio [Awesome Embedded Rust].


