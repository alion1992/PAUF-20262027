# Ejercicio: Análisis de la jornada 5 de LaLiga con diccionarios

## Objetivo

Practicar diccionarios, diccionarios anidados, bucles, condicionales,
listas, acumuladores y contadores.

## Datos iniciales

``` python
jornada5 = {
    "Rayo Vallecano - Espanyol": {"local":"Rayo Vallecano","visitante":"Espanyol","goles_local":2,"goles_visitante":1},
    "Alavés - Valencia": {"local":"Alavés","visitante":"Valencia","goles_local":0,"goles_visitante":1},
    "Elche - Real Madrid": {"local":"Elche","visitante":"Real Madrid","goles_local":2,"goles_visitante":3},
    "Atlético de Madrid - Osasuna": {"local":"Atlético de Madrid","visitante":"Osasuna","goles_local":4,"goles_visitante":0},
    "Deportivo - Sevilla": {"local":"Deportivo","visitante":"Sevilla","goles_local":0,"goles_visitante":1},
    "Barcelona - Racing": {"local":"Barcelona","visitante":"Racing","goles_local":7,"goles_visitante":2}
}
```

## Reto 1. Mostrar todos los partidos

Recorre el diccionario y muestra cada resultado con el formato
`Local 2 - 1 Visitante`.

## Reto 2. Número total de goles

Calcula el total de goles marcados en todos los partidos sin escribir
manualmente los resultados.

## Reto 3. Partido con más goles

Determina qué partido tuvo más goles sumando los tantos locales y
visitantes. Muestra el partido, resultado y total de goles.

## Reto 4. Victorias locales, visitantes y empates

Cuenta cuántos partidos terminaron con victoria local, empate y victoria
visitante.

## Reto 5. Buscar un equipo

Solicita el nombre de un equipo. Si participa en la jornada, muestra su
partido y resultado. Si no aparece, informa de ello.

## Reto 6. Equipos con 3 o más goles

Crea una lista con los equipos que hayan marcado 3 o más goles.
Comprueba tanto locales como visitantes.

## Reto 7. Mayor goleada

Determina el partido con mayor diferencia de goles. Muestra los equipos,
resultado y diferencia.

## Reto 8. Clasificación de la jornada

Crea `clasificacion = {}` y asigna 3 puntos por victoria, 1 por empate y
0 por derrota. Muestra todos los equipos y sus puntos.

## Reto 9. Estadísticas completas por equipo

Genera automáticamente:

``` python
estadisticas = {
    "Real Madrid": {
        "GF": 3,
        "GC": 2,
        "DG": 1,
        "puntos": 3
    }
}
```

Para todos los equipos. `GF` son goles a favor, `GC` goles en contra,
`DG` diferencia de goles y `puntos` los puntos obtenidos.

## Reto 10. Clasificación ordenada

Usando `estadisticas`, muestra los equipos ordenados por puntos de mayor
a menor. En caso de empate, usa la diferencia de goles como segundo
criterio.

Formato orientativo:

``` text
1. Equipo A - 3 puntos - DG: +4
2. Equipo B - 3 puntos - DG: +2
...
```

Este último reto puede requerir investigar `sorted()`.
