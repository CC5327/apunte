---
title: "Auxiliar 2"
date: 2020-03-26T20:12:32-03:00
draft: true
weight: 2
menu:
  auxiliares:
    parent: "enunciados"
---

En esta _auxiliar_ veremos más en profundidad las diferencias entre cifradores de bloque y cifradores de flujo, además de entender un poco mejor el funcionamiento de algunos cifradores de flujo conocidos.

## Resumen Cifradores

A continuación se muestra una tabla comparativa entre cifradores de flujo y de bloque.


|  Característica | Cifrador de Bloque | Cifrador de Flujo |
|-----------------|--------------------|-------------------|
| ¿Es capaz de cifrar mensajes de largo arbitrario? | Sí, al combinarse con un modo de operación | Sí, por sí solo |
| ¿Resiste Ataque _Two Time Pad_? | Sí (gracias a difusión de resultado) | No (debido a que los bytes encriptados mantienen su posición)
| ¿Requiere Padding? | Sí, ya que cifra bloques de tamaño determinado | No, ya que cifra de a unidades pequeñas (Bit a bit, byte a byte) |
| ¿Ofrece _Confidencialidad Perfecta_? | No | No
|  Ejemplos       | DES, 3DES ,AES      | ARC4, Salsa20, ChaCha20 |        

## Cifradores de Flujo conocidos

Ahora discutiremos algunas implementaciones de cifradores de flujo conocidos.

{{< alert icon="👉" >}}
Ojo, si bien acá hay unas implementaciones de ejemplo, hay que recordar que en la vida real muy probablemente no debes tú mismo implementar estas operaciones. Busca alguna librería existente que haga este trabajo por ti.
{{< /alert >}}

### ARC4

ARC4 toma su nombre del cifrador de flujo patentado RC4, desarrollado por RSA el año 1987. En el año 1994, una lista de correo _Cypherpunk_ (Activistas pro-privacidad) filtra su implementación, a partir de lo cual sus implementaciones de código abierto se empiezan a llamar _ARC4_ (Alleged RC4) para evitar problemas de marca.

ARC4 también es bastante conocido por estar relacionado a _WEP_ (Wired Equivalent Privacy), _WPA_ (Wireless Privacy Algorithm), _SSL_ (Secure Sockets Layer) y _TLS_ (Transport Layer Security).

En los últimos años se le han encontrado falencias y vulnerabilidades a ARC4 (fundamentalmente al usar llaves débiles), por lo que su uso ya no es recomendado.

#### ¿Cómo funciona?

ARC4 considera dos algoritmos:

* **Preparación de Llave** _(Key Scheduling Algorithm)_: Se usa para preparar un arreglo S con la permutación de todos los posibles valores de un byte, a partir de una llave dada. Por lo general, la llave utilizada se suele combinar con un IV para evitar repetición de valores.

```python
# key es la llave, de largo entre 1 y 256
def KSA():
    S = [0] * 256
    for i in range(256)
        S[i] = i
    j = 0
    for i in range(255):
        j = (j + S[i] + key[i % len(key)]) % 256
        S[i], S[j] = S[j], S[i]
    return S
```

* **Algoritmo de Generación Pseudoaleatoria** _(Pseudo Random Generation Algorithm)_: Cada vez que se necesite, este algoritmo permite generar el próximo byte a usarse para encriptar un texto. La encriptación (como en todos los cifradores de flujo vistos) se realiza aplicando XOR entre el valor devuelto y el byte a encriptar.

```python
# Asumir que son globales
i = 0 
j = 0 
def Encrypt(plain):
    encrypted = []
    for x in range(len(plain)):
        i = (i + 1) % 256
        j = (j + S[i]) % 256
        S[i], S[j] = S[j], S[i]
        K = S[S[i] + S[j] % 256]
        encrypted.append(plain[x] ^ K)
    return encrypted
```

