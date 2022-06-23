---
title: "Tarea 5: Seguridad de Redes"
date: 2022-06-22T9:00:00-03:00
draft: false
weight: 5
---

## Instrucciones

Al igual que en la tarea, antes de partir, recuerden leer la sección de [reglas de laboratorios](../reglas/).

Para el desarrollo completo de este laboratorio, es necesario que se encuentre conectado a la VPN del CEC. Siga las instrucciones de la [Auxiliar 5](../../auxiliares/enunciados/auxiliar-5) para esto.

### DNS Spoofing (6 puntos)

En esta pregunta atacaremos a un cliente en la IP `172.17.69.241` que resuelve RRs de tipo A sobre el dominio `spoofed.tarea5.cc5327.xor.cl` contra un resolver DNS dentro de la red del DCC cada 2 segundos. Sin embargo, las consultas DNS son realizadas siempre con el puerto de origen UDP `55312`, un ID de request DNS entre `1` y `1024`, **y sin revisar que la IP de origen sea la correcta**. Luego, el cliente se conecta vía `HTTP` a la url `http://<ip-resuelta>:5312?flag=<f>`, donde `f` es un código [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm).

Su objetivo es robar el código TOTP para un momento determinado. Para demostrar su obtención, necesita adjuntar tanto el código TOTP como el momento exacto de su obtención.

* (2 puntos) Expliquen la factibilidad de un ataque de tipo DNS Spoofing si cada uno de estos problemas de implementación no existiera (manteniendo los otros problemas de implementación):
  * (0.6 puntos): Si el puerto de origen de la consulta DNS variase constantemente.
  * (0.6 puntos): Si el ID de request pudiese tomar cualquier valor.
  * (0.8 puntos): Si el cliente no esperara paquetes de cualquier IP de origen.

* (2 puntos) Teniendo en cuenta el impacto de cada problema de implementación mencionado anteriormente, creen un script basado en [Scapy](https://scapy.readthedocs.io/en/latest/) que permita realizar un ataque de DNS Spoofing sobre las respuestas del cliente al dominio consultado, cambiando la IP del dominio `spoofed.tarea5.cc5327.xor.cl` por la IP de ustedes en la VPN del CEC. 

**Consejo**: Usen el código de ejemplo (La función de paquetes DNS) enlazado al inicio de este enunciado y modifíquenlo para realizar este ataque. Pueden aprovechar de realizar **una sola consulta DNS** ) (la pueden reutilizar durante todo el ataque) a un resolver accesible a ustedes (pueden usar `1.1.1.1` (Google), `8.8.8.8` (Cloudflare), `172.17.66.9` (Resolver dentro de la VPN) y luego modificar los campos correspondientes en la respuesta (IP destino, ID de consulta DNS, IP asociada al dominio consultado) para hacer el ataque.

* (2 puntos) Recuperen los parámetros de la consulta de la víctima levantando un servidor HTTP en el puerto 5312. Adjunten el código usado para levantar el servidor (pueden basarse en el código visto en la auxiliar 5). Repitan el ataque del punto anterior hasta recibir una consulta al servidor levantado y anoten tanto el código TOTP como la fecha y hora exactas en la que lo obtuvieron. 

**Consejo 3**: El código TOTP está compuesto de 6 dígitos y cambia cada 30 segundos, por lo que deben registrar la hora de su obtención lo más cerca posible al momento en el que reciben el mensaje de la víctima. Les recomendamos automatizarlo en su script de Python.
