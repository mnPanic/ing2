# Resumen final ing2 - Parte 2, sistemas concurrentes

## Modelado

Queremos modelar **sistemas concurrentes**, con multiples threads. Nos interesa
su *reactividad* (qué es lo que hacen en cuanto a la interacción con su ambiente
durante su ejecución).

> Esta no es la única forma de hacerlo. Los sistemas concurrentes pueden ser
> transformacionales (nos interesa solo su output), y los programas secuenciales
> pueden ser reactivos.

Concurrencia != paralelismo (lo vimos tantas veces que me parece que no vale la
pena explicarlo de vuelta acá)

Vamos a escribir programas con FSP (Finite State Processes) y darles semántica
con LTSs (Labeled Transition Systems).

![](../lecciones/img/11/fsp-sintaxis-sem-den.png)

### LTS

Los LTS son autómatas donde las etiquetas representan acciones. Se definen
formalmente con una tupla $P = (S, A, \Delta, S_0)$ donde,

- Estados es el universo de todos los estados
- Act es el universo de todas las acciones observables
- $S \subseteq$ Estados es un conjunto finito
- $A \subseteq Act \cup \{ \tau \}$ es un conj de etiquetas o acciones
- $\Delta \subseteq (S \times A \times S)$ es el conjunto de transiciones
- $S_0 \in S$ es el estado inicial.

Otras definiciones,

- El **alfabeto** de $P$ es

  $$\alpha P = A \setminus \{ \tau \}$$

- Una **ejecución** es una secuencia $s_0, l_0, s_1, l_1, \dots$ donde para cada
  $i$, $s_i, l_i, s_{i+1} \in \Delta$ (secuencia de acciones y estados)

- Una **traza** es una proyección sobre $\alpha P$ de una ejecución (solo
  acciones)

Ejemplo:

![](../lecciones/img/11/lts-def-ej.png)

Ejecución: 0, sitdown, 1, right.get, 2, left.get, 3, eat, 4, left.put, 5,
right.put, 6, arise, 0, sitdown, ...

Traza derivada de la ejecución: sitdown, right.gvet, left.get, eat, left.put,
right.put, arise, sitdown, ...

### FSP

Aspectos importantes del lenguaje

- Lo que representa un evento es que *hubo una interacción*, y se abstrae de
  quién la inició.
- No hay parámetros. Codificamos todo como etiquetas.

Primitivas esenciales

- **Proceso nulo**: `STOP`
- **Action prefix**: `(x -> P)`
- **Recursión**: `(P = x -> P)`
- **Elección**: `(x -> P | y -> Q)`
- **Elección no determinística**: `(x -> P | x -> Q)`
- **Ocultamiento de acciones**: `(P) \ {x}` o `P @ {x}`
- **Extensión del alfabeto**: `(P) + {a_1, ..., a_n}`
- **Composición paralela**: `||S = (P || Q)`

#### Composición Paralela

> Es la operación más importante de cualquier álgebra de proceso.

![](../lecciones/img/11/lts-comp.png)
![](../lecciones/img/11/fsp-comp.png)
![](../lecciones/img/11/lts-comp-ex.png)

Si P y Q son procesos, entonces (P || Q) representa la ejecución concurrente de
P y Q. Nos muestra un proceso paralelo como si fuera secuencial, el
comportamiento entrelazado (interleaved).

Intuitivamente, en M || N

1. Si tengo una acción l que solo forma parte de la interfaz de M, solo
   evoluciono M y dejo N como está
2. Igual pero l no pertenece a la de M y solo a la de N.
3. Si forma parte de ambas interfaces, se **sincronizan** y avanzan al mismo
   tiempo.

> $\tau$ no aplica a la regla 3 porque es solo para la interfaz visible. Los
> procesos avanzan de forma independiente sobre tau, y nunca hay una sync en
> ella aunque sea la misma etiqueta. Siempre matchea con las reglas 1 y 2 de ||

#### Syntactic sugar

- **Procesos y acciones indexadas**

  ```fsp
  BUFF(N=3) = (in[i:0..N] -> out[i] -> BUFF).
  ```

  es equivalente a

  ```fsp
  BUFF = (
    in[0] -> out[0] -> BUFF |
    in[1] -> out[0] -> BUFF |
    in[2] -> out[0] -> BUFF |
    in[3] -> out[0] -> BUFF
  ).
  ```

- **Constantes y rangos**

  ```fsp
  const N = 1
  range R = 0 .. 2*N
  ```

- **Guardas**

  ```fsp
  (when B x -> P | y -> Q)
  ```

  cuando B es verdadera tanto x como y pueden ser elegidas, caso contrario x no
  puede ser elegida.

  ![](../lecciones/img/11/sugar-guardas-2.png)

