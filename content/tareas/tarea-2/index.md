---
title: "Tarea 2: Cripto Aplicaciones"
date: 2020-06-05T9:00:00-03:00
draft: false
weight: 2
---
{{< alert icon="👉" >}}

Al igual que en el laboratorio anterior, antes de partir, recuerden leer la sección de [reglas de tareas](reglas).

{{< /alert >}}

En esta auxiliar se replicará el espíritu del ataque CRIME, pero sin intervenir conexiones ni ejecutar solicitudes web. 

La replicación se hará de una forma muy parecida a la de la tarea 1, por lo que pueden reutilizar código de esa tarea si lo desean.

Para entender como funciona CRIME, ver [este anexo](../../auxiliares/anexos/crime). Al final hay un código en Python que puede servirles de inspiración para entender la implementación.

## Parte 1: Servidor (1 punto)

Deberán crear un programa en python que reciba conexiones y texto por el puerto 5327/TCP y devuelva una versión [comprimida con gzip](https://docs.python.org/3/library/gzip.html), cifrada (con AES-CBC, llave e IV pueden variar luego de cada llamada) y codificada en hexstring del siguiente texto (se marcan los saltos de línea como \n explícitamente):

```
GET <URL> HTTP/1.1\n
Cookie: Secret=<SECRET>\n
Host: cc5325.dcc\n
\n
```

El parámetro `<URL>` es el texto completo que se recibe por el socket, mientras que el parámetro `<SECRET>` es un texto alfanumérico (mayúsculas, minúsculas, números) de 32 caracteres que se configura por un archivo de configuración o como argumento en el programa servidor.

El programa servidor deberá registrar las acciones realizadas mediante un log, de tal forma que se muestre en cada linea:
    * Fecha del log
    * Texto en request recibida
    * IP en request recibida
    * Largo de respuesta no comprimida
    * Largo de respuesta comprimida con gzip y luego cifrada (largo del hexstring / 2)


Adjunte un "README.txt" que explique cómo ejecutar el programa.

* (1 punto) Programar el Servidor con las especificaciones anteriores.

## Parte 2: Cliente (3 puntos)

Deberán crear un programa en Python que se conecte a una IP definida como argumento o configuración y al puerto 5327/TCP. El programa debe ejecutar el ataque CRIME definido en el anexo enlazado al inicio de esta página.

El programa servidor deberá registrar las acciones realizadas mediante un log, de tal forma que se muestre en cada linea:
    * Fecha del log
    * Texto en request enviada
    * Largo de texto en request
    * lista de posibles valores de `Secret`

Adjunte un "README.txt" que explique cómo ejecutar el programa.

* Programar el Cliente con las especificaciones anteriores:
    * (1 punto) crear una rutina que se conecte al servidor y envíe mensajes predefinidos, logueando lo enviado.
    * (1 punto) crear un método que ejecute paso a paso el ataque CRIME definido en el anexo, logueando los valores posibles.
    * (1 punto) Conectar las dos partes anteriores para que el Cliente ejecute el ataque satisfactoriamente.

## Parte 3: Ataque y variaciones (2 puntos)

* (1 punto) Dividirse los programas cliente y servidor (cada integrante ejecuta uno de ellos). Apuntar el programa cliente a la IP en la VPN del CEC del usuario servidor (puerto 5327/TCP) y dejar correr. Comprobar que el secreto es recuperado entre todas las opciones posibles y adjuntar los logs.
* Invertir en el programa Servidor el orden de las operaciones de cifrado y compresión (es decir, ahora hay que cifrar primero y luego comprimir):
    * (0.3 puntos) ¿En qué afecta esto al protocolo?
    * (0.3 puntos) ¿En qué afecta esto al ataque?
    * (0.4 puntos) Proponga un cambio más eficiente al propuesto.
