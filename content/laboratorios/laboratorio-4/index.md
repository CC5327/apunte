---
title: "Laboratorio 4: Seguridad de Redes"
date: 2020-07-03T9:00:00-03:00
draft: false
weight: 4
---

## Instrucciones

La siguiente actividad está pensada para su desarrollo en un bloque de auxiliar y un poco más (~3 horas), en grupos de dos personas como máximo. Sin embargo, se les la posibilidad de entregarla hasta el **próximo martes 21 de julio, a las 23:59 hrs, sin descuento por ello** (El plazo es un poco mayor a 7 días debido al feriado e interferiado).

Al igual que en el laboratorio anterior, antes de partir, recuerden leer la sección de [reglas de laboratorios](../).

Para el desarrollo completo de este laboratorio, es necesario que se encuentre conectado a la VPN del CEC. Siga las instrucciones de la [Auxiliar 5](../../auxiliares/auxiliar-5) para esto.

### Pregunta 1: Ataques de amplificación (3 puntos)

En esta pregunta compararemos tres distintos servicios conocidos por ser propensos a ataques de amplificación, todos corriendo en el dominio `lab4.cc5312.xor.cl`:

* [Memcached](https://github.com/memcached/memcached/blob/master/doc/protocol.txt) (Puerto 11211), usando los comandos:
    * `stats`
    * `get` y `set` usados en conjunto.
* [DNS](https://www.ietf.org/rfc/rfc1035.txt) (Puerto 53) ([Documentación en Scapy](https://scapy.readthedocs.io/en/latest/api/scapy.layers.dns.html#scapy.layers.dns.DNS))
, a través de consultas de tipo:
    * [ANY](https://blog.cloudflare.com/deprecating-dns-any-meta-query-type/) (qtype 255)
    * [AXFR](https://tools.ietf.org/html/rfc5936) (qtype 252)
    * [TXT](https://www.ietf.org/rfc/rfc1035.txt) (qtype 16)
* [NTP](http://www.ntp.org/) (Puerto 123) ([Documentación en Scapy](https://scapy.readthedocs.io/en/latest/api/scapy.layers.ntp.html?highlight=ntpPrivate)
), a través de comando:
    * [monlist](https://www.meinbergglobal.com/english/news/meinberg-security-advisory-mbgsa-1401-ntp-monlist-network-traffic-amplification-attacks.htm).

(1 punto por servicio) Para cada servicio, realicen las siguientes tareas:
 * (2 décimas) Expliquen, para cada comando del servicio, cuál es la utilidad original de éste y qué características de él lo vuelven adecuado para ejecutar ataques de amplificación, además de qué condiciones deben cumplirse en la configuración del servidor para poder ejecutar este ataque.
 * (2 décimas) Elaboren un script basado en [este código](https://github.com/cc5312/aux6) que calcule el tamaño de paquete recibido y enviado en cada comando estudiado del protocolo. Para calcular los tamaños de las respuestas y preguntas, pueden usar la función de Python `len()`.
 * (3 décimas) Determinen y justifiquen un método para maximizar la eficiencia en la amplificación de un mensaje que dependa solamente del atacante. Definiremos la **eficiencia de amplificación** \\(EF(M)\\) como \\(EF(M) = \frac{len(AMP(M))}{len(M)}\\), donde \\(M\\) es el mensaje de entrada, \\(AMP(M)\\) es el mensaje de salida y \\(len\\) es una función de longitud en bytes.
 * (3 décimas) Modifiquen el script inicial para que utilice el método programado por ustedes (por ejemplo, enviando uno o más paquetes configurados de tal forma que una respuesta sea bastante grande) y muestre la eficiencia de amplificación obtenida (El resultado entregado en la práctica por su script puede ser menor a la eficiencia calculada, pero deben explicar qué condiciones deberían darse para maximizar esta eficiencia).


Para cada servicio y comando, les recomendamos revisar la documentación enlazada en cada nombre para entender los cómo funcionan y contestan. También tienen a su disposición la documentación de la libería de [Scapy](https://scapy.readthedocs.io/en/latest/).

**Consejo para Memcached**: Entiendan bien para qué sirve y cómo se utiliza el servicio Memcached. Revisen solamente el modo de uso de los comandos `stats`  y el uso conjunto de los comandos `get` y `set`, definidos [acá](https://github.com/memcached/memcached/blob/master/doc/protocol.txt) (no hay que leer el documento entero, solo hay que entender las definiciones de los comandos mencionados).

**Consejo para DNS**: Enfóquense en las consultas de los tipos expuestos en la lista de arriba solamente. Consideren tanto los límites máximos de paquetes UDP según el [RFC1035](https://www.ietf.org/rfc/rfc1035.txt) (Sección 4.2.1), así como también las formas de maximizar la cantidad de RRs o el largo de respuestas en el protocolo.

**Consejo para NTP**: Consideren solamente el comando `MONLIST`, tal cual se usa en el código de ejemplo, e investiguen a partir de esta [documentación](https://www.meinbergglobal.com/english/news/meinberg-security-advisory-mbgsa-1401-ntp-monlist-network-traffic-amplification-attacks.htm) cómo podrían maximizar el tamaño de la respuesta.


### Pregunta 2: DNS Spoofing (3 puntos)

En esta pregunta atacaremos a un cliente en la IP `172.17.69.106` que resuelve RRs de tipo A sobre el dominio `spoofed.lab4.cc5312.xor.cl` contra un resolver DNS dentro de la red del DCC cada 2 segundos. Sin embargo, las consultas DNS son realizadas siempre con el puerto de origen UDP `55312`, un ID de request DNS entre `1` y `1024`, **y sin revisar que la IP de origen sea la correcta**. Luego, el cliente se conecta vía `HTTP` a la url `http://<ip-resuelta>:5312?flag=<f>`, donde `f` es un código [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm).

Su objetivo es robar el código TOTP para un momento determinado. Para demostrar su obtención, necesita adjuntar tanto el código TOTP como el momento exacto de su obtención.

* (1 punto) Expliquen la factibilidad de un ataque de tipo DNS Spoofing si cada uno de estos problemas de implementación no existiera (manteniendo los otros problemas de implementación):
  * (0.3 puntos): Si el puerto de origen de la consulta DNS variase constantemente.
  * (0.3 puntos): Si el ID de request pudiese tomar cualquier valor.
  * (0.4 puntos): Si el cliente no esperara paquetes de cualquier IP de origen.

* (1 punto) Teniendo en cuenta el impacto de cada problema de implementación mencionado anteriormente, creen un script basado en [Scapy](https://scapy.readthedocs.io/en/latest/) que permita realizar un ataque de DNS Spoofing sobre las respuestas del cliente al dominio consultado, cambiando la IP del dominio `spoofed.lab4.cc5312.xor.cl` por la IP de ustedes en la VPN del CEC. 

**Consejo**: Usen el código de ejemplo (La función de paquetes DNS) enlazado al inicio de este enunciado y modifíquenlo para realizar este ataque. Pueden aprovechar de realizar **una sola consulta DNS** ) (la pueden reutilizar durante todo el ataque) a un resolver accesible a ustedes (pueden usar `1.1.1.1` (Google), `8.8.8.8` (Cloudflare), `172.17.66.9` (Resolver dentro de la VPN) y luego modificar los campos correspondientes en la respuesta (IP destino, ID de consulta DNS, IP asociada al dominio consultado) para hacer el ataque.

* (1 punto) Recuperen los parámetros de la consulta `de la víctima levantando un servidor HTTP en el puerto 5312. Adjunten el código usado para levantar el servidor (pueden basarse en el código visto en la auxiliar 5). Repitan el ataque del punto anterior hasta recibir una consulta al servidor levantado y anoten tanto el código TOTP como la fecha y hora exactas en la que lo obtuvieron. 

**Consejo 3**: El código TOTP está compuesto de 6 dígitos y cambia cada 30 segundos, por lo que deben registrar la hora de su obtención lo más cerca posible al momento en el que reciben el mensaje de la víctima. Les recomendamos automatizarlo en su script de Python.
