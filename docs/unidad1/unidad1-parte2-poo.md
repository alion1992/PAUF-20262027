# Unidad 1. Fundamentos de Python: Programación Orientada a Objetos - Parte 2

## 1.9. Introducción a la POO

La **Programación Orientada a Objetos (POO)** organiza el código
alrededor de objetos. Un objeto reúne **datos** (atributos) y
**comportamientos** (métodos).

!!! info "Conceptos fundamentales" - **Clase:** plantilla que define
atributos y métodos. - **Objeto:** instancia concreta de una clase. -
**Atributo:** dato asociado a un objeto. - **Método:** función definida
dentro de una clase.

------------------------------------------------------------------------

## 1.10. Clases y objetos

Las clases se definen con `class`.

``` python
class Persona:
    pass

persona1 = Persona()
persona2 = Persona()

print(type(persona1))
print(isinstance(persona1, Persona))
```

`persona1` y `persona2` son objetos diferentes de la misma clase.

------------------------------------------------------------------------

## 1.11. Constructor `__init__()` y `self`

`__init__()` se ejecuta al crear el objeto. `self` representa al objeto
actual.

``` python
class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad

persona1 = Persona("Ana", 25)
persona2 = Persona("Carlos", 31)

print(persona1.nombre)
print(persona2.nombre)
```

Cada objeto mantiene sus propios valores.

!!! warning "Importante" `self` aparece como primer parámetro de los
métodos de instancia, pero no se pasa manualmente al invocarlos.

------------------------------------------------------------------------

## 1.12. Métodos

``` python
class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad

    def saludar(self):
        print(f"Hola, soy {self.nombre}")

    def es_mayor_edad(self):
        return self.edad >= 18

persona = Persona("Laura", 22)
persona.saludar()
print(persona.es_mayor_edad())
```

Los métodos también pueden recibir parámetros:

``` python
class Cuenta:
    def __init__(self, saldo):
        self.saldo = saldo

    def ingresar(self, cantidad):
        self.saldo += cantidad

cuenta = Cuenta(1000)
cuenta.ingresar(250)
print(cuenta.saldo)
```

------------------------------------------------------------------------

## 1.13. Atributos de instancia y de clase

Los **atributos de instancia** pertenecen a cada objeto. Los **atributos
de clase** son compartidos.

``` python
class Alumno:
    centro = "CIFP Virgen de Gracia"

    def __init__(self, nombre, nota):
        self.nombre = nombre
        self.nota = nota

alumno1 = Alumno("Ana", 8)
alumno2 = Alumno("Luis", 5)

print(alumno1.nombre)
print(alumno2.nombre)
print(Alumno.centro)
```

  Tipo        Pertenece a   Ejemplo
  ----------- ------------- -----------------
  Instancia   Cada objeto   `self.nombre`
  Clase       La clase      `Alumno.centro`

------------------------------------------------------------------------

## 1.14. Encapsulación

La encapsulación permite controlar el acceso a los datos internos.

### Público

``` python
self.nombre = nombre
```

### Protegido por convención

``` python
self._nombre = nombre
```

### Privado mediante *name mangling*

``` python
class Cuenta:
    def __init__(self, saldo):
        self.__saldo = saldo
```

!!! note Python no aplica `private` como Java. `_atributo` expresa una
convención y `__atributo` activa *name mangling*.

------------------------------------------------------------------------

## 1.15. Getters, setters y `@property`

### Getters y setters tradicionales

``` python
class Persona:
    def __init__(self, edad):
        self.__edad = edad

    def get_edad(self):
        return self.__edad

    def set_edad(self, edad):
        if edad < 0:
            raise ValueError("La edad no puede ser negativa")
        self.__edad = edad
```

### Propiedades

``` python
class Persona:
    def __init__(self, edad):
        self.__edad = edad

    @property
    def edad(self):
        return self.__edad

    @edad.setter
    def edad(self, valor):
        if valor < 0:
            raise ValueError("La edad no puede ser negativa")
        self.__edad = valor

persona = Persona(25)
print(persona.edad)
persona.edad = 30
```

------------------------------------------------------------------------

## 1.16. Herencia

La herencia permite crear una clase a partir de otra.

``` python
class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad

    def mostrar_datos(self):
        print(f"{self.nombre} - {self.edad} años")


class Empleado(Persona):
    pass

empleado = Empleado("Ana", 30)
empleado.mostrar_datos()
```

`Empleado` hereda los atributos y métodos de `Persona`.

------------------------------------------------------------------------

## 1.17. `super()`

`super()` permite acceder al comportamiento de la clase padre.

``` python
class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad


class Empleado(Persona):
    def __init__(self, nombre, edad, salario):
        super().__init__(nombre, edad)
        self.salario = salario

empleado = Empleado("Carlos", 32, 25000)
print(empleado.nombre, empleado.salario)
```

------------------------------------------------------------------------

## 1.18. Sobrescritura de métodos

Una clase hija puede redefinir un método heredado.

``` python
class Persona:
    def presentarse(self):
        print("Soy una persona")


class Empleado(Persona):
    def presentarse(self):
        print("Soy un empleado")

Persona().presentarse()
Empleado().presentarse()
```

