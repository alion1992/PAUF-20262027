# Unidad 1. Fundamentos de Python: bases y orientación a objetos - Parte 1

Python es un **lenguaje de alto nivel** de programación interpretado
cuya filosofía hace hincapié en la legibilidad de su código. Se trata de
un lenguaje de programación multiparadigma, ya que soporta parcialmente
la orientación a objetos, programación imperativa y, en menor medida,
programación funcional.

Es un **lenguaje interpretado, dinámico y multiplataforma**.

Administrado por *Python Software Foundation*, posee una licencia de
código abierto denominada *Python Software Foundation License*.

------------------------------------------------------------------------

## 1.1. Características y paradigmas

Python es un lenguaje de programación **multiparadigma**. Esto significa
que, más que forzar a los programadores a adoptar un estilo particular
de programación, permite varios estilos:

-   Programación orientada a objetos.
-   Programación imperativa.
-   Programación funcional.

Otros paradigmas están soportados mediante el uso de extensiones.

Python usa **tipado dinámico** y **conteo de referencias para la gestión
de memoria**.

``` python
x = 0
x = "Hola"

x = x + " a todos"
print(x)
```

**Salida:**

``` text
Hola a todos
```

------------------------------------------------------------------------

## 1.2. Variables y tipos de datos

En Python no es necesario declarar el tipo de las variables
explícitamente. Algunos tipos básicos son:

``` python
nombre = "Francisco"
edad = 33
altura = 1.79
vivo = True
```

Aquí hemos creado cuatro variables:

-   `nombre` -\> tipo texto.
-   `edad` -\> entero.
-   `altura` -\> valor decimal.
-   `vivo` -\> valor lógico.

### Tipos básicos en Python

``` python
# int
x = 42
y = -7

# float
pi = 3.1416
temperatura = -2.5

# str
saludo = "Hola, mundo"

# bool
estudiante = True
aprobado = False
```

En Python **no hace falta indicar el tipo de la variable**, se detecta
automáticamente según el valor que le asignemos.

Podemos cambiar el tipo de dato de una variable simplemente asignando
otro valor.

### Comprobar el tipo de una variable

Con la función `type()` podemos comprobar de qué tipo es una variable.

``` python
x = 3.14
print(type(x))
```

**Salida:**

``` text
<class 'float'>
```

------------------------------------------------------------------------

## 1.3. Operadores

Los operadores permiten realizar operaciones con variables y valores.

### Operadores aritméticos

  Operador   Descripción       Ejemplo      Resultado
  ---------- ----------------- ---------- -----------
  `+`        Suma              `5 + 3`            `8`
  `-`        Resta             `5 - 3`            `2`
  `*`        Multiplicación    `5 * 3`           `15`
  `/`        División          `5 / 2`          `2.5`
  `//`       División entera   `5 // 2`           `2`
  `%`        Módulo            `5 % 2`            `1`
  `**`       Potencia          `2 ** 3`           `8`

### Operadores de comparación

  Operador   Descripción     Ejemplo    Resultado
  ---------- --------------- ---------- -----------
  `==`       Igual que       `5 == 5`   `True`
  `!=`       Distinto que    `5 != 3`   `True`
  `>`        Mayor que       `5 > 3`    `True`
  `<`        Menor que       `5 < 3`    `False`
  `>=`       Mayor o igual   `5 >= 5`   `True`
  `<=`       Menor o igual   `3 <= 5`   `True`

### Operadores lógicos

  -------------------------------------------------------------------------
  Operador          Descripción       Ejemplo             Resultado
  ----------------- ----------------- ------------------- -----------------
  `and`             `True` si ambas   `5 > 3 and 4 < 6`   `True`
                    condiciones son                       
                    verdaderas                            

  `or`              `True` si al      `5 > 3 or 4 > 6`    `True`
                    menos una                             
                    condición es                          
                    verdadera                             

  `not`             Niega una         `not(5 > 3)`        `False`
                    condición                             
  -------------------------------------------------------------------------

