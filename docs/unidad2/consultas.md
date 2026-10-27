# Consultas y búsquedas con Django ORM

En esta clase aprenderemos a consultar, filtrar, ordenar y combinar resultados utilizando el ORM de Django. Trabajaremos con los modelos `Programador`, `Proyecto`, `Sprint`, `Etiqueta` y `Tarea`.

## 1. ¿Qué es el ORM?

El ORM (*Object-Relational Mapper*) permite trabajar con la base de datos utilizando objetos y métodos de Python, sin tener que escribir SQL directamente en cada consulta.

Por ejemplo:

```python
Tarea.objects.filter(completa=True)
```

Es una consulta que recupera las tareas cuyo campo `completa` vale `True`.

Importa los modelos desde `models.py`:

```python
from .models import Programador, Proyecto, Sprint, Etiqueta, Tarea
```

Estos ejemplos se pueden ejecutar en una vista, en una función propia o desde la consola interactiva de Django:

```bash
python manage.py shell
```

## 2. Obtener todos los registros: `all()`

El método `all()` devuelve todos los registros de un modelo.

```python
# Todas las tareas
tareas = Tarea.objects.all()

# Todos los programadores
programadores = Programador.objects.all()

# Todos los proyectos
proyectos = Proyecto.objects.all()
```

Podemos recorrer los resultados:

```python
for tarea in Tarea.objects.all():
    print(tarea.nombre)
```

`all()` devuelve un `QuerySet`, es decir, un conjunto de resultados de una consulta.

## 3. Filtrar registros: `filter()`

El método `filter()` devuelve los registros que cumplen las condiciones indicadas.

```python
# Tareas completas
Tarea.objects.filter(completa=True)

# Tareas pendientes
Tarea.objects.filter(completa=False)

# Programadores llamados Ana
Programador.objects.filter(nombre="Ana")

# Proyectos con un nombre exacto
Proyecto.objects.filter(nombre="Aplicación Web")

# Tareas asignadas al programador con ID 1
Tarea.objects.filter(programador_id=1)
```

### Combinar condiciones con AND

Si pasamos varias condiciones a `filter()`, todas deben cumplirse.

```python
# Tareas completas cuyo nombre es "Diseño"
Tarea.objects.filter(completa=True, nombre="Diseño")

# Tareas pendientes del programador con ID 2
Tarea.objects.filter(completa=False, programador_id=2)
```

## 4. Buscar por coincidencia parcial

Django permite buscar textos con distintos *lookups*. Se escriben después del nombre del campo y dos guiones bajos (`__`).

| Lookup | Significado | Ejemplo |
|---|---|---|
| `exact` | Coincidencia exacta | `nombre__exact="Diseño"` |
| `iexact` | Coincidencia exacta sin distinguir mayúsculas | `nombre__iexact="diseño"` |
| `contains` | Contiene el texto | `nombre__contains="API"` |
| `icontains` | Contiene el texto sin distinguir mayúsculas | `nombre__icontains="api"` |
| `startswith` | Empieza por el texto | `nombre__startswith="Crear"` |
| `istartswith` | Empieza por el texto sin distinguir mayúsculas | `nombre__istartswith="crear"` |
| `endswith` | Termina por el texto | `nombre__endswith="web"` |
| `iendswith` | Termina por el texto sin distinguir mayúsculas | `nombre__iendswith="WEB"` |

Ejemplos:

```python
# Tareas cuyo nombre contiene "api"
Tarea.objects.filter(nombre__icontains="api")

# Programadores cuyo nombre empieza por "mar"
Programador.objects.filter(nombre__istartswith="mar")

# Proyectos cuyo nombre contiene "web"
Proyecto.objects.filter(nombre__icontains="web")

# Etiquetas que terminan en "bug"
Etiqueta.objects.filter(nombre__iendswith="bug")
```

La sensibilidad a mayúsculas y minúsculas de algunos lookups puede depender de la base de datos utilizada.

## 5. Búsquedas por relaciones

