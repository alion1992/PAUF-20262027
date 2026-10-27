# Vistas en Django: GET, POST, PUT, PATCH y DELETE

En esta clase aprenderemos a crear vistas Django que reciban peticiones HTTP, consulten los modelos y permitan listar, crear, actualizar y eliminar registros.

Trabajaremos con los modelos `Programador`, `Proyecto`, `Sprint`, `Etiqueta` y `Tarea` de la práctica.

## 1. ¿Qué es una vista?

Una vista es una función o una clase de Python que recibe una petición HTTP y devuelve una respuesta HTTP.

El flujo habitual de una petición es:

1. El cliente solicita una URL.
2. Django busca la URL en `urls.py`.
3. La URL dirige la petición a una vista.
4. La vista consulta o modifica los modelos si es necesario.
5. La vista devuelve una respuesta.

Las respuestas pueden ser HTML, JSON, una redirección o un error HTTP.

## 2. Crear una primera vista

En el archivo `views.py` de la aplicación:

```python
from django.http import HttpResponse


def inicio(request):
    return HttpResponse("Bienvenido a mi aplicación Django")
```

- `request` contiene la información de la petición.
- `HttpResponse` devuelve una respuesta HTTP.
- La vista debe devolver una respuesta al cliente.

### Asociar una URL a la vista

En el `urls.py` de la aplicación:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.inicio, name="inicio"),
]
```

Si la aplicación se llama `tareas`, en el `urls.py` principal del proyecto se puede incluir así:

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("tareas.urls")),
]
```

Cambia `tareas` por el nombre real de tu aplicación.

## 3. Los métodos HTTP

Los métodos HTTP indican qué operación quiere realizar el cliente.

| Método | Uso habitual |
|---|---|
| `GET` | Consultar información |
| `POST` | Enviar datos; se usa a menudo para crear recursos |
| `PUT` | Sustituir o actualizar un recurso completo en una API |
| `PATCH` | Modificar parcialmente un recurso en una API |
| `DELETE` | Eliminar un recurso en una API |

En una aplicación web tradicional, los formularios HTML utilizan normalmente GET o POST. Los formularios HTML no admiten directamente PUT, PATCH ni DELETE como valores de `method`; estos métodos suelen utilizarse en APIs o mediante mecanismos adicionales.

## 4. Consultar datos con GET

`GET` se utiliza para obtener información. Para devolver datos en JSON podemos utilizar `JsonResponse`.

### 4.1. Listar todas las tareas

```python
from django.http import JsonResponse
from .models import Tarea


def listar_tareas(request):
    if request.method != "GET":
        return JsonResponse(
            {"error": "Método no permitido"},
            status=405
        )

    tareas = Tarea.objects.all()

    datos = [
        {
            "id": tarea.id,
            "nombre": tarea.nombre,
            "fecha_inicio": (
                tarea.fecha_inicio.isoformat()
                if tarea.fecha_inicio else None
            ),
            "fecha_fin": (
                tarea.fecha_fin.isoformat()
                if tarea.fecha_fin else None
            ),
            "completa": tarea.completa,
            "sprint_id": tarea.sprint_id,
            "programador_id": tarea.programador_id,
        }
        for tarea in tareas
    ]

    return JsonResponse(datos, safe=False)
```

`JsonResponse` espera por defecto un diccionario. Utilizamos `safe=False` para devolver una lista como respuesta principal.

### 4.2. Obtener una tarea por su ID

```python
from django.shortcuts import get_object_or_404
from django.http import JsonResponse
from .models import Tarea


def detalle_tarea(request, id):
    if request.method != "GET":
        return JsonResponse(
            {"error": "Método no permitido"},
            status=405
        )

    tarea = get_object_or_404(Tarea, pk=id)

    datos = {
        "id": tarea.id,
        "nombre": tarea.nombre,
        "completa": tarea.completa,
        "programador_id": tarea.programador_id,
        "sprint_id": tarea.sprint_id,
        "etiquetas": list(
            tarea.etiquetas.values_list("id", flat=True)
        ),
    }

    return JsonResponse(datos)
```

`get_object_or_404()` devuelve el objeto si existe o una respuesta HTTP 404 si no se encuentra.

## 5. Crear registros con POST

Para crear una tarea recibiremos los datos mediante POST. En este ejemplo, los datos llegan desde un formulario HTML.

```python
from django.views.decorators.http import require_POST
from django.http import JsonResponse
from .models import Tarea


@require_POST
def crear_tarea(request):
    nombre = request.POST.get("nombre", "").strip()

    if not nombre:
        return JsonResponse(
            {"error": "El nombre es obligatorio"},
            status=400
        )

    tarea = Tarea.objects.create(nombre=nombre)

    return JsonResponse(
        {
            "mensaje": "Tarea creada",
            "id": tarea.id,
            "nombre": tarea.nombre,
        },
        status=201
    )
```

