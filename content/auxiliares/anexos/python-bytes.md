---
title: "Strings y Bytes en Python 3"
date: 2020-03-24T10:00:00-03:00
draft: false
menu:
  auxiliares:
    parent: "anexos"
---

A continuación explicaremos cómo manejar strings y bytes en Python 3.

[Fuente de casi todo este material](https://docs.python.org/3/library/stdtypes.html)

### Tipo str en Python

El tipo `str` en Python 3 representa una **cadena de texto inmutable**. 

#### Representación al hacer `print`

Un objeto de tipo `str` suele ser mostrado entre comillas cuando se le hace `print`.

`"esto es un objeto de tipo str"`

#### Declaración de un `str`

Un `str` se puede declarar con:

* **comillas simples**: `'hola mundo'`
* **comillas dobles**: `"hola mundo"`
* **tres comillas simples (multilínea)**:

```python
'''
texto
multi
línea
'''
```

#### Transformar `str` en bytes

Para transformar un objeto de tipo `str` con encoding `UTF-8` en un objeto de tipo `bytes`, hay que ejecutar el método `encode()`:

```python
byte_str = "hola mundo".encode()
# byte_str = b'hola mundo'
```

#### Modificar caracteres del `str`

Los caracteres de un objeto `str` son inmutables. Lo que sí se puede hacer es crear un nuevo objeto que concatene las partes del `str` que queremos con el nuevo valor:

```python
palabra = "casa"
# si queremos cambiar el byte 3 a otro valor solo nos queda hacer esto:
palabra = palabra[:2] + "r" + palabra[3:]
# Ahora palabra es "cara"
```

### Tipo `bytes` en Python

El tipo `bytes` en Python 3 representa un **arreglo inmutable de bytes**. 

#### Representación al hacer `print`
Un objeto de tipo `bytes` es mostrado como un `str`, pero con una `b`antes de las comillas:

`b"esto es un objeto de tipo bytes"`

si el byte no es mostrable como un caracter, se muestra en **representación binaria**:

* Byte `0x00` se representa como `\x00`
* Byte `0x47` se representa como `G` (El código en bytes del caracter `G` es `0x47`).


#### Declaración de un `bytes`

Un objeto de tipo bytes se crea con el constructor `bytes(...)`, el cual puede recibir:
* **Un número `n`**: Crea un arreglo de bytes de tamaño `n` inicializado en bytes con valor `\x00`: `bytes(3) = b'\x00\x00\x00'`
* **Un iterable**: Crea un arreglo de bytes en el que intenta transformar cada elemento del iterable en un byte: `bytes([1,2,71]) = b'\x01\x02G'` 

También se puede declarar como un  `str`, anteponiendo una `b` a las comillas usadas:

* `b'hola mundo'`
* `b"hola mundo"`
* `b'''hola mundo'''`

#### Transformar `bytes` en `str`

Para transformar `bytes` en un `str` con encoding `utf-8`, usamos el método `.decode()`:

```python
b = bytes([72, 111, 108, 97])
b_str = b.decode()
# 'Hola'
```

si algún byte no representa un caracter en el encoding `utf-8` (como muy probablemente ocurra con gran parte de los caracteres de un texto cifrado), la función lanzará una excepción.

#### Modificar caracteres del `bytes`

Al igual que en el caso de `str`, los caracteres de bytes no son directamente modificables. Una solución es usar el mismo truco que en el caso `str`:

```python
b = b'\x01\x02\x00\x04'
# si queremos cambiar el byte 3 a otro valor solo nos queda hacer esto:
b = b[:2] + b'\x03' + b[3:]
# Ahora b es b'\x01\x02\x03\x04'
```

La otra alternativa que podemos usar es crear un objeto de tipo `bytearray` a partir del objeto de tipo bytes. La próxima sección da más detalles de los objetos de tipo `bytearray`.

#### Extra: ¿Qué es un hexstring y qué tiene que ver con bytes?

Un `hexstring` no es mas que un objeto de tipo `str` que intenta representar datos en hexadecimal. El objeto sin embargo es de tipo `str` posee las mismas propiedades que cualquier otro `str`, salvo que para estar bien formado además debe:
* Tener una cantidad par de caracteres
* Solo estar compuesto de números y las letras `a`, `b`, `c`, `d`, `e`, `f`

Para transformar un hexstring a un objeto de tipo bytes, usamos esta función:

```python
b = bytes.fromhex("486f6c61")
# b será b'\x48\x6f\x6c\x61', o mejor dicho, b'Hola'
```


### Tipo `bytearray` en Python

El tipo `bytearray` en Python 3 representa un **arreglo mutable de bytes**, el cual se comporta prácticamente igual que el tipo `bytes`, salvo que es posible mutar sus valores.

#### Representación al hacer `print`
Un objeto de tipo `bytearray` es mostrado como un `bytes`, pero dentro de `bytearray()`:

`bytearray(b'esto es un objeto de tipo bytearray')`

Al igual que en el caso de `bytes`, si el byte no es mostrable como un caracter, se muestra en **representación binaria**:

* Byte `0x00` se representa como `\x00`
* Byte `0x47` se representa como `G` (El código en bytes del caracter `G` es `0x47`).


#### Declaración de un `bytearray`

Un objeto de tipo bytes se crea con el constructor `bytearray(...)`, el cual puede recibir:
* **Nada**, creándose un arreglo de tamaño 0 de tipo `bytearray()`
* **Un número `n`**: Crea un arreglo de bytes de tamaño `n` inicializado en bytes con valor `\x00`: `bytearray(3) = bytearray(b'\x00\x00\x00')`
* **Un iterable**: Crea un arreglo de bytes en el que intenta transformar cada elemento del iterable en un byte: `bytearray([1,2,71]) = bytearray(b'\x01\x02G')` 

#### Transformar `bytearray` en `str`

Para transformar `bytes` en un `str` con encoding `utf-8`, usamos el método `.decode()`:

```python
b = bytearray([72, 111, 108, 97])
b_str = b.decode()
# 'Hola'
```

si algún byte no representa un caracter en el encoding `utf-8` (como muy probablemente ocurra con gran parte de los caracteres de un texto cifrado), la función lanzará una excepción.

#### Modificar caracteres del `bytearray`

Los caracteres del `bytearray` se modifican como si de cualquier arreglo de Python se tratase, pero asignando números de entre 0 y 255 a las celdas:

```python
b = bytearray(b'\x01\x02\x00\x04')
# si queremos cambiar el byte 3 a otro valor solo nos queda hacer esto:
b[2] = 20 # equivale a \x14 en decimal
# Ahora b es bytearray(b'\x01\x02\x14\x04')
```

#### Extra: ¿Puedo pasar directamente de hexstring a bytearray?

Para transformar un hexstring a un objeto de tipo `bytearray`, usamos esta función:

```python
b = bytearray.fromhex("486f6c61")
# b será bytearray(b'\x48\x6f\x6c\x61'), o mejor dicho, bytearray(b'Hola')
```