También podemos reutilizar el método padre:

``` python
class Empleado(Persona):
    def presentarse(self):
        super().presentarse()
        print("Y además soy empleado")
```

------------------------------------------------------------------------

## 1.19. Herencia múltiple y MRO

Python permite heredar de varias clases.

``` python
class Volador:
    def volar(self):
        print("Estoy volando")


class Nadador:
    def nadar(self):
        print("Estoy nadando")


class Pato(Volador, Nadador):
    pass

pato = Pato()
pato.volar()
pato.nadar()

print(Pato.mro())
```

El **MRO (Method Resolution Order)** determina el orden en el que Python
busca los métodos.

------------------------------------------------------------------------

## 1.20. Polimorfismo

Objetos de clases diferentes pueden responder al mismo método.

``` python
class Perro:
    def hacer_sonido(self):
        return "Guau"


class Gato:
    def hacer_sonido(self):
        return "Miau"


animales = [Perro(), Gato()]

for animal in animales:
    print(animal.hacer_sonido())
```

Esto permite escribir código que trabaja con diferentes tipos de objetos
mediante una interfaz común.

------------------------------------------------------------------------

## 1.21. Clases abstractas

El módulo `abc` permite definir clases que sirven como contrato para sus
subclases.

``` python
from abc import ABC, abstractmethod


class Empleado(ABC):
    @abstractmethod
    def calcular_salario(self):
        pass


class Programador(Empleado):
    def __init__(self, salario):
        self.salario = salario

    def calcular_salario(self):
        return self.salario


programador = Programador(2500)
print(programador.calcular_salario())
```

Una subclase debe implementar los métodos marcados con `@abstractmethod`
para poder instanciarse.

------------------------------------------------------------------------

## 1.22. Métodos estáticos

`@staticmethod` define un método que no necesita `self` ni `cls`.

``` python
class Calculadora:
    @staticmethod
    def sumar(a, b):
        return a + b

print(Calculadora.sumar(10, 5))
```

------------------------------------------------------------------------

## 1.23. Métodos de clase

`@classmethod` recibe la clase mediante `cls`.

``` python
class Usuario:
    total_usuarios = 0

    def __init__(self, nombre):
        self.nombre = nombre
        Usuario.total_usuarios += 1

    @classmethod
    def mostrar_total(cls):
        return cls.total_usuarios

Usuario("Ana")
Usuario("Luis")
print(Usuario.mostrar_total())
```

También permite crear constructores alternativos:

``` python
class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad

    @classmethod
    def desde_cadena(cls, datos):
        nombre, edad = datos.split(",")
        return cls(nombre, int(edad))

persona = Persona.desde_cadena("Ana,25")
```

------------------------------------------------------------------------

## 1.24. Métodos especiales

Los métodos especiales o **dunder methods** permiten personalizar
operaciones del lenguaje.

### `__str__()`

``` python
class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad

    def __str__(self):
        return f"{self.nombre} ({self.edad} años)"

persona = Persona("Ana", 25)
print(persona)
```

### `__repr__()`

``` python
def __repr__(self):
    return f"Persona(nombre={self.nombre!r}, edad={self.edad})"
```

### `__eq__()`

``` python
class Producto:
    def __init__(self, codigo, nombre):
        self.codigo = codigo
        self.nombre = nombre

    def __eq__(self, otro):
        if not isinstance(otro, Producto):
            return False
        return self.codigo == otro.codigo
```

  Método       Utilidad
  ------------ --------------------------------
  `__init__`   Inicializar un objeto
  `__str__`    Representación legible
  `__repr__`   Representación para depuración
  `__eq__`     Operador `==`
  `__len__`    Función `len()`
  `__lt__`     Operador `<`
  `__add__`    Operador `+`

------------------------------------------------------------------------

## 1.25. Composición

En composición, un objeto contiene otros objetos.

``` python
class Motor:
    def arrancar(self):
        print("Motor arrancado")


class Coche:
    def __init__(self, marca):
        self.marca = marca
        self.motor = Motor()

    def arrancar(self):
        print(f"Arrancando {self.marca}")
        self.motor.arrancar()

coche = Coche("Toyota")
coche.arrancar()
```

!!! tip "Herencia o composición" - **Herencia:** un `Programador` **es
un** `Empleado`. - **Composición:** una `Empresa` **tiene** empleados.

------------------------------------------------------------------------

## 1.26. Relaciones entre clases

### Asociación

Dos objetos se relacionan y pueden existir independientemente.

``` python
class Profesor:
    def __init__(self, nombre):
        self.nombre = nombre


class Modulo:
    def __init__(self, nombre, profesor):
        self.nombre = nombre
        self.profesor = profesor

profesor = Profesor("Francisco")
modulo = Modulo("PAUF", profesor)

print(modulo.profesor.nombre)
```

### Composición uno a muchos

``` python
class Empresa:
    def __init__(self, nombre):
        self.nombre = nombre
        self.empleados = []

    def contratar(self, empleado):
        self.empleados.append(empleado)
```

------------------------------------------------------------------------

