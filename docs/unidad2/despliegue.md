# Práctica: despliegue de una aplicación Django con Docker en un servidor Linux

## 1. Objetivos

En esta práctica vamos a desplegar una aplicación Django en un servidor
Linux del instituto utilizando Docker y Docker Compose.

Al finalizar, el alumnado habrá trabajado con:

-   Git y GitHub para obtener el código fuente.
-   Configuración de Django mediante variables de entorno.
-   Creación de una imagen con un `Dockerfile`.
-   Gestión de contenedores con Docker Compose.
-   Persistencia de datos mediante volúmenes.
-   Acceso remoto a un servidor Linux.
-   Despliegue y comprobación de una aplicación web.

## 2. Datos del entorno

  Elemento                           Valor
  ---------------------------------- --------------------------------------------------
  Proyecto                           ProyectoScrumDjango
  Repositorio                        https://github.com/alion1992/ProyectoScrumDjango
  Servidor                           `192.168.100.8`
  Sistema de contenedores            Docker
  Servidor WSGI                      Gunicorn
  Base de datos                      SQLite
  Ruta del proyecto en el servidor   `/home/alia/ProyectoScrumDjango`

> **Nota sobre el puerto:** durante las pruebas se accedió correctamente
> a la aplicación mediante `http://192.168.100.8:8000/api/bienvenida/`.
> Si se decide asignar el puerto 8001 u otro puerto a cada alumno, hay
> que modificar el mapeo de puertos de su `docker-compose.yml` y
> comprobar que ese puerto está libre.

## 3. Preparación del proyecto en el ordenador

### 3.1. Clonar el repositorio

En una terminal, clonar el proyecto y entrar en su carpeta:

``` bash
git clone https://github.com/alion1992/ProyectoScrumDjango.git
cd ProyectoScrumDjango
```

La estructura del proyecto contiene el archivo `requirements.txt` en la
raíz y el proyecto Django dentro de la carpeta `ejemplo`:

``` text
ProyectoScrumDjango/
├── requirements.txt
└── ejemplo/
    ├── manage.py
    ├── db.sqlite3
    ├── ejemplo/
    │   ├── settings.py
    │   ├── urls.py
    │   ├── wsgi.py
    │   └── asgi.py
    ├── call/
    └── scrum/
```

### 3.2. Adaptar `settings.py`

El archivo que se modifica es `ejemplo/ejemplo/settings.py`.

Se importan los módulos necesarios:

``` python
import os
from pathlib import Path
```

La configuración de seguridad se obtiene de variables de entorno:

``` python
SECRET_KEY = os.environ["DJANGO_SECRET_KEY"]

DEBUG = os.environ.get("DJANGO_DEBUG", "False") == "True"

ALLOWED_HOSTS = os.environ.get(
    "DJANGO_ALLOWED_HOSTS",
    "localhost,127.0.0.1"
).split(",")
```

La base de datos SQLite se configura para poder guardar el fichero en
una ruta indicada desde el entorno:

``` python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": os.environ.get(
            "DJANGO_DB_PATH",
            str(BASE_DIR / "db.sqlite3")
        ),
    }
}
```

Para recoger los archivos estáticos:

``` python
STATIC_URL = "static/"
STATIC_ROOT = BASE_DIR / "staticfiles"
```

La clave secreta no debe guardarse directamente en el repositorio. Se
utilizará un archivo `.env` distinto en cada entorno.

### 3.3. Añadir Gunicorn a las dependencias

En `requirements.txt` se mantienen las dependencias del proyecto y se
añade Gunicorn:

``` text
asgiref==3.12.1
call==0.1
Django==6.1.2
sqlparse==0.6.0
gunicorn
```

Gunicorn será el servidor WSGI que ejecutará Django dentro del
contenedor.


## 4. Crear los archivos de Docker

Los siguientes archivos se crean en la raíz del repositorio, al mismo
nivel que `requirements.txt`.

### 4.1. Dockerfile

Crear un archivo llamado `Dockerfile`:

``` dockerfile
FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY ejemplo/ .

RUN mkdir -p /data

EXPOSE 8000

CMD ["gunicorn", "ejemplo.wsgi:application", "--bind", "0.0.0.0:8000"]
```

Este archivo indica cómo construir la imagen:

1.  Parte de una imagen de Python 3.12.
2.  Establece `/app` como directorio de trabajo.
3.  Copia e instala las dependencias.
4.  Copia el código Django.
5.  Crea el directorio destinado a los datos persistentes.
6.  Arranca la aplicación con Gunicorn en el puerto 8000.

### 4.2. Archivo `.dockerignore`

Crear `.dockerignore` para evitar copiar archivos innecesarios a la
imagen:

``` text
.venv/
__pycache__/
*.pyc
.git/
.idea/
.vscode/
.DS_Store
.env
```

### 4.3. Archivo `docker-compose.yml`

