# Relaciones entre tablas en Django

## 1. Introducción

En una base de datos relacional, las tablas pueden estar relacionadas entre sí. Por ejemplo, una empresa puede tener varios departamentos y cada departamento puede tener varios empleados.

Django permite definir estas relaciones mediante su ORM (*Object-Relational Mapping*), utilizando campos especiales en los modelos.

Existen tres tipos principales de relaciones:

| Tipo de relación | Campo Django | Ejemplo |
|---|---|---|
| Uno a muchos | `ForeignKey` | Un equipo tiene varios jugadores. |
| Uno a uno | `OneToOneField` | Un usuario tiene un perfil. |
| Muchos a muchos | `ManyToManyField` | Un alumno cursa varias asignaturas. |

## 2. Relación uno a muchos: ForeignKey

### 2.1. ¿Qué es?

Una relación uno a muchos permite que un registro de una tabla esté relacionado con varios registros de otra tabla.

Por ejemplo:

- Un equipo puede tener muchos jugadores.
- Cada jugador pertenece a un único equipo.

La clave foránea se almacena en la tabla del lado muchos. En este caso, la tabla `Jugador` tendrá un campo que referencia al identificador de `Equipo`.

### 2.2. Crear los modelos

En el archivo `models.py` creamos el modelo `Equipo`:

```python
from django.db import models


class Equipo(models.Model):
    nombre = models.CharField(max_length=50)
    ciudad = models.CharField(max_length=50)

    def __str__(self):
        return self.nombre
```

A continuación, creamos el modelo `Jugador`:

```python
class Jugador(models.Model):
    nombre = models.CharField(max_length=50)
    edad = models.IntegerField()

    equipo = models.ForeignKey(
        Equipo,
        on_delete=models.PROTECT,
        related_name="jugadores"
    )

    def __str__(self):
        return self.nombre
```

El campo `equipo` establece la relación entre ambos modelos.

### 2.3. Parámetros de ForeignKey

**`Equipo`**

Indica el modelo con el que queremos establecer la relación.

**`on_delete`**

Define qué ocurre cuando se intenta eliminar el objeto relacionado.

| Opción | Comportamiento |
|---|---|
| `models.CASCADE` | Elimina también los registros que dependen del objeto eliminado. |
| `models.PROTECT` | Impide eliminar el objeto mientras existan registros relacionados. |
| `models.SET_NULL` | Asigna `NULL` a la relación. Requiere `null=True`. |
| `models.SET_DEFAULT` | Asigna el valor por defecto. Requiere definir `default`. |

**`related_name`**

Permite establecer el nombre utilizado para acceder desde el modelo relacionado.

En nuestro ejemplo, `related_name="jugadores"` permite consultar los jugadores de un equipo mediante `equipo.jugadores.all()`.

### 2.4. Generar las migraciones

Después de crear o modificar los modelos, debemos actualizar la base de datos.

```bash
python manage.py makemigrations
python manage.py migrate
```

- `makemigrations`: genera los archivos de migración a partir de los cambios realizados en los modelos.
- `migrate`: aplica las migraciones a la base de datos.

Django creará una columna llamada `equipo_id` en la tabla de jugadores. Esta columna almacena la clave primaria del equipo al que pertenece cada jugador.

**Importante:** si la tabla ya contiene jugadores, añadir una relación obligatoria puede requerir asignar un valor por defecto durante la migración. Para una práctica inicial, se puede trabajar con una base de datos de prueba vacía o permitir temporalmente valores nulos.

### 2.5. Registrar los modelos en el administrador

En `admin.py`:

```python
from django.contrib import admin
from .models import Equipo, Jugador

admin.site.register(Equipo)
admin.site.register(Jugador)
```

Ahora podemos crear equipos y jugadores desde el panel de administración de Django.

Al crear un jugador, seleccionaremos el equipo al que pertenece.

## 3. Consultar relaciones con Django ORM

### 3.1. Acceder desde el jugador al equipo

Podemos obtener un jugador y consultar los datos de su equipo:

```python
jugador = Jugador.objects.get(nombre="Carlos")

print(jugador.equipo.nombre)
print(jugador.equipo.ciudad)
```

Django permite acceder al objeto relacionado mediante el nombre del campo definido en `ForeignKey`.

### 3.2. Acceder desde el equipo a sus jugadores

Gracias a `related_name="jugadores"`, podemos obtener todos los jugadores de un equipo:

```python
equipo = Equipo.objects.get(nombre="Los Lobos")

for jugador in equipo.jugadores.all():
    print(jugador.nombre)
```

También podemos filtrar directamente los jugadores por el nombre del equipo:

```python
jugadores = Jugador.objects.filter(
    equipo__nombre="Los Lobos"
)
```

El doble guion bajo (`__`) permite consultar campos de modelos relacionados.

## 4. Relación uno a uno: OneToOneField

### 4.1. ¿Qué es?

Una relación uno a uno permite asociar un registro de un modelo con un único registro de otro modelo, y viceversa.

Por ejemplo, cada jugador puede tener una ficha de estadísticas avanzadas.

### 4.2. Crear el modelo

```python
class EstadisticasAvanzadas(models.Model):
    jugador = models.OneToOneField(
        Jugador,
        on_delete=models.CASCADE
    )

    asistencias = models.IntegerField(default=0)
    tarjetas = models.IntegerField(default=0)
```

