# Resumen final ing2 - Parte 1, análisis de programas

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