------------------------------------------------------------------------

## 1.4. Condicionales en Python

Los **condicionales** permiten que un programa tome decisiones
dependiendo de si una condición es verdadera (`True`) o falsa (`False`).

### Condicional simple

Ejecuta un bloque de código **solo si** la condición se cumple.

``` python
edad = 20

if edad >= 18:
    print("Eres mayor de edad")
```

### Condicional doble

Si la condición se cumple, ejecuta un bloque; si no, ejecuta otro.

``` python
edad = 16

if edad >= 18:
    print("Eres mayor de edad")
else:
    print("Eres menor de edad")
```

### Condicional múltiple

Se pueden evaluar varias condiciones usando `elif` (*else if*).

``` python
nota = 7

if nota >= 9:
    print("Sobresaliente")
elif nota >= 7:
    print("Notable")
elif nota >= 5:
    print("Aprobado")
else:
    print("Suspenso")
```

------------------------------------------------------------------------

## 1.5. Bucles en Python

Un **bucle** es una estructura que permite repetir un bloque de
instrucciones varias veces.

En Python existen principalmente dos tipos de bucles: `for` y `while`.

### Bucle `for`

Se utiliza para recorrer elementos de una secuencia (listas, cadenas,
rangos de números, etc.).

``` python
ciclos = ["DAM1", "ASIR1", "DAF1"]

for ciclo in ciclos:
    print(ciclo)
```

También existe la posibilidad de utilizar `range()` para un número
determinado de números.

``` python
for i in range(3):
    print(i)

for i in range(3, 10):
    print(i)
```

### Bucle `while`

Se repite mientras la condición sea verdadera (`True`).

``` python
contador = 0

while contador < 5:
    print(contador)
    contador += 1
```

### Instrucciones útiles en los bucles

#### `break`

Interrumpe el bucle.

``` python
for i in range(10):
    if i == 5:
        break
    print(i)
```

#### `continue`

Salta a la siguiente iteración.

``` python
for i in range(5):
    if i == 2:
        continue
    print(i)
```

#### `else`

Se ejecuta cuando el bucle termina con normalidad.

``` python
for i in range(3):
    print(i)
else:
    print("Bucle terminado")
```

------------------------------------------------------------------------

## 1.6. Listas

Una **lista** es una estructura de datos que permite guardar múltiples
valores en una sola variable. Pueden contener cualquier tipo de dato.

``` python
dias = []

# append: añade un único elemento al final de la lista
dias.append("Lunes")
dias.append("Martes")
dias.append("Miercoles")
print(dias)

# remove: elimina la primera coincidencia
dias.remove("Martes")
print(dias)

# pop: elimina y devuelve un elemento
elem = dias.pop()
print(elem)
print(dias)

# Acceso mediante índices
print(dias[-1])

# extend: añade otra lista al final
fines_semana = ["Sabado", "Domingo"]
dias.extend(fines_semana)
print("Ya esta la lista")
print(dias)

# Eliminar por índice
numeros = [1, 2, 3, 4, 5, 6]
del numeros[1]
print(numeros)

del numeros[:2]
print(numeros)

# Insertar en una posición
numeros.insert(-1, 10)
print(numeros)

# Ordenar
numeros.sort()
print("Voy a imprimir ordenado")
print(numeros)

numeros.sort(reverse=True)
```

------------------------------------------------------------------------

## 1.7. Uso de ficheros

### Abrir un fichero

La función `open()` se utiliza para abrir un fichero en Python. Tiene
dos parámetros importantes:

1.  El nombre del fichero que quieres abrir.
2.  El modo en que quieres abrir el fichero (lectura, escritura, etc.).

``` python
f = open("nombre_fichero.txt", "modo")
```

Los modos más comunes son:

-   `"r"`: leer. El fichero debe existir.
-   `"w"`: escribir. Si no existe, se crea; si existe, se sobrescribe.
-   `"a"`: añadir. Escribe al final sin sobrescribir el contenido
    existente.
