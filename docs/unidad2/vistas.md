# Vistas en Django: GET, POST, PUT, PATCH y DELETE

En esta clase vamos a crear vistas basadas en funciones para trabajar con los modelos `Programador`, `Proyecto`, `Sprint`, `Etiqueta` y `Tarea` de nuestra aplicación Scrum. Utilizaremos el ORM de Django, JSON, `JsonResponse` y los códigos HTTP habituales.

> Los ejemplos son didácticos. En una aplicación real también hay que configurar autenticación, permisos y validación completa de los datos.

## 1. ¿Qué es una vista?

Una vista es una función de Python que recibe una petición HTTP (`request`) y devuelve una respuesta HTTP.

```python
from django.http import HttpResponse

def inicio(request):
    return HttpResponse("Bienvenido a mi aplicación Django")
```

- `request` contiene información de la petición.
- `HttpResponse` devuelve texto.
- `JsonResponse` permite devolver datos en formato JSON.

## 2. Configurar las URL

En `scrum/urls.py` definiremos las rutas de la aplicación:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("hola/", views.inicio, name="inicio"),
    path("tareas/", views.listar_tareas, name="tareas"),
    path("tarea/<int:id>/", views.listar_tarea_concreta, name="tarea_concreta"),
    path("tareas/crear/", views.crear_tarea, name="crear_tarea"),
    path("tareas/<int:id>/actualizar/", views.actualizar_tarea, name="actualizar_tarea"),
    path("tareas/<int:id>/parcial/", views.actualizar_tarea_parcial, name="actualizar_tarea_parcial"),
    path("tareas/<int:id>/eliminar/", views.eliminar_tarea, name="eliminar_tarea"),
]
```

En el `urls.py` principal del proyecto:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("scrum.urls")),
]
```

Como el proyecto incluye `scrum.urls` bajo el prefijo `api/`, la ruta `tareas/` se convierte en `/api/tareas/`.

## 3. Métodos HTTP

| Método | Uso habitual |
|---|---|
| `GET` | Consultar información |
| `POST` | Crear un recurso |
| `PUT` | Actualizar el recurso completo |
| `PATCH` | Actualizar parte de un recurso |
| `DELETE` | Eliminar un recurso |

## 4. Consultar todas las tareas: GET

`Tarea.objects.all()` devuelve objetos `Tarea`. Para una consulta sencilla que podamos convertir directamente a JSON, podemos utilizar `values()`, que devuelve los campos como diccionarios.

```python
from django.http import JsonResponse
from .models import Tarea


def listar_tareas(request):
    if request.method != "GET":
        return JsonResponse(
            {"error": "Operación no permitida"},
            status=405
        )

    tareas = Tarea.objects.values()
    return JsonResponse(list(tareas), safe=False)
```

- `values()` obtiene los campos de las tareas como diccionarios.
- `list()` convierte el `QuerySet` en una lista.
- `safe=False` permite que `JsonResponse` devuelva una lista en lugar de un diccionario.
- No hace falta escribir `.all().values()`: `values()` sin filtros ya consulta todos los registros.

La respuesta será una lista similar a esta:

```json
[
    {
        "id": 1,
        "nombre": "Preparar el login",
        "fecha_inicio": "2026-10-28",
        "fecha_fin": null,
        "completa": false,
        "sprint_id": 1,
        "programador_id": 2
    }
]
```

Los valores concretos dependerán de la base de datos.

## 5. Consultar una tarea por su ID: GET

Para obtener una sola tarea utilizaremos `filter()`, `values()` y `first()`.

```python
def listar_tarea_concreta(request, id):
    if request.method != "GET":
        return JsonResponse(
            {"error": "Operación no permitida"},
            status=405
        )

    tarea = Tarea.objects.filter(id=id).values().first()

    if tarea is None:
        return JsonResponse(
            {"error": "Tarea no encontrada"},
            status=404
        )

    return JsonResponse(tarea)
```

- `filter(id=id)` busca por el identificador recibido en la URL.
- `values()` convierte los campos en diccionarios.
- `first()` devuelve un único diccionario o `None` si no hay resultados.
- `JsonResponse(tarea)` devuelve directamente el diccionario.