Aspectos importantes:

- `request.POST.get()` obtiene un campo enviado mediante un formulario POST.
- `strip()` elimina los espacios al principio y al final.
- `objects.create()` crea y guarda el registro.
- El código HTTP `201 Created` indica que se ha creado un recurso.
- `@require_POST` limita la vista al método POST.

El modelo permite que otros campos de la tarea queden vacíos o usen sus valores por defecto. Si se quieren aceptar más campos, hay que validarlos antes de guardarlos.

### Formulario HTML de ejemplo

```html
<form method="post" action="/api/tareas/crear/">
    {% csrf_token %}

    <label for="nombre">Nombre de la tarea:</label>
    <input type="text" id="nombre" name="nombre" required>

    <button type="submit">Crear tarea</button>
</form>
```

En una aplicación web con formularios Django, el token CSRF ayuda a proteger las operaciones que modifican datos. No desactives esta protección sin una razón justificada.

## 6. Leer datos JSON en una petición POST

Cuando desarrollamos una API que recibe JSON, `request.POST` no es el mecanismo adecuado para leer el cuerpo JSON. Podemos utilizar `json.loads(request.body)`.

```python
import json

from django.views.decorators.http import require_POST
from django.http import JsonResponse
from django.core.exceptions import ValidationError
from .models import Tarea


@require_POST
def crear_tarea_json(request):
    try:
        datos = json.loads(request.body)
    except (json.JSONDecodeError, UnicodeDecodeError):
        return JsonResponse(
            {"error": "El cuerpo debe contener JSON válido"},
            status=400
        )

    if not isinstance(datos, dict):
        return JsonResponse(
            {"error": "Se esperaba un objeto JSON"},
            status=400
        )

    nombre = datos.get("nombre", "")
    if not isinstance(nombre, str) or not nombre.strip():
        return JsonResponse(
            {"error": "El nombre es obligatorio"},
            status=400
        )

    tarea = Tarea.objects.create(nombre=nombre.strip())

    return JsonResponse(
        {"id": tarea.id, "nombre": tarea.nombre},
        status=201
    )
```

Una petición de ejemplo podría enviar este JSON:

```json
{
  "nombre": "Preparar presentación"
}
```

En una API real, valida también los tipos y los valores de todos los campos recibidos, las relaciones y los permisos del usuario.

## 7. Actualizar un registro con PUT o PATCH

PUT suele representar la actualización completa de un recurso y PATCH una actualización parcial. Django no impone una implementación única: debemos definir el comportamiento en nuestra vista.

El siguiente ejemplo permite modificar parcialmente el nombre y el estado de una tarea mediante PATCH.

```python
import json

from django.views.decorators.http import require_http_methods
from django.shortcuts import get_object_or_404
from django.http import JsonResponse
from .models import Tarea


@require_http_methods(["PATCH"])
def actualizar_tarea(request, id):
    tarea = get_object_or_404(Tarea, pk=id)

    try:
        datos = json.loads(request.body)
    except (json.JSONDecodeError, UnicodeDecodeError):
        return JsonResponse(
            {"error": "JSON no válido"},
            status=400
        )

    if not isinstance(datos, dict):
        return JsonResponse(
            {"error": "Se esperaba un objeto JSON"},
            status=400
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
        "mensaje": "Tarea actualizada",
        "id": tarea.id,
        "nombre": tarea.nombre,
        "completa": tarea.completa,
    })
```

Ejemplo de cuerpo JSON para PATCH:

```json
{
  "completa": true
}
```

Solo se modifica el campo `completa`; los demás mantienen su valor.

Para implementar PUT, podríamos crear otra vista que exigiera todos los campos requeridos y sustituyera el estado editable completo del recurso. No conviene llamar PUT a una operación parcial sin explicar esa decisión.

## 8. Eliminar un registro con DELETE

Una API puede utilizar DELETE para eliminar una tarea.

```python
from django.views.decorators.http import require_http_methods
from django.shortcuts import get_object_or_404
from django.http import HttpResponse
from .models import Tarea


@require_http_methods(["DELETE"])
def eliminar_tarea(request, id):
    tarea = get_object_or_404(Tarea, pk=id)
    tarea.delete()

    return HttpResponse(status=204)
```

El código `204 No Content` indica que la operación se ha completado y no hay contenido que devolver.

**Atención:** en el modelo proporcionado, las relaciones con `Programador` y `Sprint` utilizan `on_delete=models.CASCADE`. Eliminar uno de esos objetos puede eliminar también las tareas relacionadas. Antes de borrar registros relacionados, hay que tener en cuenta este comportamiento.

## 9. Definir las URL de las vistas

