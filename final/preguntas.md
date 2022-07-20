# Preguntas

Escrito
1. explicar reaching definitions

    Reaching definitions es un analisis de Dataflow que dice para cada nodo del
    CFG, que definiciones se hicieron y llegaron a él sin ser modificadas. Se
    puede usar por ejemplo para detectar usos de variables no inicializadas.

    Es un análisis forward y may.

    el de reaching definitions es como el de may alias
    si te dice que NO, es porque estás seguro de que no hay reaching definitions, entonces hay un uso de una var no inicializada
    si te dice que SI hay reaching definitions, puede llegar a haber caminos pero tal vez no (porque hay inalcanzables como decís vos)

    lo que es concluyente es que te diga que hay usos de variables no inicializadas, que hay errores

    sound si dem => verdad
    NO hay reaching definitions => hay usos de vars no inicializadas

    complete si verdad => dem
    no hay usos de vars no init => el analisis lo reporta

    no emite alerta sería "no hay reaching definitions"

    un análisis estático de dataflow demuestra la prop "no hay errores" = "no hay ejecuciones con usos de variables no inicializadas"
    ... no me slae explicar esto

2. explicar mutation analysis

    Se trata de generar *mutantes* que son modificaciones leves del programa,
    para medir que tan bueno es un test suite según qué tan bien los detecta.

    Decimos que un test mata a un mutante si el resultado del test es diferente
    del código real (i.e falla y el mutante no, o al revés).

    Para generar mutantes se usan operadores de mutación, que son reglas que
    dicen qué cosas cambiar para derivar un mutante.

3. que es un algoritmo genetico y como se usa para generacion de tests

    Es un algoritmo meta heurístico que sirve para optimizar una función de
    fitness, que nos dice que tan cerca está un valor de cumplir una condición
    (para generar datos de tests)

4. dif entre analisis estatico y dinamico

    estático: analiza el código. Sobreaproxima, entonces es sound (si no hay
    fallas en un superconjunto de las ejecuciones, tampoco las hay en el
    subconjunto) e incomplete (como sobreaproxima, puede ser que diga que hay un
    error que no se corresponda con una falla)

    dinámico: analiza corridas del código. Subaproxima, entonces es complete (si
    tira una alerta, es porque hay ejecuciones, dado que lo ejecuta) pero
    unsound (pueden existir fallas que no detecte, haciendo que haya falsos negativos)

    sound: dem => verdadero

    analisis estático prop = no hay ejecuciones con fallas. Dem = no tira alerta

    dem => verdadero
    no tira alertas => no hay ejecuciones con fallas
    hay fallas => tira alerta

    no hay FN

    complete: verdadero => dem
    no hay ejecuciones con fallas => no tira alertas
    tira alertas => hay ejecuciones

    no hay FP

5. que significa que dos estructuras de datos sean isomorfas y que impacto tiene en la generación automática de tests

    que son estructuralmente iguales, genera redundancia en los tests.

6. como podrias comprobar una propiedad de un programa concurrente

    con model checking. Una prop con LTL por ej. y modelar el sistema con
    FSP/LTS y hacer el model checking que vimos,

    - pasar negacion de la prop a buchi
    - pasar lts a buchi
    - producto
    - si existe cadena en el producto, hay violacion a la prop y no lo cumple

7. Qué es un análisis branch sensitive

  o flow sensitive, es uno que toma o no en cuenta el control flow dentro de un
  procedimiento (ifs, whiles)

8. Qué significa que un análisis de punteros sea context sensitive

    que toma en cuenta el contexto de ejecución con el que se ejecuta una función

9.  Explicar las diferencias y similitudes entre buchis generalizados y no generalizados

    los gbuchi tienen un conjunto de conjuntos de aceptación, y aceptan una
    traza infinita si pasa por al menos uno de cada conjunto infinitas veces

    los buchi tienen un conj de aceptación, y aceptan si pasa por al menos uno
    infinitas veces

10. Explique la relación entre las propiedades de safety y autómatas observadores

    los autómatas observadores modelan todo el comportamiento aceptable, y para
    el que no van a un estado de error. Podemos pensarlo como que detectan los
    contraejemplos finitos a una prop de safety.

