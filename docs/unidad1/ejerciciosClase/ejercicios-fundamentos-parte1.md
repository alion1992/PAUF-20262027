# Ejercicios: Variables, condicionales y bucles

## Ejercicio 1. Control de notas

Realiza un programa que solicite el **nombre de un alumno** y las notas
obtenidas en **3 pruebas**.

El programa debe:

1.  Solicitar el nombre del alumno.
2.  Solicitar las tres notas.
3.  Calcular la nota media.
4.  Mostrar el nombre del alumno y su nota media.
5.  Mostrar la calificación correspondiente:

  Nota media              Calificación
  ----------------------- ---------------
  Menor que 5             Suspenso
  Entre 5 y menor que 7   Aprobado
  Entre 7 y menor que 9   Notable
  Mayor o igual que 9     Sobresaliente

Además, si **alguna de las tres notas es inferior a 3**, deberá indicar
que debe recuperar esa prueba.

------------------------------------------------------------------------

## Ejercicio 2. Tabla de multiplicar

Solicita un **número entero** y, utilizando un bucle `for`, muestra su
tabla de multiplicar del **1 al 10**.

``` text
Introduce un número: 7

7 x 1 = 7
7 x 2 = 14
...
7 x 10 = 70
```

Además, cuenta cuántos resultados son **pares** y cuántos **impares**.

------------------------------------------------------------------------

## Ejercicio 3. Cajero automático

El usuario comienza con un saldo de **1000 €**.

El programa debe mostrar repetidamente este menú utilizando `while`:

``` text
--- CAJERO AUTOMÁTICO ---

1. Consultar saldo
2. Ingresar dinero
3. Retirar dinero
4. Salir
```

Requisitos:

-   No permitir ingresos negativos.
-   No permitir retiradas negativas.
-   No permitir retirar más dinero del saldo disponible.
-   El programa termina únicamente al seleccionar la opción `4`.

------------------------------------------------------------------------

## Ejercicio 4. Radar de velocidad

Un radar registra la velocidad de **10 vehículos**.

Solicita mediante un bucle la velocidad de cada vehículo y clasifícala:

  Velocidad                  Clasificación
  -------------------------- ------------------
  Menor o igual a 120 km/h   Correcta
  Entre 121 y 140 km/h       Exceso leve
  Entre 141 y 160 km/h       Exceso grave
  Mayor de 160 km/h          Exceso muy grave

Al finalizar muestra:

-   Número total de vehículos.
-   Velocidad media.
-   Vehículos dentro del límite.
-   Vehículos con exceso de velocidad.
-   Velocidad máxima registrada.

### Requisito adicional

La velocidad máxima debe calcularse mediante una variable que se
actualice durante el bucle. **No se puede utilizar `max()`**.