Podemos atravesar relaciones entre modelos utilizando `__`.

### 5.1. Tareas y programadores

`Tarea` tiene una clave foránea hacia `Programador`.

```python
# Tareas de Ana
Tarea.objects.filter(programador__nombre="Ana")

# Tareas de programadores cuyo nombre contiene "an"
Tarea.objects.filter(programador__nombre__icontains="an")

# Tareas de programadores cuyo primer apellido es García
Tarea.objects.filter(programador__apellido1="García")
```

También podemos buscar directamente por el identificador de la clave foránea:

```python
Tarea.objects.filter(programador_id=1)
```

### 5.2. Tareas, sprints y proyectos

La relación entre `Tarea` y `Proyecto` pasa por `Sprint`.

```python
# Tareas del proyecto con ID 1
Tarea.objects.filter(sprint__proyecto_id=1)

# Tareas del proyecto "Aplicación Web"
Tarea.objects.filter(sprint__proyecto__nombre="Aplicación Web")

# Tareas de proyectos cuyo nombre contiene "web"
Tarea.objects.filter(sprint__proyecto__nombre__icontains="web")

# Sprints del proyecto con ID 1
Sprint.objects.filter(proyecto_id=1)
```

La expresión `sprint__proyecto__nombre` significa: desde la tarea, acceder al sprint relacionado, después al proyecto y finalmente a su nombre.

### 5.3. Tareas y etiquetas

`Tarea` tiene una relación muchos a muchos con `Etiqueta`.

```python
# Tareas con la etiqueta "Urgente"
Tarea.objects.filter(etiquetas__nombre="Urgente")

# Tareas con alguna etiqueta que contenga "bug"
Tarea.objects.filter(etiquetas__nombre__icontains="bug")
```

Una tarea puede tener varias etiquetas. Si una consulta a través de una relación muchos a muchos produce resultados duplicados, podemos utilizar `distinct()`:

```python
Tarea.objects.filter(
    etiquetas__nombre__icontains="bug"
).distinct()
```

Para obtener las etiquetas de una tarea concreta:

```python
tarea = Tarea.objects.get(id=1)

for etiqueta in tarea.etiquetas.all():
    print(etiqueta.nombre)
```

## 6. Comparar valores y fechas

Los *lookups* de comparación permiten filtrar fechas y otros valores.

| Lookup | Significado |
|---|---|
| `gt` | Mayor que |
| `gte` | Mayor o igual que |
| `lt` | Menor que |
| `lte` | Menor o igual que |
| `in` | Pertenece a una lista de valores |
| `isnull` | Es nulo |

Ejemplos con los campos de fecha de `Tarea`:

```python
# Tareas iniciadas a partir del 1 de octubre de 2026
Tarea.objects.filter(fecha_inicio__gte="2026-10-01")

# Tareas finalizadas antes del 1 de noviembre de 2026
Tarea.objects.filter(fecha_fin__lt="2026-11-01")

# Tareas sin fecha de finalización
Tarea.objects.filter(fecha_fin__isnull=True)

# Tareas con fecha de finalización
Tarea.objects.filter(fecha_fin__isnull=False)

# Tareas iniciadas durante 2026
Tarea.objects.filter(fecha_inicio__year=2026)

# Tareas cuyo ID es 1, 3 o 5
Tarea.objects.filter(id__in=[1, 3, 5])
```

Como `fecha_inicio` y `fecha_fin` permiten valores nulos, `isnull=True` resulta útil para localizar tareas sin esa fecha.

## 7. Búsquedas con OR mediante `Q`

Por defecto, varias condiciones en `filter()` se combinan con AND. Para expresar OR utilizamos `Q`.

```python
from django.db.models import Q

# Tareas completas O tareas cuyo nombre contiene "api"
Tarea.objects.filter(
    Q(completa=True) | Q(nombre__icontains="api")
)
```

Otro ejemplo:

