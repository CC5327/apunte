---
title: "Tarea 4: Seguridad Web"
date: 2020-06-10T9:00:00-03:00
draft: false
weight: 3
---

## Instrucciones

La siguiente actividad está pensada para su desarrollo en un bloque de auxiliar y un poco más (~3 horas), en grupos de dos personas como máximo. Sin embargo, se les la posibilidad de entregarla hasta el **próximo viernes 26 de junio, a las 23:59 hrs, sin descuento por ello**.

Al igual que en el laboratorio anterior, antes de partir, recuerden leer la sección de [reglas de tareas](../reglas/).

### Pregunta única: Capture the flag

Este laboratorio está inspirado en los eventos de _Capture the Flag_ realizados por comunidades de hacking desde hace mucho tiempo. Si luego del laboratorio encuentran que les interesan este tipo de desafíos, les recomiendo partir por [este enlace](https://securityeducationresourcecollection.net/) (categoría: CTF) para buscar otros ejercicios similares y disponibles en Internet.

El objetivo de esta actividad es encontrar un valor secreto (Flag) escondido en la misma base de datos de la aplicación web entregada. La URL sobre la que debe trabajar cada grupo será enviada de forma directa, y es "secreta" (nadie aparte de ustedes y el equipo docente debería poder acceder al sitio de forma directa). Lo anterior es para evitar que externos no autorizados por ustedes interactúen con su instancia.

#### Primera parte: XSS (3 puntos)

Partiremos obteniendo acceso al panel de administración de la página entregada. Al intentar realizar inyecciones SQL tanto en el formulario de visitas como en el de inicio de sesión, notan que no están teniendo resultados. Sin embargo, ustedes saben de fuentes confiables que este restaurante tiene un empleado muy motivado e indeciso, el cual cambia el nombre del plato del día cada ciertos segundos y luego va a la página inicial a ver cómo quedó. Para poder hacer el cambio, el empleado se encuentra autentificado en el sitio como administrador.

Su objetivo es adquirir acceso como el usuario empleado del sitio a través de una inyección XSS en la base de datos del libro de visitas. Agregue el código que usó y describa en su informe los pasos que siguieron para esto. Es necesario que expliquen también al menos dos mitigaciones que hubiesen hecho que su ataque no resultara, con sus beneficios y limitaciones (las mitigaciones pueden ser al mismo problema o a problemas distintos).

**Consejo 1:** El empleado que navega por la página podría dejar de hacerlo temporalmente si ésta es severamente modificada (por ejemplo, agregando diálogos de alerta). Para corregir estos problemas, puede ir a la página `/reset.php`, lo cual devuelve la base de datos del sitio a su estado habitual.

#### Segunda parte: Inyección SQL Ciega (3 puntos)

En el panel de administración, encuentran el formulario que se usa para cambiar el plato del día. Experimentando, notan que algunas entradas entregan un error de tipo SQL. Sospechan entonces que es posible realizar una inyección SQL ciega para extraer la flag que busca.

Si bien ustedes sabe que podría [obtener información de la estructura de la tabla](http://pentestmonkey.net/cheat-sheet/sql-injection/postgres-sql-injection-cheat-sheet) con este tipo de inyecciones, prefieren preguntar a un informante infiltrado en el restaurante por la estructura de la tabla para ahorrar tiempo. Esta persona les dice que la tabla del plato del día se llama `configuraciones`, posee dos columnas (`nombre` y `valor`) y que se guarda la flag en la columna `valor` bajo el `nombre` `FLAG`. La flag está compuesta solamente letras mayúsculas, minúsculas y números.

Desarrollen un script que realice de forma automática las consultas sobre el formulario y extraiga la flag (incluyendo el código que usan para hacer la inyección SQL ciega). Expliquen en su informe los pasos que siguieron para esto y adjunten el script desarrollado. Expliquen además dos mitigaciones posibles para el o los problemas que le permitieron extraer este valor, con sus beneficios y limitaciones.

**Consejo 2:** Para el correcto funcionamiento del script, necesitan contar con la `cookie` de inicio de sesión del administrador. Si están usando la librería Python-Requests, es recomendable que sigan [este ejemplo](https://2.python-requests.org/en/master/user/quickstart/#cookies) para entender como agregar la cookie a sus consultas.

**Consejo 3:** La inyección SQL tal cual como se hizo en la auxiliar 5 traerá problemas. Esto debido a que la consulta SQL usada para cambiar el plato del día es algo similar a `UPDATE configuraciones SET value = '$plato' WHERE name = 'platodeldia'`. Si su inyección parte con `'; <lo demás> -- `, la primera instrucción SQL quedará de la forma `UPDATE configuraciones SET value = '';`. Esto cambia el valor de TODAS las filas de la tabla configuraciones, eliminando del paso el valor de `FLAG`. Por lo tanto, necesitan inventar una forma de "anular" el update anterior. Para devolver la flag a la tabla, ejecuten `reset.php` (Eliminará también los cambios en la tabla de comentarios).

**Consejo 4:** Se considerará válida una respuesta que no involucre inyecciones SQL ciegas, siempre y cuando se detalle completamente el método seguido y se adjunte un script que realice este trabajo de forma automática, contando solamente con la cookie de administrador.