**¿Por qué aquí no usamos `list()` ni `safe=False`?** Porque devolvemos un único diccionario. Para varias tareas devolvemos una lista; para una tarea, un diccionario.

Ejemplo de URL: `/api/tarea/45/`.

## 6. Crear una tarea: POST y JSON

Para crear una tarea enviaremos un JSON en el cuerpo de la petición. En este ejemplo usamos `@csrf_exempt` para simplificar las pruebas de una API independiente con Postman. Más adelante hay que configurar correctamente la seguridad.

```python
import json
from json import JSONDecodeError

from django.http import JsonResponse
from django.views.decorators.csrf import csrf_exempt
from .models import Tarea, Sprint, Programador


@csrf_exempt
def crear_tarea(request):
    if request.method != "POST":
        return JsonResponse(
            {"error": "Operación no permitida"},
            status=405
        )

    try:
        datos = json.loads(request.body)
    except (JSONDecodeError, UnicodeDecodeError):
        return JsonResponse(
            {"error": "JSON inválido"},
            status=400
        )

    if not isinstance(datos, dict):
        return JsonResponse(
            {"error": "Se esperaba un objeto JSON"},
            status=400
        )

    nombre = datos.get("nombre")
    if not isinstance(nombre, str) or not nombre.strip():
        return JsonResponse(
            {"error": "El nombre es obligatorio"},
            status=400
        )

    sprint_id = datos.get("sprint_id")
    if sprint_id is not None and not Sprint.objects.filter(id=sprint_id).exists():
        return JsonResponse(
            {"error": "El sprint no existe"},
            status=400
        )

    programador_id = datos.get("programador_id")
    if programador_id is not None and not Programador.objects.filter(id=programador_id).exists():
        return JsonResponse(
            {"error": "El programador no existe"},
            status=400
        )

    nueva_tarea = Tarea.objects.create(
        nombre=nombre.strip(),
        fecha_inicio=datos.get("fecha_inicio"),
        fecha_fin=datos.get("fecha_fin"),
        completa=datos.get("completa", False),
        sprint_id=sprint_id,
        programador_id=programador_id
    )

    return JsonResponse(
        {
            "mensaje": "Tarea creada correctamente",
            "id": nueva_tarea.id
        },
        status=201
    )
```

### Entender el POST

1. Comprobamos que el método sea `POST`.
2. `json.loads(request.body)` convierte el cuerpo JSON en un objeto Python.
3. Controlamos los errores de JSON y comprobamos que recibimos un diccionario.
4. Validamos el nombre.
5. Comprobamos que el sprint y el programador existan si se han indicado.
6. `Tarea.objects.create(...)` crea y guarda la tarea automáticamente; no hace falta llamar después a `save()`.

### JSON para probar el POST

En Postman selecciona `POST` → `Body` → `raw` → `JSON`.

```json
{
    "nombre": "Desarrollar el login",
    "fecha_inicio": "2026-10-28",
    "fecha_fin": null,
    "completa": false,
    "sprint_id": 1,
    "programador_id": 2
}
```

Los identificadores deben existir. Como sprint y programador son opcionales en el modelo, puedes omitirlos o enviar `null`.

### ¿Se pueden mapear automáticamente los campos?

Python permite desempaquetar un diccionario con `**`:

```python
Tarea.objects.create(**datos)
```

Pero no conviene pasar directamente todo el JSON recibido a `create()`: el cliente podría enviar campos que no queremos aceptar y las relaciones requieren un tratamiento adecuado. Es preferible validar y seleccionar los campos permitidos.

## 7. Actualizar una tarea: PUT

Para actualizar una tarea existente, la buscamos por su ID, modificamos sus atributos y llamamos a `save()`. En este ejemplo, PUT exige el nombre y espera que se envíe el estado de todos los campos editables, incluidos los opcionales que se quieran conservar.