```python
# Programadores llamados Ana O cuyo apellido es García
Programador.objects.filter(
    Q(nombre="Ana") | Q(apellido1="García")
)
```

Los operadores principales son:

- `|`: OR, basta con cumplir una condición.
- `&`: AND, deben cumplirse ambas condiciones.
- `~`: NOT, niega una condición.

Ejemplo con NOT:

```python
# Tareas cuyo nombre no contiene "api"
Tarea.objects.filter(~Q(nombre__icontains="api"))
```

También podemos combinar condiciones con paréntesis para controlar cómo se agrupan:

```python
# Tareas completas cuyo nombre contiene "api",
# o tareas pendientes asignadas al programador con ID 1
Tarea.objects.filter(
    Q(completa=True, nombre__icontains="api")
    | Q(completa=False, programador_id=1)
)
```

## 8. Excluir resultados: `exclude()`

`exclude()` devuelve los registros que no cumplen las condiciones indicadas.

```python
# Tareas que no están completas
Tarea.objects.exclude(completa=True)

# Programadores cuyo nombre no es Ana
Programador.objects.exclude(nombre="Ana")

# Tareas que no pertenecen al proyecto con ID 1
Tarea.objects.exclude(sprint__proyecto_id=1)
```

Ten en cuenta que las relaciones opcionales pueden contener valores nulos. Si necesitas un comportamiento concreto para tareas sin sprint o sprints sin proyecto, comprueba expresamente esos casos.

## 9. Obtener un solo objeto: `get()`

`get()` se utiliza cuando esperamos encontrar exactamente un registro.

```python
tarea = Tarea.objects.get(id=1)

print(tarea.nombre)
print(tarea.completa)
```

Si no existe ningún registro, se produce `Tarea.DoesNotExist`. Si la consulta devuelve varios registros, se produce `Tarea.MultipleObjectsReturned`.

Para gestionar la ausencia de un objeto en una vista, podemos utilizar `get_object_or_404()`:

```python
from django.shortcuts import get_object_or_404

tarea = get_object_or_404(Tarea, id=1)
```

Diferencia importante:

```python
Tarea.objects.filter(id=1)  # Devuelve un QuerySet
Tarea.objects.get(id=1)     # Devuelve un objeto Tarea
```

## 10. Ordenar resultados: `order_by()`

```python
# Orden alfabético por nombre
Tarea.objects.order_by("nombre")

# Orden alfabético inverso
Tarea.objects.order_by("-nombre")

# Orden por fecha de inicio
Tarea.objects.order_by("fecha_inicio")

# Ordenar por estado y después por nombre
Tarea.objects.order_by("completa", "nombre")

# Ordenar por nombre del programador
Tarea.objects.order_by("programador__nombre")
```

Un signo menos (`-`) delante del campo indica orden descendente.

## 11. Limitar resultados

Podemos utilizar *slicing* para obtener una parte de los resultados.

```python
# Primeras cinco tareas
Tarea.objects.all()[:5]

# Registros de la sexta a la décima posición
Tarea.objects.all()[5:10]

# Cinco tareas más recientes con fecha de inicio
Tarea.objects.filter(
    fecha_inicio__isnull=False
).order_by("-fecha_inicio")[:5]
```

Es recomendable definir un orden cuando necesitamos que los resultados aparezcan de forma predecible.

## 12. Métodos útiles de los QuerySet

| Método | Para qué sirve | Ejemplo |
|---|---|---|
| `count()` | Cuenta resultados | `Tarea.objects.filter(completa=True).count()` |
| `exists()` | Comprueba si hay resultados | `Tarea.objects.filter(completa=False).exists()` |
| `first()` | Devuelve el primer objeto o `None` | `Tarea.objects.order_by("nombre").first()` |
| `last()` | Devuelve el último objeto o `None` | `Tarea.objects.order_by("nombre").last()` |
| `values()` | Devuelve diccionarios con los campos seleccionados | `Tarea.objects.values("id", "nombre")` |
| `values_list()` | Devuelve tuplas o valores de los campos seleccionados | `Tarea.objects.values_list("nombre", flat=True)` |
| `distinct()` | Elimina duplicados | `Tarea.objects.filter(etiquetas__nombre="Bug").distinct()` |

