# Tests automatizados en Django

## 1. ¿Qué es un test?

Un test o prueba automatizada es un fragmento de código que comprueba
que una parte de nuestra aplicación funciona como esperamos.

Por ejemplo, podemos comprobar:

-   Que se crea correctamente una tarea.
-   Que una tarea tiene el nombre esperado.
-   Que una petición `POST` devuelve el código HTTP adecuado.
-   Que una petición `GET` no está permitida en una vista que solo
    admite `POST`.
-   Que se devuelve un error cuando el JSON está mal formado.
-   Que las etiquetas se crean si no existen y se reutilizan si ya
    existen.

Django incorpora un sistema de pruebas basado en `unittest` de Python.

## 2. ¿Dónde escribimos los tests?

Normalmente, creamos un archivo llamado `tests.py` dentro de nuestra
aplicación Django:

``` text
proyecto/
├── manage.py
└── tareas/
    ├── models.py
    ├── views.py
    ├── urls.py
    └── tests.py
```

También podemos organizar las pruebas en una carpeta `tests/` cuando el
proyecto tenga muchos tests.

## 3. Nuestro primer test

En `tests.py`:

``` python
from django.test import TestCase


class OperacionesTest(TestCase):

    def test_suma(self):
        resultado = 2 + 3

        self.assertEqual(resultado, 5)
```

### Explicación

-   `TestCase`: clase de Django que proporciona herramientas para
    realizar pruebas.
-   `OperacionesTest`: clase que agrupa pruebas relacionadas.
-   `test_suma`: los métodos de prueba deben comenzar por `test_`.
-   `assertEqual(valor1, valor2)`: comprueba que ambos valores son
    iguales.

Si la condición se cumple, el test pasa. Si no se cumple, Django informa
del fallo.

## 4. Preparar datos con setUp()

En muchos tests necesitamos objetos de la base de datos. Para no repetir
su creación en cada prueba, podemos utilizar el método `setUp()`.

Django ejecuta `setUp()` antes de cada método de prueba.

Supongamos que nuestros modelos tienen una estructura similar a esta:

``` python
class Programador(models.Model):
    nombre = models.CharField(max_length=100)


class Sprint(models.Model):
    fecha_inicio = models.DateField()


class Tarea(models.Model):
    nombre = models.CharField(max_length=100)
    fecha_inicio = models.DateField()
    fecha_fin = models.DateField(null=True, blank=True)
    completa = models.BooleanField(default=False)

    sprint = models.ForeignKey(Sprint, on_delete=models.CASCADE)
    programador = models.ForeignKey(Programador, on_delete=models.CASCADE)
```

El test sería:

``` python
from datetime import date
from django.test import TestCase
from .models import Tarea, Sprint, Programador


class TareaTest(TestCase):

    def setUp(self):
        # Estos objetos estarán disponibles en cada test
        self.programador = Programador.objects.create(
            nombre="Francisco"
        )

        self.sprint = Sprint.objects.create(
            nombre="Sprint 1"
        )

    def test_crear_tarea(self):
        tarea = Tarea.objects.create(
            nombre="Desarrollar el login",
            fecha_inicio=date(2026, 10, 28),
            completa=False,
            sprint=self.sprint,
            programador=self.programador
        )

        self.assertEqual(tarea.nombre, "Desarrollar el login")
        self.assertFalse(tarea.completa)
        self.assertEqual(tarea.sprint, self.sprint)
        self.assertEqual(tarea.programador, self.programador)
```

**Importante:** `setUp()` se ejecuta antes de cada test, no una sola vez
para toda la clase. Django utiliza una base de datos de pruebas aislada.

## 5. Métodos assert más utilizados

Los métodos `assert` permiten comprobar si se cumple una condición.

  -----------------------------------------------------------------------
  Método                              ¿Qué comprueba?
  ----------------------------------- -----------------------------------
  `assertEqual(a, b)`                 Que dos valores son iguales.

  `assertNotEqual(a, b)`              Que dos valores son diferentes.

  `assertTrue(x)`                     Que una condición es verdadera.

  `assertFalse(x)`                    Que una condición es falsa.

  `assertIsNone(x)`                   Que el valor es `None`.

  `assertIsNotNone(x)`                Que el valor no es `None`.

  `assertIn(a, b)`                    Que un elemento está dentro de una
                                      colección.

  `assertNotIn(a, b)`                 Que un elemento no está dentro de
                                      una colección.

  `assertRaises()`                    Que se produce una excepción
                                      determinada.
  -----------------------------------------------------------------------

Ejemplo de `assertRaises()`:

``` python
def test_division_por_cero(self):
    with self.assertRaises(ZeroDivisionError):
        resultado = 10 / 0
```

## 6. Probar las vistas con el cliente de Django

También podemos comprobar que nuestras vistas responden correctamente a
las peticiones HTTP.

Django proporciona un cliente de pruebas mediante `self.client`. No
necesitamos arrancar el servidor para utilizarlo.

Supongamos que nuestra URL para crear tareas es `/tareas/crear/`.

``` python
def test_crear_tarea_api(self):

    datos = {
        "nombre": "Desarrollar el login",
        "fecha_inicio": "2026-10-28",
        "fecha_fin": None,
        "completa": False,
        "sprint_id": self.sprint.id,
        "programador_id": self.programador.id,
        "etiquetas": ["Backend", "SQL"]
    }

    respuesta = self.client.post(
        "/tareas/crear/",
        data=datos,
        content_type="application/json"
    )

    self.assertEqual(respuesta.status_code, 200)
    self.assertEqual(Tarea.objects.count(), 1)
```

### ¿Qué estamos comprobando?