```python
@csrf_exempt
def actualizar_tarea(request, id):
    if request.method != "PUT":
        return JsonResponse(
            {"error": "Operación no permitida"},
            status=405
        )

    try:
        datos = json.loads(request.body)
    except (JSONDecodeError, UnicodeDecodeError):
        return JsonResponse(
            {"error": "JSON inválido"},
            status=400
        )

    if not isinstance(datos, dict):
        return JsonResponse(
            {"error": "Se esperaba un objeto JSON"},
            status=400
        )

    tarea = Tarea.objects.filter(id=id).first()
    if tarea is None:
        return JsonResponse(
            {"error": "Tarea no encontrada"},
            status=404
        )

    nombre = datos.get("nombre")
    if not isinstance(nombre, str) or not nombre.strip():
        return JsonResponse(
            {"error": "El nombre es obligatorio"},
            status=400
        )

    sprint_id = datos.get("sprint_id")
    if sprint_id is not None and not Sprint.objects.filter(id=sprint_id).exists():
        return JsonResponse(
            {"error": "El sprint no existe"},
            status=400
        )

    programador_id = datos.get("programador_id")
    if programador_id is not None and not Programador.objects.filter(id=programador_id).exists():
        return JsonResponse(
            {"error": "El programador no existe"},
            status=400
        )

    tarea.nombre = nombre.strip()
    tarea.fecha_inicio = datos.get("fecha_inicio")
    tarea.fecha_fin = datos.get("fecha_fin")
    tarea.completa = datos.get("completa", False)
    tarea.sprint_id = sprint_id
    tarea.programador_id = programador_id
    tarea.save()

    return JsonResponse({
        "mensaje": "Tarea actualizada correctamente",
        "id": tarea.id
    })
```

Ejemplo: `PUT /api/tareas/45/actualizar/` con este JSON:

```json
{
    "nombre": "Desarrollar el login",
    "fecha_inicio": "2026-10-28",
    "fecha_fin": null,
    "completa": true,
    "sprint_id": 1,
    "programador_id": 2
}
```

### Diferencia entre `create()` y `save()`

Crear un objeto nuevo:

```python
nueva_tarea = Tarea.objects.create(nombre="Preparar el login")
```

Modificar un objeto que ya existe:

```python
tarea = Tarea.objects.get(id=45)
tarea.nombre = "Nuevo nombre"
tarea.save()
```

`create()` crea y guarda; `save()` persiste los cambios del objeto existente.

## 8. Actualizar solo algunos campos: PATCH

PATCH permite modificar únicamente los campos recibidos. En este ejemplo permitimos cambiar `nombre` y `completa`.

```python
@csrf_exempt
def actualizar_tarea_parcial(request, id):
    if request.method != "PATCH":
        return JsonResponse(
            {"error": "Operación no permitida"},
            status=405
        )

    try:
        datos = json.loads(request.body)
    except (JSONDecodeError, UnicodeDecodeError):
        return JsonResponse(
            {"error": "JSON inválido"},
            status=400
        )

    if not isinstance(datos, dict):
        return JsonResponse(
            {"error": "Se esperaba un objeto JSON"},
            status=400
        )

    tarea = Tarea.objects.filter(id=id).first()
    if tarea is None:
        return JsonResponse(
            {"error": "Tarea no encontrada"},
            status=404
        )

    if "nombre" in datos:
        nombre = datos["nombre"]
        if not isinstance(nombre, str) or not nombre.strip():
            return JsonResponse(
                {"error": "El nombre debe ser un texto no vacío"},
                status=400
            )
        tarea.nombre = nombre.strip()

    if "completa" in datos:
        completa = datos["completa"]
        if not isinstance(completa, bool):
            return JsonResponse(
                {"error": "El campo completa debe ser true o false"},
                status=400
            )
        tarea.completa = completa

    tarea.save()

    return JsonResponse({
        "mensaje": "Tarea actualizada correctamente",
        "id": tarea.id,
        "nombre": tarea.nombre,
        "completa": tarea.completa
    })
```

Para marcar la tarea 45 como completada, envía `PATCH /api/tareas/45/parcial/` con:

```json
{
    "completa": true
}
```

Solo cambia `completa`; el nombre y los demás campos se mantienen.

## 9. Eliminar una tarea: DELETE