En este ejemplo:

- Cada ficha pertenece a un único jugador.
- Un jugador no puede tener dos fichas de este modelo.
- Si se elimina el jugador, también se elimina su ficha de estadísticas debido a `CASCADE`.

### 4.3. Consultar la relación

Desde la ficha podemos acceder al jugador:

```python
estadisticas = EstadisticasAvanzadas.objects.first()

print(estadisticas.jugador.nombre)
```

Desde el jugador podemos acceder a su ficha utilizando el nombre inverso predeterminado:

```python
jugador = Jugador.objects.get(nombre="Carlos")

print(jugador.estadisticasavanzadas.asistencias)
```

Si el jugador no tiene ficha, el acceso inverso puede producir una excepción `RelatedObjectDoesNotExist`.

## 5. Relación muchos a muchos: ManyToManyField

### 5.1. ¿Qué es?

Una relación muchos a muchos permite que varios registros de un modelo estén relacionados con varios registros de otro modelo.

Por ejemplo:

- Un jugador puede participar en varios torneos.
- Un torneo puede contar con muchos jugadores.

### 5.2. Crear los modelos

```python
class Torneo(models.Model):
    nombre = models.CharField(max_length=100)

    jugadores = models.ManyToManyField(
        Jugador,
        related_name="torneos"
    )

    def __str__(self):
        return self.nombre
```

Django crea automáticamente una tabla intermedia para almacenar las relaciones entre los torneos y los jugadores.

### 5.3. Consultar los jugadores de un torneo

```python
torneo = Torneo.objects.get(
    nombre="Torneo de invierno"
)

for jugador in torneo.jugadores.all():
    print(jugador.nombre)
```

### 5.4. Consultar los torneos de un jugador

Gracias a `related_name="torneos"`:

```python
jugador = Jugador.objects.get(nombre="Carlos")

for torneo in jugador.torneos.all():
    print(torneo.nombre)
```

### 5.5. Añadir jugadores a un torneo

Podemos establecer relaciones desde el ORM:

```python
torneo = Torneo.objects.get(
    nombre="Torneo de invierno"
)

jugador = Jugador.objects.get(nombre="Carlos")

torneo.jugadores.add(jugador)
```

También podemos eliminar una relación sin eliminar el jugador ni el torneo:

```python
torneo.jugadores.remove(jugador)
```

## 6. Comparación de los tres tipos de relaciones

| Característica | ForeignKey | OneToOneField | ManyToManyField |
|---|---|---|---|
| Tipo | Uno a muchos | Uno a uno | Muchos a muchos |
| Clave foránea | En el modelo que contiene el campo | En el modelo que contiene el campo | Tabla intermedia |
| Ejemplo | Equipo y jugadores | Jugador y ficha | Torneos y jugadores |
| Permite varios relacionados | Sí, en el lado inverso | No | Sí, en ambos lados |

## 7. Buenas prácticas

- Utilizar nombres de modelos en singular: `Jugador`, `Equipo`, `Torneo`.
- Definir `__str__()` para facilitar la identificación de los objetos en el administrador.
- Elegir `on_delete` según las reglas del negocio.
- Utilizar `related_name` cuando queramos controlar cómo se accede a la relación inversa.
- Ejecutar `makemigrations` y `migrate` después de modificar los modelos.


## 8. Comandos para pruebas

```python
from django.core.management.base import BaseCommand
from call.models import Persona


class Command(BaseCommand):
    help = 'Comando de carga'
    def handle(self, *args, **options):
        persona = Persona(nombre="Fran",primer_apellido="Alia",segundo_apellido="Hernandez")
        persona.save()
```

## Ejercicio VUELING

Queremos crear una nueva app llamada vueling para registrar los vuelos de los pasajeros. Para ello vamos a tener un catalogo con 20 ciudades. Son las siguientes:

```text
ID	Ciudad	Aeropuerto
1	Madrid	Adolfo Suárez Madrid-Barajas
2	Barcelona	Josep Tarradellas Barcelona-El Prat
3	Málaga	Málaga-Costa del Sol
4	Sevilla	Sevilla-San Pablo
5	Valencia	Valencia
6	Alicante	Alicante-Elche Miguel Hernández
7	Bilbao	Bilbao
8	Palma de Mallorca	Palma de Mallorca
9	Ibiza	Ibiza
10	Santiago de Compostela	Santiago-Rosalía de Castro
11	París	París-Charles de Gaulle
12	Londres	Heathrow
13	Roma	Roma-Fiumicino
14	Berlín	Berlín-Brandeburgo
15	Ámsterdam	Schiphol
16	Lisboa	Humberto Delgado
17	Bruselas	Bruselas-Zaventem
18	Nueva York	John F. Kennedy (JFK)
19	Tokio	Haneda
20	Dubái	Aeropuerto Internacional de Dubái
``` 

Vamos a tener una lista con 50 pasajeros, en el que se va a indicar el nombre, apellidos, fecha de nacimiento, tipo de identificacion (DNI, NIF) número de documento.

Cuando un pasajero va a realizar un vuelo se registra la ciudad de origen, la ciudad de destino, hora de salida, hora de llegada y la referencia de vuelo.

Establecer 3 vuelos con 10 pasajeros cada uno diferentes. Las referencias de vuelo son : REF001, REF002 y REF003.