En `urls.py` de la aplicación:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("api/tareas/", views.listar_tareas, name="listar_tareas"),
    path("api/tareas/<int:id>/", views.detalle_tarea, name="detalle_tarea"),
    path("api/tareas/crear/", views.crear_tarea_json, name="crear_tarea"),
    path("api/tareas/<int:id>/actualizar/", views.actualizar_tarea, name="actualizar_tarea"),
    path("api/tareas/<int:id>/eliminar/", views.eliminar_tarea, name="eliminar_tarea"),
]
```

**Nota sobre el orden de las rutas:** la ruta `api/tareas/<int:id>/` no coincide con `api/tareas/crear/`, porque `crear` no es un entero. Las rutas específicas pueden colocarse antes de las rutas con parámetros para facilitar la lectura.

Las rutas del ejemplo usan nombres ilustrativos. Si eliges la ruta REST habitual `api/tareas/<int:id>/` para el detalle, actualización y eliminación, puedes asociar la misma URL a una vista que distinga entre GET, PATCH y DELETE según `request.method`.

## 10. Probar las peticiones

Puedes probar las vistas con herramientas como Postman, Insomnia o `curl`.

| Operación | Método | URL de ejemplo |
|---|---|---|
| Listar tareas | GET | `/api/tareas/` |
| Consultar tarea 1 | GET | `/api/tareas/1/` |
| Crear tarea | POST | `/api/tareas/crear/` |
| Actualizar parcialmente tarea 1 | PATCH | `/api/tareas/1/actualizar/` |
| Eliminar tarea 1 | DELETE | `/api/tareas/1/eliminar/` |

Ejemplo de consulta GET:

```bash
curl http://127.0.0.1:8000/api/tareas/
```

Ejemplo de creación JSON:

```bash
curl -X POST http://127.0.0.1:8000/api/tareas/crear/ \
  -H "Content-Type: application/json" \
  -d '{"nombre":"Preparar presentación"}'
```

Ejemplo de actualización parcial:

```bash
curl -X PATCH http://127.0.0.1:8000/api/tareas/1/actualizar/ \
  -H "Content-Type: application/json" \
  -d '{"completa":true}'
```

Ejemplo de eliminación:

```bash
curl -X DELETE http://127.0.0.1:8000/api/tareas/1/eliminar/
```

En estos ejemplos se presupone que el servidor está ejecutándose en el puerto 8000 y que las rutas coinciden con las definidas en el proyecto.

## 11. Códigos HTTP habituales

| Código | Significado | Cuándo utilizarlo |
|---|---|---|
| `200 OK` | Petición correcta | Consulta o actualización con respuesta |
| `201 Created` | Recurso creado | Creación correcta |
| `204 No Content` | Correcto, sin contenido | Eliminación correcta |
| `400 Bad Request` | Petición incorrecta | JSON inválido o datos no válidos |
| `404 Not Found` | Recurso no encontrado | No existe la tarea solicitada |
| `405 Method Not Allowed` | Método no permitido | Se usa un método no admitido por la vista |

## 12. Buenas prácticas

- Limitar los métodos permitidos mediante `require_POST`, `require_GET` o `require_http_methods`.
- Validar todos los datos antes de guardarlos.
- Gestionar errores de JSON mal formado.
- Utilizar `get_object_or_404()` cuando el recurso solicitado puede no existir.
- Proteger las operaciones de escritura y comprobar los permisos del usuario.
- Mantener la protección CSRF en formularios web.
- No asumir que recibir un ID significa que el usuario tiene permiso para modificar ese objeto.
- Recordar que `request.POST` sirve para datos de formularios, no para decodificar automáticamente un cuerpo JSON.
- Evitar borrar objetos relacionados sin comprobar las consecuencias de `on_delete`.

## 13. Actividades propuestas

Implementa y prueba estas operaciones usando los modelos de la práctica:

1. Crear un programador mediante POST.
2. Listar todos los proyectos mediante GET.
3. Consultar un sprint por su ID.
4. Crear una tarea indicando nombre y estado.
5. Actualizar el nombre de una tarea mediante PATCH.
6. Marcar una tarea como completa mediante PATCH.
7. Eliminar una etiqueta mediante DELETE.
8. Devolver un error 404 al consultar un objeto inexistente.
9. Devolver un error 400 cuando el JSON esté mal formado.
10. Probar qué ocurre si se intenta utilizar un método HTTP no permitido.

### Objetivo final

El alumnado debe saber crear vistas que respondan a diferentes métodos HTTP, trabajar con los modelos del ORM y devolver respuestas apropiadas. Esta clase utiliza vistas basadas en funciones; posteriormente se pueden estudiar las vistas basadas en clases y Django REST Framework para construir APIs más completas.
