Crear el fichero llamando notas.txt

```
Ana;Programación;8.5
Luis;Programación;4.2
Marta;Programación;7.3
Pedro;Programación;3.8
Lucía;Programación;9.1
Carlos;Programación;5.6
```

Realiza un programa Python que lea el fichero y construya automáticamente un diccionario con esta estructura:

```
alumnos = {
    "Ana": {
        "modulo": "Programación",
        "nota": 8.5
    },
    "Luis": {
        "modulo": "Programación",
        "nota": 4.2
    }
}
```

Ejercicio 1:
Mostrar los alumnos

Recorre el diccionario y muestra:
```
Ana - Programación - 8.5
Luis - Programación - 4.2
...
```

Ejercicio 2:
Muestra dos listados:

```
APROBADOS
Ana: 8.5
Marta: 7.3
...

SUSPENSOS
Luis: 4.2
Pedro: 3.8
```

Ejercicio 3:
Calcular la nota media

Calcula la nota media de todos los alumnos utilizando los datos almacenados en el diccionario.

Ejercicio4: 
Generar un nuevo fichero
Crea automáticamente aprobados.txt que contenga únicamente los alumnos aprobados:

```
Ana;8.5
Marta;7.3
Lucía;9.1
Carlos;5.6
```