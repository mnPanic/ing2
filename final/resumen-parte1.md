<!-- omit in toc -->
# Resumen final ing2 - Parte 1, análisis de programas

- [Tipos de análisis](#tipos-de-análisis)
- [Análisis estático](#análisis-estático)
  - [Soundness y completeness](#soundness-y-completeness)
  - [Dataflow Analysis](#dataflow-analysis)
    - [Esquema](#esquema)
    - [Algoritmo: Chaotic iteration](#algoritmo-chaotic-iteration)
  - [Pointer analysis](#pointer-analysis)
- [Testing automatizado](#testing-automatizado)
  - [Terminología](#terminología)
  - [Mutation testing](#mutation-testing)
  - [Órdenes de mutación](#órdenes-de-mutación)

## Tipos de análisis

- **Dynamic** (runtime)

  Corren el programa y analizan su comportamiento.

- **Static** (compile time)

  Analizan solamente el código fuente.

- **Hybrid** (static + dynamic)

|               | Dynamic                                          | Static                                       |
| ------------- | ------------------------------------------------ | -------------------------------------------- |
| Cost          | Proporcional al tiempo de ejecución del programa | Proporcional al tamaño del programa          |
| Effectiveness | Unsound (puede perderse errores)                 | Incomplete (Puede reportar errores espurios) |

## Análisis estático

Trabajan sobre CFGs (Control Flow Graphs). Abstraen estados que juntan muchos
valores posibles en vez de trabajar con estados concretos, porque no corren el
programa.

Esto hace que sacrifiquen **completitud**. Puede pasar que haya ejecuciones que
no cumplan la propiedad pero como abstraemos nos la perdemos.

> Un proof system es completo si puede demostrar todo lo verdadero.
> i.e no tira alertas para los programas que no tienen fallas.
> no tiene fallas => no tira alertas
>
> tira alertas => tiene fallas
>
> pero podría haber una falla que suceda para un estado abstracto, pero tan
> abstracto que en realidad no pasa en el programa, causando un falso positivo.

En cambio, son sound

> Sound si se puede probar => es verdadero
> no tira alertas => no tiene bugs
> tiene bugs => tira alertas
>
> como estamos sobre representando los inputs, si no tira alertas, como estamos
> sobre un superconjunto de los inputs, en particular vale para todos los inputs
> del programa, entonces podemos afirmar que no tiene bugs.
>
> Por eso es sound.

### Soundness y completeness

En lógica, hay un *proof system* y un *model*. El proof system es un conjunto de
reglas con las cuales uno puede probar propiedades (o *statements*) sobre el
modelo. El modelo es una estructura matemática, como por ejemplo los naturales.

Un proof system L es

- *sound* (o correcto) si las propiedades que puede probar son efectivamente
  verdaderas en el modelo.

  se puede probar => es verdadera

- *complete* si puede probar cualquier propiedad verdadera sobre el modelo.

  propiedad es verdadera => se puede probar

> Muchos proof systems no son sound y complete, ver el [teorema de incompletitud
> de
> gödel](https://en.wikipedia.org/wiki/G%C3%B6del%27s_incompleteness_theorems).
> Puede haber algunas propiedades verdaderas que L no puede probar (incomplete)
> o puede "probar" propiedades falsas junto con las verdaderas (unsound o
> incorrect)

Se pueden ver los análisis estáticos como un proof system para establecer que
ciertas propiedades se cumplen para las *ejecuciones* de programas. El programa
cumple el rol del modelo.

Para un analizador, un programa P cumple con una propiedad R si no emite ninguna
alerta mientras analiza P. Si levanta una alerta, no tiene la propiedad.

Por ejemplo, si un static analyzer A quiere probar la propiedad R "ninguna
ejecucion del programa va a exhibir runtime errors por división por 0",

- A se debería considerar **sound** para R si, cuando dice que R se cumple,
  efectivamente no hay ejecuciones que exhiben un runtime error

  En cambio, sería unsound si dice que la propiedad se cumple cuando en realidad
  no, i.e al menos una ejecución es errónea.

  se puede probar => es verdadera

  traduciendo a lo que quiere decir que se pueda probar y que sea verdadera,

  no emite alertas => no hay ejecuciones

  sii (contrarecíproco)

  hay ejecuciones => emite alertas

  **no hay falsos negativos**, pues

- A se debería considerar **complete** si puede probar R si es verdadera

  si es verdadera, es porque no existen ejecuciones que tiran error.

  Si no existen ejecuciones => no tira alertas
  
  sii (contrarecíproco)

  tira alertas => existen ejecuciones que fallan

  esto quiere decir que **no hay falsos positivos**, ya que toda alerta se
  corresponde a ejecuciones que fallan. Pero puede haber falsos negativos

  que es el contrarecíproco de la definición de lógica

  no lo prueba => la prop no es verdadera

![](img/1-analisis/soundcomplete.jpg)

### Dataflow Analysis

Es un análisis **estático** que razona sobre el *flujo de datos* de un programa.
Sacrifican completeness pero garantizan soundness y terminación.

Formas clásicas:

- Reaching definitions analysis

  Goal: Determinar para cada punto del programa, qué asignaciones se hicieron y
  llegaron a él sin ser sobreescritas.

  El algoritmo es monotónico, IN y OUT crecen y tienen una cota superior, por lo
  tanto siempre termina.

  > Encontrar usos de variables no inicializadas

- Very busy expressions analysis

  Goal: Determinar en el exit de cada nodo cuales son las **very busy**
  expressions.
  
  Decimos que una expresión está very busy si se usa antes de que se redefinan
  las variables que contiene, sin importar el camino que se tome.

  > Reducir tamaño del código

  En el algoritmo los conjuntos se modifican de forma monótona decreciente (no
  estricta, porque se puede elegir el mismo nodo)

- Available expressions analysis

  Goal: Determinar en cada punto del programa qué expresiones fueron computadas
  pero **no** modificadas en ningún camino que lleva a él

  > Evitar recomputar expresiones, como una cache. El valor puede cambiar en el
  > tiempo, pero en un punt ose que tengo el valor disponible y loi puedo usar
  > sin computarlo. Por ej. sirve en un loop

- Live variables analysis

  Goal: determinar para cada punto del programa qué variables podrían estar
  **live** a su salida.

  Una variable está live si hay un camino a un uso de una variable que no la
  redefine.

  > Asignar registros a variables eficientemente. Si sabemos en cada nodo
  > cuantas variables están vivas a la vez, sabemos cuantos registros
  > necesitamos concurrentemente.

Cada uno tiene un *goal* que especifica la información de dataflow que computa

#### Esquema

- In: Conjunto de facts que fueron calculados por el análisis a la entrada de un
  nodo n
- Out: facts a la salida de un nodo n
- Gen: Conjunto de hechos que son generados por el análisis al fluir por el nodo
  n
- Kill: Conjunto de hechos que son eliminados por el análisis al fluir por el
  nodo n

Se pueden clasificar como **forwards** o **backwards** y como **may** o **must**

- forward: calcula OUT a partir de IN
- backwards: calcula IN a partir de OUT
- may: reporta hechos que suceden en algún camino (unión)
- must: reporta hechos que suceden en todos los caminos (interseccion)

|          | May                     | Must                  |
| -------- | ----------------------- | --------------------- |
| Forward  | Reaching definitions    | Available expressions |
| Backward | Live variables analysis | Very busy expressions |

#### Algoritmo: Chaotic iteration

Para computarlos usamos un algoritmo de *iteración caótica*. Se elije un nodo
random y se calculan los in y out. El algoritmo frena cuando llega a un punto
fijo (o está saturado), que es que no cambian los IN y OUT para todos los nodos.

### Pointer analysis

Los punteros permiten que haya *aliasing*, por ejemplo

```c
Circle x = new Circle();
Circle z = ?
x.radius = 1;
// [x.radius == 1]
z.radius = 2;
// [x.radius == ?] depende de si z es un alias de x
y = x.radius;
assert(y == 1)
```

Dos tipos de análisis: may-alias analysis (pointer analysis) y must-alias. Son
problemas duales, pero must requiere una maquinaria más compleja que may.

Se centra en la pregunta: **x may-alias z?**

- No: No hay posibilidad de que x,z sean aliases (x $\neq$ z)
- Si: No se puede determinar si lo son (tal vez lo son, puede haber falsos
  positivos)

Los algoritmos por lo tanto son sound, y sacrifican completeness. Se diferencian
según

sound si puede probar => verdadero
x != z => no son alias

- Como abstraen el **heap**: tienen que particionar un espacio no acotado de objetos concretos
  en una cantidad finita de *objetos abstractos*
  - **Allocation site based**: un objeto abstracto por allocation site
  - **Type based**: uno por tipo
  - **Heap insensitive**: un solo objeto representa todo el heap.

  ![](../lecciones/img/3/heap-abs-schemes.png)

- Como abstraen el **control flow**
  - **Flow sensitivity**: Como modelar control flow *dentro* de procedimientos o
    funciones (intra-procedural)
    - Flow insensitive:
      - Hacen weak updates.
      - Ven a los programas como statements sin orden,
      - nunca matan facts.
      - Suelen alcanzar para may alias.
    - Flow sensitive: strong updates
      - Pueden matar facts
      - suelen ser requeridos para must alias analysis.
  - **Context sensitivity**: Como modelar control flow *a través* de los
    procedimientos o funciones (inter-procedural)
    - Context insensitive
      - analizan cada función una sola vez, sin importar cuantas veces se llaman
      - Son imprecisos porque juntan aliasing facts que vienen de distintos
        contextos de ejecución
      - Pero son muy eficientes, ya que analizan cada proc una sola vez
    - Context sensitive
      - analizan cada proc una vez por cada *abstract calling context*
      - Son relativamente precisos pero caros

- Como tratan **aggregate data types**, arrays y records
  - Arrays: se usa un solo campo `[*]` para representar todos los obj del array,
    haciendo que no se puedan distinguir elementos.
  - Records (structs)
    1. **Field insensitive**: juntar todos los campos de cada record object

        ![](../lecciones/img/3/record-field-insens.png)

        Previene que distingan entre f1 y f2 en a1 o a2

    2. **Field based**: junta cada campo de todos los objetos

        ![](../lecciones/img/3/record-field-based.png)

        No permite distinguir campos entre objetos. Entre a1 y a2 pero si f1 y f2

    3. **Field sensitive**: Cada campo de cada abstract record object está separado.

        ![](../lecciones/img/3/record-field-sens.png)

        Es la más precisa de todas.

Para realizar los análisis, se lleva el programa a **forma normal** (un conjunto
de instrucciones) y se construye un points-to-graph sobre él. El algoritmo para
construirlo es uno de iteración caótica. Para cada statement, se aplica la regla
correspondiente a él hasta que el grafo no cambie.

Una vez que está construido el points-to-graph, decimos que dos variables
may-alias si apuntan al mismo allocation site. Si no lo hacen, estamos seguros
de que no son alias.

## Testing automatizado

- Tipos de tests
  - Caja blanca
  - Caja negra
  - Manual
  - Automatizado
- Pre y post condiciones
- Coverage

### Terminología

1. Un programador crea un **defecto** en el código
2. Cuando es ejecutado, se genera una **infección**
3. La infección se *propaga*
4. Hasta ser observable, momento en el cual se convierte en una
   **falla**

Donde,

- **Error**: desviación de lo esperado o correcto
- **Defecto**: Error en el código, que puede crear una infección que conduce a
  una falla

  > pero no siempre la ejecución de un código defectuoso lleva a una infección

- **Infección**: Un error en el estado del programa que puede conducir a una
  falla

  > Pero no siempre una infección se propaga hasta ser visible

- **Falla**: un error *externamente visible* en el comportamiento del programa

### Mutation testing

- Vamos a juzgar la efectividad de un test suite midiendo que tan bien puede
  encontrar defectos *artificiales*. Los **mutantes** van a ser una versión
  levemente modificada del programa original, que tienen cambios simples
  (emulando typos) que sean sintácticamente válidos.

  Por ejemplo,

  ```diff
  int foo(int x, int y) {
    if (x < y)
  -    return x + y
  +    return x - y // mutante
    return x * y
  }
  ```

- Se van a generar con **operadores de mutación** que dicen qué es lo que hay
  que cambiar. Son reglas para derivar mutantes a partir de un programa.

- Decimos que un mutante está **muerto** si hubo un test que lo detectó (i.e
  pasa para el programa y falla para el mutante o al revés) y **vivo** si no.

  Si hay un mutante vivo, es porque hacen falta más tests.

- Mutation score = #muertos / #total

  (con #total = #muertos + #vivos)

#### Órdenes de mutación

- **FOM** (First Order Mutant)

  - Tiene exactamente una mutación
  - Cada mutación define un conjunto de FOMs
  - #FOM = # referencias a datos $\times$ # referencias a objetos

- **HOM** (Higher Order Mutant)
  - Es una mutación de otra mutación
  - #HOM = $2^{\#FOM} - 1$

#### Mutantes equivalentes

Como las mutaciones son cambios *sintácticos*, no necesariamente alteran la
**semántica** del programa. Pueden ser alcanzados y que no se infecte el estado
del programa, o producir una infección sin propagación.

Por ejemplo

```diff
if (x > 0) {
- if (y > x) {
+ if (y > abs(x)) {
    ...
  }
}
```

pero como en el branch true x > 0, abs(x) = x y ambos programas son
equivalentes. No vamos a poder matar al mutante.

#### Mejoras en performance

Hay muchos mutantes, y hay que correr todos los tests para cada uno, lo cual
puede ser muy lento. Se puede optimizar en diferentes aspectos

- Fewer
  - Sampling de mutantes: no ejecutar todos
  - Mutación selectiva: usar un subconjunto de operadores

    Hago estudios empíricos, me quedo con el minimo cjto de operadores de
    mutación que sea equivalente al de todos.

    Un estudio dice que es ABS, AOR, COR, ROR, UOI.

- Smarter
  - Paralelización
  - Mutación débil: ver afectación en el estado del programa en vez de esperar
    a que sea observable. (fuerte: que mute hasta una falla observable)
  - Uso de coverage: solo ejecutar los tests que pasen por la línea modificada.
  - Impacto
- Faster
  - Mutar bytecode directamente (evita compilar para cada mutante)
  - Meta-mutantes: en lugar de generar distintas versiones del mismo programa,
    generar una única versión distinta que contenga todos los cambios. Y con
    alguna configuración elegir qué mutante usamos, por ej. con un switch. (te
    ahorrás compilación)

### Fuzzing

Random testing (fuzzing) tiene como idea darle inputs random a un programa, y
ver si se comporta "correctamente". Es un caso particular de mutation analysis.

Es un *paradigma* y no algo que usamos out of the box. La generación de inputs
va a depender de cada programa, y se puede especificar por ejemplo con
gramáticas.

Toma como premisa el **infinite monkey theorem**,

> "A monkey hitting keys at random on a typewriter keyboard will produce any
> given text, such as the complete works of Shakespare, with probability
> approaching 1 as time increases".

El monkey es un fuzz testing tool, y un texto es encontrar un input que expone
un bug en el programa siendo testeado.

#### Herramientas reales

- **Monkey tool**: de google para fuzzing en android. Genera una secuencia de
  eventos, puede generar gestos.

- **Cuzz**: De microsoft para apps multithread
  - En los programas concurrentes los bugs se trigerrean con schedules
    particulares de los threads, que no es determinístico.
  - Para forzar un schedule, se introducen automáticamente delays random entre
    instrucciones (`sleep`s)
  
  - **Bug depth** = el número de *ordering constraints* (cuantos threads tienen
    que correr en cierto orden) que un schedule tiene que satisfacer para
    encontrar bug.

    Ejemplos

    ```text
    // thread 1
    t = new T();

    // thread 2
    t.state == 1
    ```

    como hay un solo constraint, depth 1

    ```text
       // thread 1
    1: p = null;

       // thread 2
    1: if (p != null) {
    2:   p.close();
       }
    ```

    bug depth 2 (porque 2.1 antes de 1.1 y 2.2 después de 1.1).

  - típicamente tienen poco depth los bugs. "small test case hyphotesis": si hay
    un bug, hay un test chiquito que lo encuentra.

#### Pros y cons

Pros

- Fácil de implementar
- Se puede probar que tiene buen cubrimiento con suficientes tests
- Puede funcionar con programas de cualquier formato
- Está bueno para vulns de seguridad (encuentra cosas raras que los testers
  humanos no encuentran)

Cons

- Test suite ineficiente (test bloat)
- Puede encontrar bugs que no son tan importantes (tal vez side effects
  benignos)
- Mala cobertura

### Generación automática de tests

#### Korat (Systematic testing)

Usa los tipos para generar los tests, por lo tanto es **whitebox**.

Para poder explorar todas las posibles formas de un tipo sistemáticamente, vamos
a representar estructuras como vectores ordenando los campos de un struct. De
esa manera, cada *forma* de un tipo es un vector de valores de los campos.

Y como enumeramos los vectores?

Algoritmo naive:

- Se selecciona un tamaño máximo K
- Se generan todos los posibles inputs hasta tamaño K
- Se descartan los que no cumplen la precond
- Corre el programa en los que quedan
- Valida los resultados usando la post cond

Problemas:

- Es ineficiente generar inputs y después chequear la pre condición, sería mejor
  generar inputs válidos de una.

En Korat, se *instrumenta* la precondición (func `repOK`) para ver qué campos
accede. Y se van enumerando en ese orden. Itera el último campo visitado por la
pre condición. Cuando se terminan las posibilidades, hace backtracking.

Evita **isomorfismos** permitiendo a lo sumo incrementos de 1 en la numeración
del vector.

Por ejemplo, `repOK` de un árbol binario (usa BFS)

```text
public boolean repOK(BinaryTree bt) {
  if (bt.root == null) return true;
  Set visited = new HashSet();
  List workList = new LinkedList();
  visited.add(bt.root);
  workList.add(bt.root);
  while (!workList.isEmpty()) {
    Node current = workList.removeFirst();
    if (current.left != null) {
      if (!visited.add(current.left)) return false;
      workList.add(current.left);
    } 
    if (current.right !=null) {
      if (!visited.add(current.right)) return false;
      workList.add(current.right);
    } 
  } 
  return true;
}
```

Pros y cons

- Es bueno cuando podemos enumerar todas las posibilidades: estructuras con
  procedimientos simples y en unit testing
- Es débil cuando no se puede enumerar todo, por ej. con enteros floats o
  strings. (DSE para esto)
- Es tan bueno con las pre y post conds.

#### Randoop (Feedback directed random testing)

> RANDom tester for Object Oriented Programs

Idea: Crear de forma **aleatoria** un nuevo test, **guiado por el feedback** de
tests creados previamente. Un test es una secuencia de method calls.

Receta

- Crea nuevas secuencias incrementalmente, extendiendo las previas.
- Ni bien se crea una, se ejecuta
- Usa el resultado para guiar la generación hacia secuencias que creen nuevos
  estados de los objetos.

Inputs: clases a testear, límite de tiempo, conjunto de constraints (o
assertions)

Output: tests que violan las constraints, con estructura

```text
<secuencia de llamados a input classes que arman el estado>
<contrato stated as assertion>
```

Para que sea valido,

- No se deberia violar un contrato durante la ejecucion (secuencias
  **ilegales**)
- La assertion del final debería fallar al ser ejecutada.

Para clasificar cada secuencia que se genera,

![](../lecciones/img/6/randoop-classif.png)