```python
from django.http import HttpResponse


@csrf_exempt
def eliminar_tarea(request, id):
    if request.method != "DELETE":
        return JsonResponse(
            {"error": "Operación no permitida"},
            status=405
        )

    tarea = Tarea.objects.filter(id=id).first()
    if tarea is None:
        return JsonResponse(
            {"error": "Tarea no encontrada"},
            status=404
        )

    tarea.delete()
    return HttpResponse(status=204)
```

`204 No Content` indica que la operación se ha completado y no se devuelve contenido. Atención: en el modelo, las relaciones de `Tarea` con `Sprint` y `Programador` usan `on_delete=models.CASCADE`; eliminar un sprint o programador puede eliminar también las tareas relacionadas.

## 10. Resumen de las URL

Con las rutas definidas anteriormente:

| Operación | Método | URL |
|---|---|---|
| Listar tareas | GET | `/api/tareas/` |
| Consultar tarea 45 | GET | `/api/tarea/45/` |
| Crear tarea | POST | `/api/tareas/crear/` |
| Actualizar tarea completa | PUT | `/api/tareas/45/actualizar/` |
| Actualizar algunos campos | PATCH | `/api/tareas/45/parcial/` |
| Eliminar tarea | DELETE | `/api/tareas/45/eliminar/` |

Ejemplo de consulta con `curl`:

```bash
curl http://127.0.0.1:8000/api/tareas/
```

Ejemplo de creación:

```bash
curl -X POST http://127.0.0.1:8000/api/tareas/crear/ \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Preparar presentación"}'
```

Ejemplo de actualización parcial:

```bash
curl -X PATCH http://127.0.0.1:8000/api/tareas/45/parcial/ \
  -H "Content-Type: application/json" \
  -d '{"completa":true}'
```

Ejemplo de eliminación:

```bash
curl -X DELETE http://127.0.0.1:8000/api/tareas/45/eliminar/
```

## 11. Códigos HTTP habituales

| Código | Significado | Cuándo utilizarlo |
|---:|---|---|
| `200` | OK | Consulta o actualización correcta con respuesta |
| `201` | Created | Se ha creado una tarea |
| `204` | No Content | Eliminación correcta sin cuerpo de respuesta |
| `400` | Bad Request | JSON inválido o datos incorrectos |
| `403` | Forbidden | El usuario no tiene permisos |
| `404` | Not Found | La tarea, el sprint o el programador no existe |
| `405` | Method Not Allowed | Se utiliza un método HTTP no permitido |
| `500` | Internal Server Error | Error inesperado en el servidor |

Por ejemplo:

```python
return JsonResponse(
    {"error": "Tarea no encontrada"},
    status=404
)
```

El código HTTP comunica el resultado de la petición y el JSON explica el motivo.

## 12. ¿Qué significa `safe` en `JsonResponse`?

Por defecto, `JsonResponse` espera un diccionario:

```python
return JsonResponse({"nombre": "Preparar presentación"})
```

Para devolver una lista de diccionarios usamos `safe=False`:

```python
return JsonResponse(list(Tarea.objects.values()), safe=False)
```

`safe=False` permite devolver una lista como respuesta principal. No es necesario cuando devolvemos un único diccionario.

## 13. ¿Qué hace `@csrf_exempt`?

`@csrf_exempt` desactiva la comprobación CSRF en esa vista. En estos ejemplos se utiliza para simplificar las pruebas con herramientas como Postman. No autentica al usuario ni comprueba sus permisos. No debe añadirse indiscriminadamente en una aplicación real; si se utiliza autenticación mediante sesiones y cookies, la protección CSRF sigue siendo importante.

## 14. Buenas prácticas

- Comprobar qué método HTTP admite cada vista.
- Controlar los errores de `json.loads()`.
- Comprobar que el JSON recibido es un diccionario cuando esperamos un objeto.
- Validar los campos antes de guardar.
- Comprobar que los identificadores de las relaciones existen.
- Utilizar `create()` para crear objetos y `save()` para guardar modificaciones.
- Devolver `404` cuando no existe el recurso solicitado.
- No confiar en los datos recibidos del cliente.
- Configurar autenticación y permisos en una API real.
- Mantener la protección CSRF cuando se trabaja con sesiones y cookies.

