# Keyland IVVES

## Introducción

*(EN CONSTRUCCIÓN)*

## Descripción de datos

Los datos utilizados son datos provenientes de sensores instalados en máquinas, los cuales son recogidos y almacenados cada ciclo de 
producción. En ellos se recogen multitud de parámetros (entre los que podemos encontrar valores como potencias, voltajes, consumos de energía,
potencias aparentes, frecuencias o capacidades), diferenciados por cada una de las partes en las que se divide la máquina.

## Conceptos

### Autoencoder, Sparse Autoencoder, Stacked Sparse Autoencoder

#### Autoencoder

Un autoencoder (o autocodificador) es una herramienta que sirve para, a través de una red neuronal, recrear los datos de entrada de la manera
más fiel posible al original. Ya que los datos de entrada no están etiquetados, se trata de un modelo sin supervisión, en el que
comprimimos en las capas interiores las características más importantes de esas imágenes.

![Autoencoder](media/KeylandIVVES-1.png?raw=true "Funcionamiento de un autoencoder")

El autoencoder está formado por dos partes:

**1. <ins>Encoder</ins>:**
    Primera parte del modelo que comprime la entrada en una representación comprimida de las principales características y puede describirse con la función h = f(x).

**2. <ins>Decoder</ins>:**
    Segunda parte del modelo que descomprime la representación del encoder y puede describirse con la función r = g(h).

La representación del autoencoder puede describirse con la función r = g(f(x)).

![Autoencoder](media/KeylandIVVES-2.png?raw=true "Representación de un autoencoder")

En el caso de que el tamaño de la entrada y de la salida sean iguales al de la capa intermedia, el modelo se limitaría a copiar las
características neurona por neurona, por lo que deberemos imponer una serie de restricciones para evitar casos como este, como pueden ser:

##### <ins>Reducir dimensión de capa oculta</ins>
La capa oculta tiene una menor dimensión que la entrada y la salida, de tal manera que se sintetizan los datos y posteriormente se reconstruyen (técnica similar a la PCA), obteniendo así información sobre las características.

##### <ins>Aumentar la dimensión de la capa oculta</ins>
La capa oculta tiene una mayor dimensión que los datos de entrada y salida, por lo que se aprendería una función de identidad extendida copiando los datos de
un sitio a otro. Para evitar estos casos, utilizamos otras restricciones como la regularización y la dispersión.

#### Sparse Autoencoder

Este tipo de autocodificadores (en castellano, autocodificador escaso) se basa en el concepto de escasez. La escasez en términos matemáticos se
refiere a la presencia mayoritaria de ceros en la entrada del algoritmo, haciendo que la mayoría de la información sea nula. Esto hace que, por
un lado, la información de entrada sea mayoritariamente inservible, y por otro, que su procesamiento sea mayor del necesario debido a ello.

A la hora de aplicar este concepto sobre este tipo de redes, lo que buscamos es la extracción de características infrecuentes, con lo que
aplicamos un factor de dispersión penalizando la activación de las capas activas, de tal manera que la mayoría de las células de dichas capas
permanecerá a 0, quedando activas únicamente las características más diferenciales.

#### Stacked Sparse Autoencoder (o deep autoencoder)

Como evolución de los autocodificadores escasos, tenemos los autocodificadores escasos profundos. La lógica de estos
autocodificadores es la misma, pero teniendo múltiples capas intermedias en vez de sólo una.

Cada una de dichas capas de codificación tiene la misma penalización de escasez, con lo que la información se comprime todavía más, optimizando
también la extracción de esta y la velocidad.

### Anomaly detection - KDTree, IF, EIF

#### KDTree

KDTree es una técnica de particionado de datos a través de la cual se busca rapidez y escalabilidad. Se basa en generar un árbol a partir de
la comparación del valor actual con las apariciones anteriores.

Dicho árbol se genera en un proceso iterativo en el que, primero, se obtiene aleatoriamente una de las dimensiones seleccionadas de los
valores de entrada, después se busca el valor de la mediana entre todas las apariciones de esa dimensión, y por último, se compara el valor
actual con dicho valor de mediana, clasificando dicho valor según sea mayor o menor. Este proceso terminará una vez finalizada la iteración
sobre todas las características seleccionadas.

#### Isolation Forest

Isolation Forest (o bosque de aislamiento), es una técnica de detección de anomalías mediante la cual se buscan valores que no están dentro de
los valores normales a través de la generación de árboles de manera aleatoria (Random Forest).