- **Etiquetado de procesos**

  `a:P` prefija cada acción en el alfabeto de P con a.

  Intuición: queremos repetir la definición del mismo proceso para tener
  distintas instancias. Pero si compongo dos veces lo mismo, como tienen la
  misma etiqueta están sincronizando permanentemente, y queremos distinguirlos.

  ![](../lecciones/img/11/sugar-etiquetado.png)

- **Prefijado de procesos**

  `{a1, ..., an}::P` reemplaza cada accion i del alfabeto P con etiquetas
  `a1.i, ..., an.i`. Además, cada transición `(i -> X)` en P se reemplaza con
  `({a1.i, ..., an.i} -> X)`

  ![](../lecciones/img/11/sugar-prefijado.png)

  el alfabeto de resource cuando está prefjado con `{a, b}::` va a contener
  `a.acquire, b.acquire, a.release, b.release`.

  ![](../lecciones/img/11/sugar-ej-recursos.png)

- **Renombre de acciones**

  Un mapeo de renombre se aplica a un proceso para cambiar los nombres de sus
  acciones. La forma general de este mapeo es

  ```text
  {newlabel_1/oldlabel_1, ..., newlabel_n/oldlabel_n}
  ```

  Permite asegurar que los procesos sincronizan en acciones específicas

  ![](../leccionesimg/11/sugar-ej-renombre.png)

  Queremos que un llamado del cliente sea una sync con recepción de un pedido
  por parte del servidor. Wait debería estar sincronizando con reply del server.

  ![](../leccionesimg/11/sugar-ej-client-server.png)

## Bisimulación

Queremos una noción de equivalencia que sea **congruente** con FSP. Si $P \equiv
Q$, entonces,

- `(a -> P)` $\equiv$ `(a -> Q)`
- `(a -> P | b -> R)` $\equiv$ `(a -> Q | b -> R)`
- `(P || R)` $\equiv$ `(Q || R)`
- `P[f]` $\equiv$ `Q[f]`
- `P \ L` $\equiv$ `Q \ L`

Propuestas que no funcionan (clases de equivalencia demasiado gruesas o finas):

- Isomorfismo y ejecuciones: Distinguen aspectos estructurales irrelevantes
- Trazas: No distingue determinísmo.

Queremos,

- Relación de equivalencia (reflexiva, transitiva y simétrica)
- Abstracta con respecto a la estructura
- Más fina que las trazas
- Consistente con la denotación de LTs a procesos reales
- Congruente con FSP

### Bisimulación fuerte

![](../lecciones/img/12/bisim-fuerte.png)

Una relación entre LTSs es una bisimulación fuerte si para todo par P, Q que
está relacionado, cada uno tiene que poder imitar al otro de tal manera que caen
en un par que sigue dentro de la bisimulación.

Definimos ~ como la unión de todas las bisimulaciones fuertes, y así notamos que
P y Q son bisimilares con P ~ Q.

$$\sim = \bigcup \{ R | R \text{ es bisim fuerte}\}$$

Ejemplo:

![](../lecciones/img/12/cej-iso.png)

Doy la relación de bisimulación fuerte

```text
R = {
  (P0, Q0),
  (P1, Q1),
  (P1, Q3),
  (P2, Q2),
  (P2, Q4)
}
```

Concluimos que `P ~ Q`.

Otro ejemplo:

![](../lecciones/img/12/cej-trazas.png)

Acá no podemos dar una relación de bisimulación entre los estados.

El 0 tiene que estar relacionado con el 0. Una vez que hace toss, el 1 puede
hacer tails, no nos queda otra que vincularlo con Q1. Pero entonces P1 y Q1 no
está bien que estén en la relación, porque Q1 puede hacer heads y P1 no lo puede
imitar. Entonces no son bisimilares.

### Juego de bisimulación fuerte

La bisimulación se puede formular como un juego, en el que

- El atacante quiere mostrar que no son bisimilares
- El defensor que sí.

En el juego,

- El tablero es un par de LTSs (P, Q)
- En cada ronda
  - El atacante elige uno de los elementos de la tupla y lo hace transitar por
    alguna acción (en cada ronda puede elegir un LTS diferente)
  - El defensor toma el otro elemento de la tupla y lo hace transitar por la
    misma.
- El que no puede jugar, pierde
  - Pierde el defensor si el atacante eligió una etiqueta que no puede imitar
  - Pierde el atacante si ni P ni Q tienen transiciones salientes (deadlock)
- Si el defensor nunca pierde, gana (en un juego infinito)

Luego,

- P y Q son fuertemente bisim si el defensor tiene una **estrategia ganadora
  universal** desde (P, Q).
- No son fuertemente bisimilares si el atacante tiene una.

Cuando uso cada uno?

- Para probar que **si** son bisimilares, conviene dar la relación de
  bisimulación.