1.  Enviamos una petición `POST` a nuestra vista.
2.  Enviamos los datos en formato JSON.
3.  Comprobamos que la respuesta tiene código HTTP `200`.
4.  Comprobamos que se ha creado una tarea en la base de datos.

La ruta utilizada debe coincidir con la definida en el archivo `urls.py`
del proyecto.

## 7. Comprobar errores en las vistas

También debemos probar los casos en los que algo sale mal.

### 7.1. Comprobar que no se permite GET

Si nuestra vista solo admite peticiones `POST`:

``` python
def test_no_permitir_get(self):
    respuesta = self.client.get("/tareas/crear/")

    self.assertEqual(respuesta.status_code, 405)
```

### 7.2. Comprobar que el JSON está mal formado

``` python
def test_json_incorrecto(self):

    respuesta = self.client.post(
        "/tareas/crear/",
        data="{json incorrecto}",
        content_type="application/json"
    )

    self.assertEqual(respuesta.status_code, 400)
```

### 7.3. Comprobar que el sprint no existe

``` python
def test_sprint_inexistente(self):

    datos = {
        "nombre": "Desarrollar el login",
        "fecha_inicio": "2026-10-28",
        "sprint_id": 9999,
        "programador_id": self.programador.id,
        "etiquetas": []
    }

    respuesta = self.client.post(
        "/tareas/crear/",
        data=datos,
        content_type="application/json"
    )

    self.assertEqual(respuesta.status_code, 400)
    self.assertEqual(Tarea.objects.count(), 0)
```

Este ejemplo presupone que nuestra vista devuelve el código `400` cuando
falla la validación del sprint.

## 8. Probar las etiquetas

En nuestro proyecto, la vista permite recibir una lista de etiquetas. Si
una etiqueta no existe, la crea; si ya existe, la reutiliza.

Podemos comprobar este comportamiento con tests.

### 8.1. Comprobar que se crean las etiquetas

``` python
def test_crear_etiquetas_nuevas(self):

    datos = {
        "nombre": "Desarrollar el login",
        "fecha_inicio": "2026-10-28",
        "sprint_id": self.sprint.id,
        "programador_id": self.programador.id,
        "etiquetas": ["Backend", "SQL"]
    }

    respuesta = self.client.post(
        "/tareas/crear/",
        data=datos,
        content_type="application/json"
    )

    self.assertEqual(respuesta.status_code, 200)
    self.assertEqual(Etiqueta.objects.count(), 2)
```

### 8.2. Comprobar que se reutilizan las etiquetas existentes

``` python
def test_reutilizar_etiqueta(self):

    etiqueta = Etiqueta.objects.create(nombre="Backend")

    datos = {
        "nombre": "Desarrollar el login",
        "fecha_inicio": "2026-10-28",
        "sprint_id": self.sprint.id,
        "programador_id": self.programador.id,
        "etiquetas": ["Backend"]
    }

    respuesta = self.client.post(
        "/tareas/crear/",
        data=datos,
        content_type="application/json"
    )

    self.assertEqual(respuesta.status_code, 200)
    self.assertEqual(Etiqueta.objects.count(), 1)

    tarea = Tarea.objects.get(nombre="Desarrollar el login")

    self.assertIn(etiqueta, tarea.etiquetas.all())
```

Estos ejemplos presuponen que existe el modelo `Etiqueta`, que `Tarea`
tiene una relación ManyToMany con él y que la vista asocia las etiquetas
recibidas.

## 9. Ejecutar los tests

Desde la terminal, en la carpeta donde se encuentra `manage.py`:

Ejecutar todos los tests del proyecto:

``` bash
python manage.py test
```

Ejecutar los tests de una aplicación concreta:

``` bash
python manage.py test tareas
```

Ejecutar una clase concreta:

``` bash
python manage.py test tareas.tests.TareaTest
```

Ejecutar un método concreto:

``` bash
python manage.py test tareas.tests.TareaTest.test_crear_tarea
```

## 10. ¿Qué ocurre con la base de datos?

Django crea automáticamente una base de datos de pruebas independiente.

El proceso habitual es:

1.  Crea la base de datos de pruebas.
2.  Prepara las tablas necesarias.
3.  Ejecuta los tests.
4.  Comprueba los resultados.
5.  Destruye la base de datos de pruebas al finalizar.

Por tanto, los tests no modifican los datos de nuestra base de datos
habitual.


## 11. Actividad propuesta: batería de tests para la API de tareas

Crea una clase `TareaTest` que compruebe el funcionamiento de la API de
gestión de tareas.

Debes implementar los siguientes tests:

1.  **Crear una tarea correctamente:** comprobar que se devuelve el
    código HTTP esperado y que la tarea aparece en la base de datos.
2.  **Método HTTP incorrecto:** comprobar que una petición GET devuelve
    el código `405`.
3.  **JSON mal formado:** comprobar que se devuelve el código `400`.
4.  **Sprint inexistente:** comprobar que se devuelve un error y que no
    se crea la tarea.
5.  **Programador inexistente:** comprobar que se devuelve un error y
    que no se crea la tarea.
6.  **Crear etiquetas nuevas:** enviar dos etiquetas que no existan y
    comprobar que se crean.
7.  **Reutilizar etiquetas:** enviar una etiqueta que ya exista y
    comprobar que no se duplica.
8.  **Varias etiquetas:** comprobar que una tarea puede tener asociadas
    varias etiquetas.

### Condiciones

-   Utiliza `TestCase`.
-   Utiliza `setUp()` para preparar los datos.
-   Utiliza métodos `assert` para comprobar los resultados.
-   Cada prueba debe estar en un método independiente que comience por
    `test_`.
-   Ejecuta las pruebas con `python manage.py test`.

**Objetivo:** aprender a comprobar automáticamente tanto los casos
correctos como los errores que puede producir nuestra API.
