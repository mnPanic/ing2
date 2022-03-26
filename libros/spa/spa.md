# Notas de "Static Program Analysis"

https://cs.au.dk/~amoeller/spa/

Lunes 28/3/22: caps 1,2, 4 y 5

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
> precise approximations (which is colloquially called the full employment
> theorem for static program analysis designers).
>
> https://en.wikipedia.org/wiki/Full_employment_theorem