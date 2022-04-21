---
title: "CRIME Attack"
date: 2022-03-24T10:00:00-03:00
draft: false
menu:
  auxiliares:
    parent: "anexos"
---

CRIME (_Compression Ratio Info-leak Made Easy_) es una vulnerabilidad encontrada el año 2012, la cual ocurre al utilizar cifrado y compresión al mismo tiempo y que permite exfiltrar información sin conocer la llave de cifrado.

La hipótesis de su existencia fue levantada por Adam Langley, y fue [demostrada](https://docs.google.com/presentation/d/11eBmGiHbYcHR9gL5nDyZChu_-lCa2GizeuOfaLU2HOU) por los investigadores Juliano Rizzo y Thai Duong (los mismos de la vulnerabilidad BREACH).

## Supuestos

* Atacante puede ver el tráfico en el canal de comunicación.
* Datos viajan cifrados entre servidor y víctima, pero son comprimidos antes de cifrarse.
* Atacante puede obligar a víctima a cargar rutas específicas del sitio a atacar.
* Atacante quiere robarse una cookie u otro valor importante de una consulta HTTPS regular.

En otras palabras, aparte de ver (aunque cifrado) el tráfico entre la víctima y el servidor, el atacante tiene la posibilidad de gatillar consultas arbitrarias al servidor de parte del cliente del cual se quieren exfiltrar las cookies del usuario. 

Esto podría realizarse en un sitio con contenido controlado por el atacante (por ejemplo, un blog con imágenes agregadas por el atacante) o con una vulnerabilidad web como _Cross Site Scripting_ (La veremos más adelante en el curso).

## Compresión + Cifrado

La gracia de cifrar datos es que no podemos saber nada de lo que éstos representan en texto plano.

Bueno, casi nada. En general podemos conocer más o menos el largo. (Más o menos porque el texto suele tener padding en algunos modos de cifrado)

Por otro lado, los algoritmos de compresión sin pérdida sí nos permiten obtener algo de información de un dato parcialmente desconocido si éste posee una componente de texto elegida por el atacante. A esto se le suele llamar *Oráculo de Compresión*.

El tamaño del texto comprimido al cifrarse no varía (mucho), por lo que el cifrado no afecta (mucho) al rendimiento del oráculo de compresión.

## Implementación Genérica 

En la implementación genérica, asumimos que hay texto desconocido antes y después del texto que podemos enviar de entrada, y que dentro del texto desconocido, hay tanto una parte conocida (`KNOWN`) como una parte desconocida inmediatamente después de lo conocido (`SECRET`)

A continuación mostramos una implementación recursiva y genérica de cómo usar la información entregada por un algoritmo de compresión para detectar el texto comprimido.


`Función COMPRESSION_ORACLE(DATA)`
1. Sean `GZIP(X)` la versión comprimida de un bytearray o string X y `L(X)` la longitud del bytearray o string X.
1. Devuelve `L(GZIP(UNKNOWN_START + DATA + SECRET_END))`, donde `UNKNOWN_START` y `UNKNOWN_END` son valores desconocidos.

`Función ALGORITHM(KNOWN)`
1. Sean `W` el alfabeto de posibles caracteres en el secreto que queremos encontrar y `KNOWN` una pequeña parte conocida del texto que no controlamos que es adyacente al secreto que queremos encontrar.
1. Definimos `POSSIBLE` como un arreglo vacío con todas las respuestas posibles.
1. Definimos `UNCOMPRESSED_LENGTH` como `L(COMPRESS_ORACLE(KNOWN + y))`, con `y` un conjunto de varios (¿5 a 10?) caracteres fuera de `W`
1. Definimos `NO_W` como un conjunto de entre 5 y 10 caracteres no pertenecientes a `W`.
1. Para cada caracter `c` en `W`:
  1. Definimos `BASE_LENGTH = COMPRESS_ORACLE(KNOWN + NO_W + C)`
  1. Definimos `C_LENGTH = COMPRESS_ORACLE(KNOWN + C + NO_W)`
  1. Si `C_LENGTH < BASE_LENGTH`, agregamos `c` a la lista `POSSIBLE`.
1. Definimos `RESPONSES` como una lista vacía
1. Si `POSSIBLE` no está vacío, para cada caracter `p` en `POSSIBLE`, `RESPONSES += ALGORITHM(KNOWN+C)`
1. Devolver `RESPONSES`

`RESPONSES` contendrá todas las posibles soluciones para `SECRET` justo después de `KNOWN`

Notar que para que el algoritmo pueda partir adecuadamente, la primera invocación de `ALGORITHM` debe no ser vacía (y ojalá lo más larga posible).


## Cifradores de Bloque y compresión

El algoritmo anterior no nos sirve tal cual con cifradores de bloque, dado que el largo del texto cifrado no es directamente proporcional al largo del texto plano, debido a la necesidad de agregar _padding_ al texto.

El caso ideal sería que `BASE_LENGTH` generara un texto comprimido con padding de exactamente 1 bloque (16 bytes). De esta forma, cuando comprimamos el texto con un `c` candidato, el padding bajará a 1 byte, reduciéndose el número de bloques en 1.

Si seguimos extendiendo el texto, el tamaño comprimido debiese ser el mismo, por lo que el primer padding adecuado que calculemos servirá de ahí hasta exfiltrar completamente el secreto.

Entonces, lo que haremos es agregar antes de `KNOWN` (el texto inicial conocido) un texto compuesto de caracteres aleatorios (para evitar su compresión) denominado `BASURA`, cuyo largo es el adecuado para cumplir la condición de padding.

`Función COMPUTE_PADDING(KNOWN):`
1. Calcular `BASE_LENGTH = COMPRESS_ORACLE(KNOWN)`
1. Definir `NEW_LENGTH = BASE_LENGTH`
1. Definir `BASURA = ''` (String vacío)
1. Mientras `NEW_LENGTH == BASE_LENGTH`:
  1. Redefinir `NEW_LENGTH = COMPRESS_ORACLE(BASURA + KNOWN)`
  1. Si `NEW_LENGTH <= BASE_LENGTH`:
    1. Redefinir `BASURA += RANDOM(W)`, donde `RANDOM(Z)` obtiene un caracter aleatorio del alfabeto `Z`
1. Devolver `BASURA[:-1]`, es decir, `BASURA` sin su último caracter (que es el texto cuyo largo el el máximo posible en el que el largo no aumenta)

Luego, redefinir `KNOWN` como `BASURA + KNOWN` y usar el algoritmo original.

## Enlaces

* [DEFLATE](), el algoritmo de compresión usado en GZIP, usa una combinazión de LZ77 y Codificación de Huffman
* [LZ77](https://archive.ph/20130107232302/http://oldwww.rasip.fer.hr/research/compress/algorithms/fund/lz/lz77.html)
* [Huffman Coding](https://courses.cs.washington.edu/courses/cse143/10su/lectures/8-13/22-huffman.pdf)
* [CRIME-poc](https://github.com/mpgn/CRIME-poc/blob/master/CRIME-cbc-poc.py): Implementación en Python. Muy parecida a lo que se busca en la tarea 2 y base de esta explicación.