![Muestra de fase de generación](https://en.wikipedia.org/wiki/File:RC4.svg)
Muestra de fase de generación.


**Trivia**: Algunos sistemas operativos (Como Linux y OpenBSD) incluyen un generador de números aleatorio llamado `arc4random`. Como su nombre indica, inicialmente estaba basado en ARC4. Sin embargo, en los últimos años su implementación fue cambiada a ChaCha20 en varios sistemas operativos. Hoy en día, si llamas al `man` del algoritmo, sus mantenedores aclaran que el significado del comando es _A Replacement Call For Random`.



### Salsa20 y ChaCha20

Salsa20 es un algoritmo de cifrado de flujo creado el 2005, mientras que ChaCha20 está basado en el algoritmo anterior y fue creado el 2008. Está basado en una función pseudoaleatoria que ejecuta operaciones de rotación, suma aritmética y XOR. Su estado inicial está compuesto de una llave de 256 bits, un nonce de 64 bits, posición de 64 bits y 128 bits fijos equivalentes a la palabra en ASCII "expand 32-byte k". Estos valores están dispuestos en una matriz de 4 x 4 de la siguiente forma (en los paréntesis va el "número de bloque"):


|        |       |       |       |
|--------|-------|-------|-------|
| "expa" (0) | Llave (1) | Llave (2)| Llave (3)|
| Llave (4) | "nd 3" (5) | Nonce (6) | Nonce (7)|
| Pos (8) | Pos (9) | "2-by" (10) | Llave (11)|
| Llave (12) | Llave (13) | Llave (14) | "te k" (15)|




Asumiendo la existencia de esta función auxiliar:

```python
def rot32(a, b):
    return a << b | a >> (32 - b)
```
Se define una operación principal que se ejecuta a lo largo del proceso de generación de aleatoriedad. A continuación la versión en python de esta operación:

```python
def QR(a, b, c, d):
    b = b ^ rot32((a + d), 7)
    c = c ^ rot32((b + a), 9)
    d = d ^ rot32((c + b), 13)
    a = a ^ rot32((d + c), 18)
    return a, b, c, d
```

Aplicamos esta operación en cada fila en las rondas pares y en cada columna en las impares.

En Salsa20, se suelen usar 20 rondas.

El algoritmo para procesar un bloque es:

```python
def process_block(input_block): 
    i = 0
    x = [0]*16
    out = [0]*16
    for i in range(16):
        x[i] = input_block[i]
    for i in range(ROUNDS/2):
        # Columnas
        x[0], x[4], x[8], x[12] = QR(x[0], x[4], x[8], x[12])
        x[5], x[9], x[13], x[1] = QR(x[5], x[9], x[13], x[1])
        x[10], x[14], x[2], x[6] = QR(x[10], x[14], x[2], x[6])
        x[15], x[3], x[7], x[11] = QR(x[15], x[3], x[7], x[11])

        # Filas
        QR(x[0], x[1], x[2], x[3])
        QR(x[5], x[6], x[7], x[4])
        QR(x[10], x[11], x[8], x[9])
        QR(x[13], x[14], x[15], x[12])
    for i in range(16):
        out[i] = (x[i] + input_block[i]) % (1 << 32)
    return out
```

En el caso de ChaCha20, el algoritmo es muy similar, pero cambian tres partes:

* Disposición de estado inicial:

|        |       |       |       |
|--------|-------|-------|-------|
| "expa" (0) | "nd 3" (1) | "2-by" (2)| "te k" (3)|
| Llave (4) | Llave (5) | Llave (6) | Llave (7)|
| Llave (8) | Llave (9) | Llave (10) | Llave (11)|
| Nonce (12) | Nonce (13) | Nonce (14) |  Nonce (15)|


* En vez de filas se usan diagonales

```python
    ...
        # Columnas
        QR(x[0], x[4], x[8], x[12])
        QR(x[5], x[9], x[13], x[1])
        QR(x[10], x[14], x[2], x[6])
        QR(x[15], x[3], x[7], x[11])

        # Diagonales
        QR(x[0], x[5], x[10], x[15])
        QR(x[1], x[6], x[11], x[12])
        QR(x[2], x[7], x[8], x[13])
        QR(x[3], x[4], x[9], x[14])
    ...
```

* Función QR es distinta

```python
def QR(a, b, c, d):
    a += b
    d ^= a
    d = rot32(d, 16)
    
    c += d
    b ^= c
    b = rot32(b, 12)

    a += b
    d ^= a
    d = rot32(d, 8)
    
    c += d
    b ^= c
    b = rot32(b, 7)
    return a, b, c, d
```

Los entendidos dicen que la nueva QR sobre las diagonales ayudan a la difusión del input, es decir, que cambiando un bit del mensaje, su versión encriptada varía en una mayor cantidad de bits. Además, las operaciones definidas en las _quarter-rounds_ (QRs) son más amigables con algunas arquitecturas de procesadores.


### Ejercicios

* Intenta hacer una implementación "de juguete" de ambos algoritmos. Por simplicidad, asume que el input en el caso de ChaCha/Salsa será siempre de tamaño múltiplo de 16 bytes.