- Para probar que **no** son bisimilares, conviene dar la estrategia ganadora
  universal del atacante (suele ser práctico describirla con un árbol).

### Algoritmo (análisis) de bisimulación

Algoritmo que en tiempo finito me diga si son bisim o no, de **máximo punto
fijo**

- Arranco con $\sim_0$, todos los pares posibles de estados.
- Calculo la secuencia $\sim_0, \sim_1, \dots$
- Cuando $\sim_n = \sim_{n+1}$, freno.

Agrego a n+1 un par si puedo hacer un paso y caer en el conjunto anterior.
$\sim_n$ son todos los pares que se pueden bisimular en $n$ pasos.

### Bisimulación débil

Para tratar las acciones tau como cómputo interno y no algo que distinga a dos
LTSs.

![](../lecciones/img/12/debil-def.png)

y el juego

![](../lecciones/img/12/juego-debil.png)

La diferencia es que el atacante va a ser jugadas de 1 paso, mientras que el
defensor puede transicionar por los taus. Solo cambia como defiende.

### Equivalencias

![](img/12/eqs.png)

> Otra forma definir la semántica de los LTS a partir de sus árboles de cómputo
> infinitos, y ahí si la equivalencia serían los isomorfismos.

### Minimización

Para calcular la minimización, se puede tomar una relación de bisimulación con
un LTS con si mismo y las clases de equivalencia generadas son los estados del
LTS mínimo.

Q es el mínimo de P si son bisimilares (fuerte o débilmente, depende de la
minimización que hagamos) y además Q es el LTS con menor cantidad de estados que
es equivalente a P.

## Análisis de sistemas concurrentes

### Análisis de modelos formales

Hay dos tipos de procesos que podemos hacer,

- Validación: es un chequeo manual que sirve para incremental la confianza de
  que un modelo formal se corresponde con la realidad.

- Verificación: Garantizar matemáticamente que dos modelos formales están
  vinculados. Puede ser manual, semi manual o automático.

### Estados de error

Los **estados de error** son estados de los que no hay transiciones salientes,
como un deadlock. En la composición paralela, a diferencia de un deadlock, un
estado es de error si *cualquiera* de las dos lo es.

Es decir, el sistema entra en erorr si cualquiera de sus componentes lo hace.

> los deadlocks en la comp paralela no bloquean al otro proceso

### Propiedades de safety y liveness

- **Safety**: Nunca nada malo sucede

  Los contraejemplos son trazas finitas

  Es **composicional**: si S cumple una prop de safety P, entonces para todo T,
  (S || T) cumple P.

  > T restringe el comportamiento, entonces si no hacía algo malo S, menos aún
  > va a hacer con el comportamiento restringido.

- **Liveness**: Algo bueno siempre sucede

  Todos los contraejemplos (salvo deadlocks) son trazas infinitas

  No es composicional. Si tengo que S cumple L, y la compongo con `STOP + \alpha
  S`, no va a hacer nada y no la va a cumplir.

Si P es un autómata observador y ERROR no es alcanzable en S || P, entonces
ERROR tampoco lo es en (S || Q) || P con cualquier Q.

La composicionalidad nos dice que podemos construir modelos de componentes e ir
componiendolos, y saber para modelos de mas alto nivel que la prop se sigue
cumpliendo. Puedo verificar *composicionalmente* la correctitud de mi sistema.
componentes chiquitos y saber que aplica para los grandes.

### Observadores de propiedades

Codificamos una propiedad con un LTS

- Si el sistema la satisface, (S || Obs) ~ S
- Si no, (S || Obs) tiene un estado de error cuya traza muestra la naturaleza
del problema.

Con la kw **property** damos todo el comportamiento aceptable (de forma
**determinística**) y el resto va a error.

### Progreso

Para escribir propiedades de liveness usamos el keyword `progress`,

```fsp
progress P = {a1, .., an}
```

Donde si son las acciones que denotan un *progreso útil* del proceso. Va a ser
verdadera si para cada ejecución infinita, por lo menos una acción aparece
infinitamente bajo la suposición de **fair choice**.

#### Fair choice

Es una condición de *fairness* sobre cómo un scheduler elige las transiciones
que se ejecutan en un LTS.

Una ejecución en un LTS es válida bajo **fair choice** si un estado, al ser
visitado infinitas veces, todas sus transiciones salientes también lo son.

#### Análisis de progreso


La idea es encontrar las componentes fuertemente conexas "terminales" o bottom
de las que no se puede salir:

- Todo estado es alcanzable desde cualquier otro
- No existen transiciones de un estado del conjunto a un estado fuera de él

Bajo fair choice, una vez alcanzado un BSCC (Bottom Strongly Connected
Component) todas las transiciones son inevitables.

### Action priority

Puedo darle más prioridad o menso a ciertas acciones con sintaxis de FSP.



## LTL

## CTL