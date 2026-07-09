# Calibración del Magenotómetro

Una cosa muy importante que hay que hacer antes de utilizar un sensor e intentar desarrollar una aplicación con él
es comprobar que su lectura sea realmente correcta. Si no fuera así, tendríamos que
calibrar el sensor. También podría estar averiado: comprobar el estado de los sensores antes
y durante su uso es una idea realmente buena siempre que sea posible.

En mi caso, en dos MB2 diferentes, el magnetómetro del LSM303AGR sin calibrar presenta un error bastante
considerable.  (También tengo uno en el que el eje z parece estar averiado; el fabricante dispone de
hardware adicional y un proceso para ayudar a detectar esto, pero no vamos a abordar esa complejidad aquí).

Existe un procedimiento especificado por el fabricante para calibrar el magnetómetro.  La calibración
implica bastantes cálculos matemáticos (matrices), por lo que no la trataremos en detalle aquí: esta [Nota de diseño]
describe el procedimiento si te interesan los detalles.

[Nota de diseño]: https://www.st.com/resource/en/design_tip/dt0103-compensating-for-magnetometer-installation-error-and-hardiron-effects-using-accelerometerassisted-2d-calibration-stmicroelectronics.pdf

Por suerte para nosotros, el grupo CODAL, que desarrolló el software original en C++ para el micro:bit, ya
había implementado el mecanismo de calibración del fabricante (o algo similar) en C++ [aquí].

[aquí]: https://github.com/lancaster-university/codal-microbit-v2/blob/006abf5566774fbcf674c0c7df27e8a9d20013de/source/MicroBitCompassCalibrator.cpp

Podemos encontrar una adaptación de esta calibración de C++ a Rust en `src/lib.rs`. Ten en cuenta que se trata de una
adaptación de Matlab a C++ y de ahí a Rust, y que presenta algunas decisiones interesantes. En concreto,
al leer los valores calibrados, *los ejes se invierten* de modo que, vistos desde arriba con el conector USB
hacia delante, los ejes X, Y y Z del valor calibrado quedan en orientación "estándar" (derecha, adelante, arriba).

El uso de este calibrador se muestra en `src/main.rs`.

La forma en que el usuario realiza la calibración se muestra en este vídeo de la versión en C++. (Ignora la
impresión inicial; la calibración comienza aproximadamente a mitad del vídeo).

<p align="center">
<video src="https://video.microbit.org/support/compass+calibration.mp4" loop="true" autoplay="true" />
</p>

Hay que inclinar el micro:bit hasta que se enciendan todos los LEds de la matriz. El cursor parpadeante
indica cuál es el LED de referencia en ese momento.

Ten en cuenta que el programa de demostración imprime la matriz de calibración. Esta matriz se puede codificar de forma fija en
un programa, como el programa de la brújula del [capítulo 13] (o almacenarla en la memoria flash de alguna forma), para evitar
tener que recalibrar cada vez que el usuario ejecute el programa.


[capítulo 13]: ../../13-led-compass/README.md
