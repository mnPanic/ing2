# Lección 9 - Fuzzing con gramáticas

Un programa puede aceptar inputs con una estructura determinada, como
expresiones aritméticas, dir de mail, URLs, etc. Entonces podemos usar una
gramática para generar aleatoriamente inputs válidos que espere el programa.
Esto va a ser más efectivo que hacer random testing "clásico".

## Pseudocódigo

```text

MAXSYMBOLS = 5

term = "$START"

while number_of_non_terminals(term)>0
 rule = choose random grammar rule
 new_term = apply rule to term
 if number_of_non_terminals(new_term) < MAXSYMBOLS
 term = new_term

return term 
```

## Heramientas

- LangFuzz (js)

  Extiende la gramática de JS con nuevos terminales que son fragmentos de código
  JS que ocasionaron vulns en el pasado, con la idea de atacar fixes incompletos.

- DomFuzz (documentos dom)
- JSFuzz (js)
- CSmith (c)
- XMLMate (doc xml)

## Bibliografía

The Fuzzing Book (https:// www.fuzzingbook.org/) by Andreas Zeller, Rahul
Gopinath, Marcel Böhme, Gordon Fraser, and Christian Holler • Chapter III
Syntactical Fuzzing : Fuzzing with Grammar