# 1.27. Ejemplo completo: gestión de una empresa

Este ejemplo integra **herencia, encapsulación, propiedades, clases
abstractas, polimorfismo y composición**.

``` python
from abc import ABC, abstractmethod


class Persona:
    def __init__(self, nombre, edad):
        self.nombre = nombre
        self.edad = edad

    def __str__(self):
        return f"{self.nombre} ({self.edad} años)"


class Empleado(Persona, ABC):
    def __init__(self, nombre, edad, salario):
        super().__init__(nombre, edad)
        self.salario = salario

    @property
    def salario(self):
        return self.__salario

    @salario.setter
    def salario(self, valor):
        if valor < 0:
            raise ValueError("El salario no puede ser negativo")
        self.__salario = valor

    @abstractmethod
    def calcular_bonus(self):
        pass


class Programador(Empleado):
    def __init__(self, nombre, edad, salario, lenguaje):
        super().__init__(nombre, edad, salario)
        self.lenguaje = lenguaje

    def calcular_bonus(self):
        return self.salario * 0.10

    def __str__(self):
        return f"{super().__str__()} - Programador de {self.lenguaje}"


class Administrador(Empleado):
    def calcular_bonus(self):
        return self.salario * 0.05


class Empresa:
    def __init__(self, nombre):
        self.nombre = nombre
        self.empleados = []

    def contratar(self, empleado):
        self.empleados.append(empleado)

    def mostrar_empleados(self):
        for empleado in self.empleados:
            print(empleado)
            print(f"Bonus: {empleado.calcular_bonus():.2f} €")


empresa = Empresa("Tech FP")

empresa.contratar(
    Programador("Ana", 27, 28000, "Python")
)

empresa.contratar(
    Administrador("Carlos", 40, 30000)
)

empresa.mostrar_empleados()
```

### ¿Qué conceptos aparecen?

  Concepto          Aplicación
  ----------------- -------------------------------------------------
  Clase             `Persona`, `Empleado`, `Programador`, `Empresa`
  Objeto            Cada empleado creado
  Encapsulación     `__salario`
  Propiedad         `@property`
  Herencia          `Empleado(Persona)`
  Clase abstracta   `Empleado(ABC)`
  Polimorfismo      `calcular_bonus()`
  Sobrescritura     `__str__()`
  Composición       `Empresa` contiene empleados

------------------------------------------------------------------------

# 1.28. Ejercicios propuestos

## Ejercicio 1. Cuenta bancaria

Crea una clase `CuentaBancaria`.

Debe contener:

-   Titular.
-   Saldo privado.
-   Método `ingresar()`.
-   Método `retirar()`.
-   Propiedad para consultar el saldo.
-   No se debe permitir retirar más dinero del disponible.

------------------------------------------------------------------------

## Ejercicio 2. Vehículos

Crea una clase padre `Vehiculo` con:

-   Matrícula.
-   Marca.
-   Modelo.

Crea las clases:

-   `Coche`.
-   `Moto`.

Ambas deben sobrescribir un método llamado `mostrar_tipo()`.

Guarda varios vehículos en una lista y recórrela demostrando el
**polimorfismo**.

------------------------------------------------------------------------

## Ejercicio 3. Tienda

Crea una clase `Producto` con:

-   Código.
-   Nombre.
-   Precio.

Crea una clase `Tienda` que contenga una lista de productos.

Debe permitir:

1.  Añadir productos.
2.  Buscar un producto por código.
3.  Mostrar todos los productos.
4.  Calcular el valor total del inventario.

------------------------------------------------------------------------

## Ejercicio 4. Empleados

Crea una clase abstracta `Empleado` con el método:

``` python
calcular_salario()
```

Crea dos clases hijas:

-   `EmpleadoFijo`.
-   `EmpleadoPorHoras`.

Cada clase debe calcular el salario de manera diferente.

Guarda distintos empleados en una lista y calcula sus salarios mediante
polimorfismo.

------------------------------------------------------------------------

## Ejercicio 5. Biblioteca

Diseña una pequeña aplicación utilizando:

-   `Libro`.
-   `Usuario`.
-   `Prestamo`.
-   `Biblioteca`.

La biblioteca debe permitir prestar y devolver libros.

El ejercicio debe utilizar **composición entre objetos** y controlar
mediante excepciones situaciones como:

-   Libro no encontrado.
-   Libro ya prestado.
-   Usuario inexistente.

------------------------------------------------------------------------

## Resumen

En esta parte hemos trabajado los principales conceptos de POO en
Python:

-   Clases y objetos.
-   Constructores y `self`.
-   Atributos y métodos.
-   Encapsulación.
-   Propiedades.
-   Herencia.
-   `super()`.
-   Sobrescritura.
-   Herencia múltiple y MRO.
-   Polimorfismo.
-   Clases abstractas.
-   Métodos estáticos y de clase.
-   Métodos especiales.
-   Asociación y composición.

Estos conceptos son especialmente importantes antes de trabajar con
frameworks, ya que frameworks como **Django** y **Spring Boot** hacen un
uso intensivo de clases, objetos, herencia y composición.