11. Explicar la relación de satisfacción de CTL

    ??

12. Randoop: Explique para que sirve, desafios que resuelve y como

    Randoop hace **feedback directed random testing**. Genera casos de tests
    aleatorios que son secuencias de ejecuciones de métodos, guiado por el
    feedback de tests anteriores.

    Arranca con un conjunto de métodos *seed*, y los va extendiendo con otros
    llamados a métodos incrementalmente. Cada vez que extiende los ejecuta, si
    rompe una assertion la da como output y sino la agrega a la lista de
    componentes para tenerla en cuenta.

13. Explique que es cloning-based intert-procedural analysis
14. Que es abstraccion de heap "Allocation-site based"? En que contexto y como se usa?

    Toma como el mismo objeto abstracto a cada lugar en donde se realiza una
    asignación de memoria.

15. Como funciona, como se computa y por que es relevante la bisimulacion debil?

    Es como bisimulación fuerte pero distingue al tau.

    la diferencia es que para imitar un movimiento te podes mover por * taus
    antes y después. Si se movió por tau, 0 o más taus.

16. Que significa que las propiedades de safety sean composicionales?

    Que si S cumple una prop de safety, S || T la cumple para todo T (si S no
    hace algo malo, menos lo va a hacer cuando se componga con T y su
    comportamiento se vea restringido).

    Esto quiere decir que puedo ir demostrando de a partecitas que se cumplen
    ciertas propiedades, y al ir componiendo mi sistema se que sigue valiendo.

17. Buchi generalizado en el contexto de verificacion de sistemas concurrentes.

    ya lo explique antes

18. qué es ejecución simbólica dinámica
19. diferencias randoop y korat
20. cómo probamos propiedades sobre modelos concurrentes
21. Explicar Live Variables Analysis
22. Definir isomorfismo entre dos estructuras de datos y que relevancia tiene en generacion automatica de tests
23. Que es el criterio de no exception top level method coverage y en que contexto se utiliza

Oral
- Me tomo que es point-to analisis

  el pointer analysis o may alias analysis nos responde la pregunta "x may alias
  z?"
  
  si la respuesta es NO: estamos seguros de que no son alias (x != z).
  Si nos dice que SI, pueden ser alias.

- Diferencias entre ejcucion simbolica y simbolica dinamica

  la ejecución simbólica ejecuta simbólicamente el programa, preguntándole a un
  constraint solver si cada branch de un programa es satisfacible. Si no lo
  puede determinar, asume que lo es (sobreaproxima, toma como SAT cosas que son
  UNSAT, sound pero incomplete)

  la ejecución simbólica dinámica además ejecuta el código. Le pide al
  constraint solver si es satisfacible y con qué valores. Si determina que no
  sabe, simplifica las expresiones con los valores concretos y le vuelve a
  preguntar si es satisfacible (esto hace que se pierda de ciertas corridas,
  toma como UNSAT cosas que son SAT).

- El añgoritmo de como chequear una propiedad de u  lts usando ltl
- A mí ttmb JPG, sumo Korat, randoop, live variables, dataflow analysis en gral y creo que algo de LTL
- Aaah eso, diferencia entre korat y randoop
- Me preguntó la diferencia entre cloning e inlining en análisis interprocedural
- Me preguntó los análisis estáticos de dataflow que conociese (los 4 de las diapos)
- Cómo hago para fijarme si un programa concurrente cumple una propiedad (formulas LTL y como convertirlas a automata de buchi, aunque no me pidió el algoritmo con todos sus detalles sino solo los pasos a gran escala)
- Me preguntó que es una función de fitness
- Y sobre el control distance y branch distance
- Y como se combinan para formar una función de fitness
- También las condiciones para poder combinar dos valores de un individuo en una función de fitness (no tienen que ser contradictorias, porque entonces cuando quieras mejorar una vas a estar empeorando la otra y no te sirve de nada)
- Ah sí, diferencia entre automata de buchi y automata de buchi generalizado
- ejemplo de un análisis de dataflow,
- descripción a grandes rasgos el algoritmo de model checking de LTL