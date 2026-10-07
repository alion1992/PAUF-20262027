# TUTORIAL INICIAR UN PROYECTO CON DJANGO 


## Iniciar proyecto

```bash
python -m django startproject introduccion
```

Con la anterior instrucción lo que estamos realizando es un nuevo proyecto en el que nos generara el código de un esqueleto básico de la creación de un proyecto.

![alt text](image-7.png)

![alt text](image-8.png)


Podríamos decir que el archivo  setting.py es donde están todas las configuraciones del proyecto. Por ahora no tenemos una app para el modelo. El siguiente paso seria crearla.

```bash
python manage.py startapp pruebadb
```

![alt text](image-9.png)

```bash
python manage.py runserver
```

Con la anterior instrucción desplegamos nuestro servidor local para realizar nuestras pruebas (nos vamos a centrar en el back por lo que vistas y demás lo dejamos para más adelante).

![alt text](image-10.png)

![alt text](image-11.png)

```bash
python manage.py migrate
```

Se generar las tablas por defecto que tiene django. 

![alt text](image-12.png)

Hacer migraciones del modelo.
Para realizar las migraciones del modelo debe estar en el archivo setting del proyecto.


![alt text](image-13.png)

![alt text](image-14.png)

![alt text](image-15.png)

![alt text](image-16.png)

```bash
python manage.py createsuperuser
```

![alt text](image-17.png)

Si vamos al archivo urls.py vemos que tenemos /admin (accedemos a ella).

![alt text](image-18.png)

Si queremos que nuestros modelos puedan ser administrados debemos añadirlos al archivo de configuración.

![alt text](image-19.png)

Podemos rellenar la tabla desde la administración

![alt text](image-20.png)

Si queremos que se muestre algo más que una simple referencia a memoria podemos sobreescribir el método __str__ (el toString de siempre)

[Variables en models](https://docs.djangoproject.com/en/5.1/ref/models/fields/)

[Proyecto de ejemplo](https://github.com/alion1992/djangoIntroduccion)