-   `"r+"`: leer y escribir.

### Leer de un fichero

``` python
f = open("archivo.txt", "r")
print(f.read())

# Después de trabajar con un fichero, es importante cerrarlo
f.close()
```

Otra forma de leer un fichero es usando `with`. Al terminar el bloque de
código, el fichero se cierra automáticamente.

``` python
with open("variasLineas.txt", "r") as f:
    linea = f.readline()

    while linea:
        print(linea, end="")
        linea = f.readline()
```

### Escribir en un fichero

#### `write()`

``` python
with open("nuevo_fichero.txt", "w") as f:
    f.write("Buenos dias, a por el jueves.\n")
```

#### `writelines()`

``` python
lineas = ["Primera linea\n", "Segunda linea\n", "Tercera linea\n"]

with open("nuevo_fichero.txt", "w") as f:
    f.writelines(lineas)
```

#### Añadir contenido

``` python
with open("nuevo_fichero.txt", "a") as f:
    f.write("Añadiendo esta nueva linea al final.\n")
```

Tras la ejecución del código debemos comprobar que se han creado los
ficheros correspondientes en nuestro directorio raíz.

------------------------------------------------------------------------

## 1.8. Excepciones en Python

### Excepción base

-   **`BaseException`**: clase base para todas las excepciones. No
    deberías manejarla directamente.
-   **`Exception`**: clase base para la mayoría de las excepciones. Se
    suele usar para capturar cualquier excepción que herede de ella.

### Excepciones del sistema

-   **`SystemExit`**: se lanza cuando se usa `sys.exit()` para salir del
    programa.
-   **`KeyboardInterrupt`**: se lanza cuando el usuario interrumpe la
    ejecución del programa, generalmente pulsando `Ctrl+C`.
-   **`GeneratorExit`**: se lanza cuando un generador o *coroutine*
    finaliza.

### Excepciones estándar más comunes

#### `ArithmeticError`

-   `ZeroDivisionError`: división por cero.
-   `OverflowError`: un número es demasiado grande para ser
    representado.
-   `FloatingPointError`: error en operaciones de punto flotante.

#### `LookupError`

-   `IndexError`: un índice está fuera del rango de una secuencia.
-   `KeyError`: una clave no existe en un diccionario.

#### Otras comunes

-   `ValueError`
-   `TypeError`
-   `AttributeError`
-   `ImportError`
-   `ModuleNotFoundError`
-   `NameError`
-   `UnboundLocalError`
-   `FileNotFoundError`
-   `IOError`
-   `OSError`
-   `PermissionError`
-   `FileExistsError`
-   `IsADirectoryError`
-   `RuntimeError`
-   `RecursionError`
-   `MemoryError`
-   `StopIteration`
-   `StopAsyncIteration`
-   `AssertionError`

### Ejemplos del uso de excepciones

#### División por cero

``` python
def divisionZero():
    try:
        resultado = 10 / 0
    except ZeroDivisionError:
        print("No se puede dividir entre cero.")
```

#### Varias excepciones

``` python
def multiple():
    try:
        numero = int(input("Introduce un número: "))
        resultado = 10 / numero
    except ValueError:
        print("Debes introducir un número válido.")
    except ZeroDivisionError:
        print("No se puede dividir entre cero.")
```

#### `finally`

``` python
def conFinally():
    try:
        archivo = open("archivo.txt", "r")
        contenido = archivo.read()
    except FileNotFoundError:
        print("El archivo no se encontró.")
    finally:
        archivo.close()
```

#### Lanzar una excepción con `raise`

``` python
def verificar_numero(numero):
    if numero < 0:
        raise ValueError("El número no puede ser negativo.")
    return numero


def lanzarExcepciones():
    try:
        verificar_numero(-5)
    except ValueError as e:
        print(f"Error: {e}")
```