Ejemplos:

```python
# Número de tareas pendientes del programador 1
Tarea.objects.filter(
    programador_id=1,
    completa=False
).count()

# Comprobar si el proyecto 1 tiene sprints
Sprint.objects.filter(proyecto_id=1).exists()

# Obtener solo los nombres de las tareas
Tarea.objects.values_list("nombre", flat=True)
```

## 13. Búsqueda desde una vista

Vamos a crear una vista que permita buscar tareas por nombre y estado.

En `views.py`:

```python
from django.shortcuts import render
from .models import Tarea


def buscar_tareas(request):
    nombre = request.GET.get("nombre", "").strip()
    estado = request.GET.get("estado", "")

    tareas = Tarea.objects.all()

    if nombre:
        tareas = tareas.filter(nombre__icontains=nombre)

    if estado == "completas":
        tareas = tareas.filter(completa=True)
    elif estado == "pendientes":
        tareas = tareas.filter(completa=False)

    tareas = tareas.order_by("nombre")

    return render(
        request,
        "tareas/buscar.html",
        {
            "tareas": tareas,
            "nombre": nombre,
            "estado": estado,
        }
    )
```

En `urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("tareas/buscar/", views.buscar_tareas, name="buscar_tareas"),
]
```

Plantilla `templates/tareas/buscar.html`:

```html
<h1>Buscar tareas</h1>

<form method="get">
    <label for="nombre">Nombre:</label>
    <input
        type="text"
        id="nombre"
        name="nombre"
        value="{{ nombre }}"
    >

    <label for="estado">Estado:</label>
    <select id="estado" name="estado">
        <option value="">Todas</option>
        <option value="completas"
            {% if estado == "completas" %}selected{% endif %}>
            Completas
        </option>
        <option value="pendientes"
            {% if estado == "pendientes" %}selected{% endif %}>
            Pendientes
        </option>
    </select>

    <button type="submit">Buscar</button>
</form>

<ul>
    {% for tarea in tareas %}
        <li>
            {{ tarea.nombre }} -
            {% if tarea.completa %}Completa{% else %}Pendiente{% endif %}
        </li>
    {% empty %}
        <li>No se han encontrado tareas.</li>
    {% endfor %}
</ul>
```

La búsqueda utiliza GET, por lo que los parámetros aparecen en la URL. Por ejemplo:

```text
/tareas/buscar/?nombre=api&estado=pendientes
```

La vista lee los parámetros, aplica los filtros y envía los resultados a la plantilla.

## 14. Actividades propuestas

Implementa consultas para resolver estas tareas:

1. Buscar programadores cuyo apellido contenga un texto.
2. Buscar proyectos cuyo nombre empiece por una palabra.
3. Mostrar los sprints de un proyecto concreto.
4. Mostrar las tareas pendientes de un programador.
5. Buscar tareas que tengan una etiqueta determinada.
6. Buscar tareas iniciadas a partir de una fecha.
7. Obtener tareas completas cuyo nombre contenga `"api"` **o** tareas pendientes asignadas al programador con ID 1.
8. Mostrar las cinco tareas más recientes que tengan fecha de inicio.
9. Contar cuántas tareas tiene un programador.
10. Crear una vista con un formulario GET que permita filtrar por nombre, estado y proyecto.

### Objetivos de aprendizaje

Al terminar la práctica, el alumnado debe saber:

- Obtener registros con `all()` y `filter()`.
- Buscar textos con `icontains`, `startswith` y otros *lookups*.
- Consultar modelos relacionados mediante `__`.
- Comparar fechas y gestionar valores nulos.
- Combinar condiciones con `Q`, `exclude()` y varios filtros.
- Ordenar, limitar y contar resultados.
- Utilizar consultas del ORM desde una vista Django.