Crear `docker-compose.yml`:

``` yaml
services:
  django:
    build: .
    container_name: proyecto-scrum-django
    restart: unless-stopped

    ports:
      - "8000:8000"

    environment:
      DJANGO_SECRET_KEY: ${DJANGO_SECRET_KEY}
      DJANGO_DEBUG: "False"
      DJANGO_ALLOWED_HOSTS: "192.168.100.8,localhost,127.0.0.1"
      DJANGO_DB_PATH: "/data/db.sqlite3"

    volumes:
      - django_data:/data

volumes:
  django_data:
```

El puerto de la izquierda es el puerto del servidor y el de la derecha
es el puerto del contenedor. Por ejemplo, `8000:8000` publica el
servicio en el puerto 8000 del servidor.

El volumen `django_data` conserva la base de datos SQLite aunque se
elimine y se vuelva a crear el contenedor.

> Si se asigna un puerto distinto a cada alumno, por ejemplo el 8101, se
> cambiaría el mapeo a `8101:8000`.

### 4.4. Archivo `.env`

Crear un archivo llamado exactamente `.env`, sin extensión adicional, en
la raíz del proyecto:

``` dotenv
DJANGO_SECRET_KEY=CLAVE_SECRETA_GENERADA
```

Generar una clave aleatoria. En un equipo con Python moderno:

``` bash
python -c "import secrets; print(secrets.token_urlsafe(50))"
```

También se puede utilizar OpenSSL:

``` bash
openssl rand -base64 50
```

Copiar el resultado en `.env`.

El archivo `.env` debe añadirse a `.gitignore` para evitar subir
secretos a GitHub:

``` text
.env
```

También debe aparecer en `.dockerignore`.

**No subir nunca el archivo `.env` a GitHub.** Cada entorno debe tener
su propia clave secreta.

## 5. Probar Docker en el ordenador

Comprobar que Docker y Docker Compose están disponibles:

``` bash
docker --version
docker compose version
```

Comprobar la configuración:

``` bash
docker compose config
```

Construir la imagen:

``` bash
docker compose build
```

Arrancar el contenedor:

``` bash
docker compose up -d
```

Comprobar el estado:

``` bash
docker compose ps
```

Consultar los registros:

``` bash
docker compose logs
```

Ejecutar las migraciones de Django:

``` bash
docker compose exec django python manage.py migrate
```

Crear un usuario administrador:

``` bash
docker compose exec django python manage.py createsuperuser
```

Comprobar la aplicación desde el navegador:

``` text
http://localhost:8000/api/bienvenida/
```

En la práctica realizada, la ruta `/api/bienvenida/` mostró el mensaje
de bienvenida de la aplicación, confirmando que Django y Gunicorn
estaban funcionando dentro del contenedor.



## 7. Comprobar el despliegue desde otro ordenador

Desde un equipo conectado a la red del instituto, abrir:

``` text
http://192.168.100.8:8000/api/bienvenida/
```

En la práctica se obtuvo correctamente el mensaje:

**«Hola chicos e Inmaculada estamos en nuestro primer controlador»**

Esto confirmó que la aplicación Django estaba accesible desde otro
equipo de la red a través del servidor del instituto.



## 9. Comandos Docker útiles 

  -------------------------------------------------------------------------------------------
  Comando                                                 Función
  ------------------------------------------------------- -----------------------------------
  `docker --version`                                      Consultar la versión de Docker

  `docker ps`                                             Ver contenedores en ejecución

  `docker ps -a`                                          Ver todos los contenedores

  `docker images`                                         Listar imágenes

  `docker-compose build`                                  Construir la imagen del proyecto

  `docker-compose up -d`                                  Arrancar los servicios en segundo
                                                          plano

  `docker-compose down`                                   Detener y eliminar los contenedores
                                                          y redes del proyecto

  `docker-compose logs`                                   Consultar los registros

  `docker-compose exec django python manage.py migrate`   Ejecutar migraciones dentro del
                                                          contenedor
  -------------------------------------------------------------------------------------------

> Atención: `docker-compose down -v` también elimina los volúmenes
> declarados por el proyecto. No debe ejecutarse si se quieren conservar
> los datos.

## 10. Flujo de actualización del proyecto

Cuando se realicen cambios en el código y se suban a GitHub, el
despliegue se puede actualizar desde el servidor:

``` bash
cd /home/alia/ProyectoScrumDjango

git pull

docker-compose build

docker-compose up -d
```

Si cambian los modelos de Django, ejecutar también:

``` bash
docker-compose exec django python manage.py migrate
```

## 11. Resultado final

Se ha conseguido desplegar una aplicación Django en un servidor Linux
del instituto utilizando Docker, Docker Compose, Gunicorn y SQLite.

La aplicación ha quedado accesible desde la red local del centro y se
han creado cuentas individuales para que 12 alumnos puedan conectarse
por SSH y trabajar con Docker.

