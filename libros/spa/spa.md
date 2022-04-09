# Notas de "Static Program Analysis"

https://cs.au.dk/~amoeller/spa/

- Lunes 28/3/22: caps 1,2, 4 y 5
- Lunes 4/03/22: cap 10

## Capitulo 1: Introduction

El análisis estático de programas apunta a responder automáticamente preguntas
sobre posibles comportamientos de programas.

### Aplicaciones

- Optimización en compliadores
- Bug finding y verificación
- IDEs para soportar desarrollo de programas.

### Respuestas aproximadas

Como dijo Dijkstra,

> Program testing can be used to show the presence of bugs, but never to show
> their absence.

Queremos garantías sobre lo que pueden llegar a hacer nuestros programas para
todos los inmputs posibles, y queremos que se provean automáticamente, por un
programa. Un **program analyzer** es un programa que toma otros programas como
input y determina si tienen o no cierta propiedad.

> As the ideal analyzer does not exist, there is always room for building more
> precise approximations (which is colloquially called the **full employment**
> **theorem** for static program analysis designers).
>
> https://en.wikipedia.org/wiki/Full_employment_theorem

Las aproximaciones que usamos son *conservativas* (safe) (por ej. aproximar uso
de memoria sería conservativo si nunca el estimativo es menor al uso real). Las
estimaciones conservativas están relacionadas al concepto de *soundness* en
analizadores.

> We say that a program analyzer is **sound** if it never gives incorrect
> results (but it may answer *maybe*). Thus, the notion of soundness depends on
the intended application of the analysis output, which may cause some confusion.
For example, a verification tool is typically called sound if it never misses
any errors of the kinds it has been designed to detect, but it is allowed to
produce spurious warnings (also called false positives), whereas an automated
testing tool is called sound if all reported errors are genuine, but it may miss
errors.

## Capítulo 2: TIP

> leido

## Capítulo 4: Lattice theory

### Motivating example - Sign analysis

$\top$: "dont know", $\bot$: "not a number"

$\sqcup$ join y $\sqcap$ meet

## Capítulo 8: Interprocedural Analysis

El análisis de programas completos conteniendo múltiples funciones y llamados a
funciones.

## Capítulo 10: Pointer Analysis

### Allocation site