Cada uno de estos árboles (Isolation Trees) se generan iterando recursivamente sobre las diferentes características seleccionadas de un
conjunto de nodos de entrada, obteniendo un valor aleatorio entre el máximo y el mínimo de los valores de esa característica para hacer una
división del conjunto inicial y generar dos conjuntos según este criterio separando las observaciones dependiendo de si es menor o igual,
o mayor, para continuar con el mismo proceso para el resto de las características. El punto de corte usualmente es cuando sólo hay un
elemento en uno de los nodos o cuando se alcanza un máximo configurable de subniveles.

Dicho proceso se hace una vez por cada uno de los conjuntos datos seleccionados inicialmente para generar el conjunto de árboles. Una vez
tenemos dicho conjunto de árboles, por cada muestra de entrada, se hará un promedio del número de divisiones se han necesitado para dar una
predicción, y dependiendo de este promedio, se evaluará como anómalo o no (a mayor número de divisiones, menor posibilidad de anormalidad).

#### Extended Isolation Forest

Tal y como su nombre indica, el Extended Isolation Forest es una extensión del Isolation Forest. Una de las limitaciones más grandes del
Isolation Forest es que, al generar los árboles dependiendo de la iteración sobre las características de una en una, obtenemos resultados
muy lineales, por lo que hay situaciones en las que su efectividad es mínima debido a la linealidad de las divisiones (al ir iterando sobre
cada una de las características, la división es paralela al eje de dicha característica).

Para ello, el Extended Isolation Forest propone, en vez de iterar sobre cada una de las dimensiones, hace el mismo proceso, pero en vez de
iterar, lo realiza sobre hiperplanos, por lo que hace cada uno de los cortes sobre todas las dimensiones existentes, con lo que en
determinados casos, la aproximación a la realidad es mucho mayor:

![Extended Isolation Forest](media/KeylandIVVES-3.png?raw=true "IF vs EIF")

### Explicabilidad del modelo - LIME

#### Explicabilidad del modelo

La Inteligencia Artificial (IA) ha experimentado un enorme crecimiento en los últimos años. Con nuevos modelos más complejos cada año, los
modelos de IA han empezado a superar al intelecto humano a un ritmo que nadie podía predecir. Al principio, los cálculos eran comprensibles y
evaluables de manera relativamente sencilla por el usuario, pero a medida que la complejidad de los modelos crece (lo cual se ha visto
agravado enormemente con las redes neuronales profundas), la comprensión y evaluación de este tipo de modelos se ha vuelto extremadamente
compleja.

Por ejemplo, si tengo una red neuronal profunda para la detección de osos polares en imágenes, cuando me da un resultado correcto o
incorrecto, no tengo forma de saber en qué se está fijando el algoritmo para predecir el resultado (es decir, podemos intuir que lo que tiene
que buscar son las cualidades más significativas del animal, pero nada nos puede asegurar que no hay algún otro factor dentro de las imágenes
que influye sin que nosotros lo sepamos, dando así una posible distorsión de la predicción en otros escenarios).

Otro ejemplo puede ser el pronóstico de un modelo de aprendizaje profundo sobre una enfermedad a partir de la sintomatología. El modelo predictivo dará un resultado con determinado
nivel de probabilidad a partir de los síntomas detectados, los no detectados, las características del sujeto, las variables ambientales, sociales, económicas... 
Pero el médico que lo evalúa, no sabe con exactitud la motivación de ese diagnóstico o qué factores han sido determinantes en comparación con los demás,
por lo que no tiene ninguna herramienta que le permita verificar lo acertado de la predicción.

Es por ello que en los últimos tiempos la explicabilidad de la inteligencia artificial (o XAI en su acrónimo en inglés) es algo sobre
lo que se están centrando las miradas, y sobre lo que se enfoca LIME.

#### LIME

LIME (Local Interpretable Model-agnostic Explanations) es un modelo que busca abordar esta problemática de manera accesible y sencilla. Las dos
principales características del modelo son:

**1. <ins>Agnóstico</ins>**
    LIME es agnóstico al modelo, de tal manera que puede ser utilizado en prácticamente todos los problemas del dominio.
    
**2. <ins>Explicaciones locales</ins>**
    Los resultados de LIME no son sobre todo el modelo general, sino sobre una muestra específica, en la cual se plasma qué factores son los más influyentes para que el
    resultado sea el reflejado.

*(EN CONSTRUCCIÓN)*

## System overview

*(EN CONSTRUCCIÓN)*
