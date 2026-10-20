# Copias de seguridad en Linux: rsync y tar.gz

Boceto de la actividad. Sin respuestas desarrolladas, pruebas ejecutadas ni resultados registrados todavía.

[Volver al índice de actividades](../README.md)

## Objetivo

Realizar copias de seguridad en Linux con rsync y tar.gz, restaurar su contenido y automatizar las copias remotas mediante scripts y crontab.

## Índice

1. [Informe técnico](#informe-técnico)
2. [Preparación del entorno](#preparación-del-entorno)
3. [Desarrollo práctico](#desarrollo-práctico)
4. [Comparación y conclusiones](#comparación-y-conclusiones)
5. [Fuentes](#fuentes)

## Informe técnico

### Copias de seguridad

1. ¿Qué son las copias de seguridad y por qué debemos hacerlas?
2. ¿Qué criterios de clasificación de la información permiten determinar los datos que se deben salvaguardar?
3. ¿De qué depende la duración de una copia de seguridad completa?
4. ¿Qué tipos de discos pueden utilizarse para los respaldos? ¿Influyen en su duración? ¿Cuándo suele realizar una organización sus respaldos y restauraciones?
5. ¿Qué es una copia incremental y en qué se diferencia de una diferencial?
6. ¿Dónde podemos almacenar una copia de seguridad?
7. ¿En qué consiste la estrategia 3-2-1?
8. ¿Qué herramientas de cifrado permiten proteger las copias?
9. ¿Qué debemos considerar al hacer copias de seguridad en la nube?
10. ¿Qué es una SAN?
11. ¿Qué es un NAS? Incluir ejemplos comerciales y libres.

### Explicación del proceso práctico

12. Explicar la creación y restauración de copias de una carpeta de contenidos:
    - En una misma máquina virtual.
    - Entre dos máquinas virtuales: origen y destino.
13. Comparar rsync y tar.gz para desarrollar un script de copias de seguridad. Elegir una opción y justificarla.

### RSYNC

- ¿Qué es rsync y en qué contextos puede utilizarse?
- Presentar un ejemplo de uso entre dos máquinas virtuales, basado en la práctica realizada.

### TAR.GZ

- ¿Qué es tar.gz?
- ¿Cómo se utiliza para crear copias incrementales y diferenciales? Explicar el procedimiento y mostrar las pruebas realizadas.

## Preparación del entorno

Completar estos datos durante la práctica, sin asumir direcciones ni configuraciones.

| Dato | Valor |
|---|---|
| Distribución y versión de la MV de origen | |
| Usuario e IP de origen | |
| Distribución y versión de la MV de destino | |
| Usuario e IP de destino | |
| Modo de red utilizado | |
| Ruta de la carpeta de origen | |
| Ruta de la carpeta de destino | |

El enunciado propone Ubuntu o Debian y, para la comunicación entre equipos, red solo anfitrión o dos MV con adaptador puente. La opción concreta se documentará cuando se prepare el laboratorio.

## Desarrollo práctico

Los siguientes apartados son la estructura de trabajo. Los comandos ejecutados, observaciones y capturas se incorporarán al realizar cada bloque; no se presentan como pruebas completadas.

### 1. Copia y recuperación local

- Crear las carpetas de origen y destino y varios archivos de prueba.
- Realizar la primera copia y repetirla después de modificar el origen con rsync.
- Comparar el uso de la ruta de origen con y sin barra final.
- Comprobar contenido, atributos y propietarios en el destino.
- Documentar qué ocurre después de modificar un archivo y volver a copiar.
- Recuperar el contenido invirtiendo correctamente origen y destino.
- Repetir la creación y restauración con tar.gz.
- Desarrollar y comprobar las copias incrementales y diferenciales con tar.gz solicitadas en el informe.

### 2. Copia remota

- Preparar la conectividad entre origen y destino y el servicio SSH.
- Realizar la copia remota con rsync.
- Documentar las rutas y la diferencia entre usar o no la barra final.
- Repetir el proceso de copia remota utilizando tar.gz.

### 3. Recuperación remota

- Restaurar los datos desde el equipo remoto con rsync.
- Repetir la restauración utilizando tar.gz.
- Comprobar el contenido recuperado y registrar el resultado.

### 4. Autenticación para automatizar

- Probar y explicar el método con sshpass solicitado en el enunciado.
- Probar y explicar la autenticación mediante claves SSH.
- Comparar ambos métodos y sus implicaciones de seguridad.
- No publicar contraseñas, claves privadas ni otros secretos en el repositorio.

### 5. Scripts y programación con crontab

- Crear un script para realizar una copia remota con rsync.
- Crear otro script para realizar una copia remota con tar.gz.
- Explicar qué es crontab y documentar la programación elegida.
- Ejecutar y comprobar ambos scripts y las tareas programadas.
- Incorporar el código y los resultados cuando se hayan probado.

## Comparación y conclusiones

Desarrollar al finalizar la práctica:

- Diferencias observadas entre rsync y tar.gz.
- Comportamiento de las copias y restauraciones locales y remotas.
- Elección razonada de herramienta para la automatización.
- Limitaciones y problemas encontrados.

## Fuentes

Enunciado de referencia: M10378T1A3-rsync-tar.docx, aportado para esta actividad. Las fuentes técnicas utilizadas se añadirán conforme se consulten.

[Volver al índice de actividades](../README.md)
