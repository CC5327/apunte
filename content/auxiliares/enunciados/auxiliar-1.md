---
title: "Auxiliar 1"
date: 2020-03-18T20:12:32-03:00
draft: false
weight: 1
menu:
  auxiliares:
    parent: "enunciados"
---

En esta _auxiliar_ veremos el modo de uso de la librería [Cryptography](https://cryptography.io), además de algunos ejercicios relacionados con ella.

{{< alert icon="👉" >}}

En aplicaciones reales, es muy poco probable que tengas que usar estas primitivas de forma directa. Por lo mismo, la librería _Cryptography_ las agrupa en un paquete denominado _hazmat_ (en español, _material peligroso_ que causa riesgos a la vida o al ambiente si no es manejado con precaución)
{{< /alert >}}

## Cifradores de Bloque

Puedes ver la documentación de _Cryptography_ sobre este tema [acá](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/).

_Criptography_ define la siguiente interfaz para el uso de Cifradores:

* `encryptor()`: Devuelve un contexto usable para encriptar datos
* `decryptor()`: Devuelve un contexto usable para desencriptar datos

Al mismo tiempo, tanto `encryptor` como `decryptor` implementan los siguientes métodos

* `update(_msg_)`: Agrega bytes a encriptar. Devuelve parte de los bytes encriptados (puede quedarse con algunos que no completan un bloque todavía)
* `finalize()`: Devuelve los bloques restantes a encriptar, _paddeados_ de ser necesario para completar un bloque.

Para levantar un cifrador de bloque, es necesario llamar al constructor de la clase `Cipher` con un _algoritmo_ y un _modo_: 

```python
import os
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
backend = default_backend() # Configuración que la librería pide pero no usa por un problema de diseño
key = os.urandom(32) # Llave usada por el cifrador de bloque
iv = os.urandom(16) # Vector de inicialización usado por el modo
cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=backend)
encryptor = cipher.encryptor() # Contexto de cifrado
ct = encryptor.update(b"mensaje secreto!") # Entrega parte de lo encriptado
ct += encryptor.finalize() # Devuelve todo lo encriptado, en caso de haber quedado datos sin devolver anteriormente
decryptor = cipher.decryptor() # Contexto de descifrado
print(decryptor.update(ct) + decryptor.finalize()) # devuelve el texto completo descifrado
# La última instrucción devolvería b'mensaje secreto'
```

(Ejemplo obtenido de la documentación de _Cryptography_)

### Algoritmos

Los siguientes algoritmos son soportados por la librería:

* [AES](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.AES)
* [Camellia](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.Camellia)
* [ChaCha20](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.ChaCha20)
* [3DES](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.TripleDES)
* [CAST5](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.CAST5)
* [SEED](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.SEED)
* [Blowfish](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.Blowfish) _Inseguro_
* [ARC4](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.ARC4) _Inseguro_
* [IDEA](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.algorithms.IDEA) _Inseguro_

En todos los casos, el argumento del constructor recibe la llave simétrica a usar para cifrar la información.

### Modos

La librería soporta los siguientes modos: 

* [Cipher Block Chaining (CBC)](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.modes.CBC)
* [Counter (CTR)](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.modes.CBC)
* [Output Feedback (OFB)](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.modes.OFB)
* [Cipher Feedback (CFB)](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.modes.CFB)
* 
* [Galois Counter Mode (GCM)](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.modes.GCM) _Encriptación autenticada_
* [XTS](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.modes.XTS)
* [Electronic Code Book (ECB)](https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/#cryptography.hazmat.primitives.ciphers.modes.ECB) _Inseguro. Además requiere input con padding adecuado_

En todos los casos se recibe como primer argumento un _nonce_ o _initialization vector_. Este valor entrega aleatoriedad al resultado del modo usado, y puede ser público sin comprometer la seguridad del sistema.

## Autentificación de Mensajes

_Criptography_ define la siguiente interfaz para el uso de MAC.

* `update(msg)`: Similar al caso de cifrado, agrega bytes a firmar.
* `finalize()`: Devuelve la firma producida sobre los datos recibidos por `update`.
* `verify(sig)`: Compara la firma producida sobre los datos recibidos por `update`, con la firma recibida.

La librería soporta los siguientes tipos de MAC:

* [HMAC](https://cryptography.io/en/latest/hazmat/primitives/mac/hmac/). Su constructor recibe una llave secreta, un algoritmo de hash y un _backend_ (usar `default_backend()`)
* [CMAC](https://cryptography.io/en/latest/hazmat/primitives/mac/cmac/). Su constructor recibe un algoritmo de cifrado (con llave definida) y un _backend_ (usar `default_backend()`)
* [Poly1305](https://cryptography.io/en/latest/hazmat/primitives/mac/poly1305/). Su constructor recibe solamente una llave.

La documentación de _Cryptography_ da el siguiente ejemplo para el MAC _Poly1305_: 

```python
from cryptography.hazmat.primitives import poly1305
p = poly1305.Poly1305(key)
p.update(b"message to authenticate")
print(p.finalize())
b'T\xae\xff3\xbdW\xef\xd5r\x01\xe2n=\xb7\xd2h'
p = poly1305.Poly1305(key)
p.update(b"message to authenticate")
p.verify(b"an incorrect tag") # Debería tirar una excepción
```

## Encriptar y autentificar

_Cryptography_ además provee de los siguientes modos de cifrado que integran autentificación, combinando tipos generalmente usados en conjunto:

* [ChaCha20Poly1305](https://cryptography.io/en/latest/hazmat/primitives/aead/#cryptography.hazmat.primitives.ciphers.aead.ChaCha20Poly1305). Su constructor recibe solamente una llave.
* [AES-GCM](https://cryptography.io/en/latest/hazmat/primitives/aead/#cryptography.hazmat.primitives.ciphers.aead.AESGCM). Su constructor recibe solamente una llave.
* [AES-CCM](https://cryptography.io/en/latest/hazmat/primitives/aead/#cryptography.hazmat.primitives.ciphers.aead.ChaCha20Poly1305). Su constructor recibe una llave y un tamaño para el tag generado (por defecto es 16 y no se recomienda cambiarlo)

Ejemplo con _ChaCha20Poly1305_ de documentación de _Cryptography_:

```python
import os
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
data = b"a secret message"
aad = b"authenticated but unencrypted data"
key = AESGCM.generate_key(bit_length=128)
aesgcm = AESGCM(key)
nonce = os.urandom(12)
ct = aesgcm.encrypt(nonce, data, aad)
print(aesgcm.decrypt(nonce, ct, aad))
# Devuelve b'a secret message'
```

## Funciones de Hash

La interfaz de las funciones de hash en _Cryptography_ define los siguientes métodos:

* `update(msg)`: Similar al caso de cifrado, agrega bytes a hashear.
* `finalize()`: Devuelve el hash producido sobre los datos recibidos por `update`.

La librería soporta las siguientes funciones de Hash:

* [Familia SHA-2](https://cryptography.io/en/latest/hazmat/primitives/cryptographic-hashes/#sha-2-family), es decir, _SHA224_, _SHA256_, _SHA384_, _SHA512_, _SHA512-224_ y _SHA512-256_.
* [Blake2](https://cryptography.io/en/latest/hazmat/primitives/cryptographic-hashes/#blake2), en versiones optimizadas para distintos procesadores. El constructor recibe un parámetro equivalente al tamaño del hash deseado, entre 1 y 64 bytes.
* [Familia SHA-3](https://cryptography.io/en/latest/hazmat/primitives/cryptographic-hashes/#sha-3-family) , es decir, _SHA3-224_, _SHA3-256_, _SHA3-384_, _SHA3-512_, _SHAKE128_ y _SHAKE256_. Las últimas dos funciones de hash reciben de parámetro el tamaño del hash deseado.
* [SHA-1](https://cryptography.io/en/latest/hazmat/primitives/cryptographic-hashes/#sha-1) _Insegura_
* [MD5](https://cryptography.io/en/latest/hazmat/primitives/cryptographic-hashes/#cryptography.hazmat.primitives.hashes.MD5) _Insegura_

Ejemplo de la librería:

```python
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
digest = hashes.Hash(hashes.SHA256(), backend=default_backend())
digest.update(b"abc")
digest.update(b"123")
print(digest.finalize())
# mostraría b'l\xa1=R\xcap\xc8\x83\xe0\xf0\xbb\x10\x1eBZ\x89\xe8bM\xe5\x1d\xb2\xd29%\x93\xafj\x84\x11\x80\x90'
```

## Números Pseudo aleatorios

En Python, se suele usar la función `os.urandom(len)` para generar aleatoriedad criptográficamente segura. Sin embargo, como se vio en clases, muchas veces esta seguridad depende de la cantidad de entropía a la que nuestro computador tiene acceso.

En Linux, la función anteriormente mencionada obtiene aleatoriedad del dispositivo `/dev/urandom`, mientras que en Windows se obtiene de la función `CryptGenRandom`.

Por lo tanto, al crear variables aleatorias para su uso en otras funciones criptográficas (como por ejemplo, para claves o vectores de inicialización), se debe usar `os.urandom(len)`.

Desde Python 3.6, es posible usar la función `secrets.token_bytes(n)` de la librería [secrets](https://docs.python.org/3/library/secrets.html) para obtener un arreglo de bytes de largo n. 

## Ejemplos prácticos de temas vistos en clase

A continuación veremos algunos problemas vistos en clase, pero usando código en Python. El código fuente usado se puede encontrar en la sección _Material Docente_ del curso.

### Cifrado de Bloques

En clases vimos que el Modo ECB puede filtrar información estructural de los datos encriptados. El típico ejemplo de este problema es la siguiente imagen de la mascota de Linux, Tux.

#### Imagen Original
![Imagen Original](https://upload.wikimedia.org/wikipedia/commons/5/56/Tux.jpg) 
#### Imagen encriptada en ECB
![Imagen Encriptada en ECB](https://upload.wikimedia.org/wikipedia/commons/f/f0/Tux_ecb.jpg) 
#### Imagen encriptada con un modo más seguro
![Imagen Encriptada con CBC o CTR](https://upload.wikimedia.org/wikipedia/commons/a/a0/Tux_secure.jpg) 

_(La historia de cómo esta imagen se hizo tan famosa es bastante interesante, y la pueden encontrar [acá](https://blog.filippo.io/the-ecb-penguin/))_

El código `encrypt_image.py` toma la imagen `tux.png` y la encripta en modo ECB, dejándola en el archivo `tux_encrypted.png`. Si bien cada vez que ejecutes el código la imagen será distinta, la silueta se debiese de poder ver claramente en la mayoría de los casos.

* Cambia el cifrador de bloques y revisa si esta situación cambia
* Cambia el modo de encriptación y revisa si esta situación cambia

### Hashing

A veces, al bajar archivos de Internet, es normal encontrarse con que al lado de lo descargado hay un _hash_. Este valor se usa para demostrar que el archivo que bajaste es el mismo que quién lo publicó quería que bajaras, ya que dentro de las propiedades de un buen hash criptográfico, se encuentra la dificultad de encontrar un segundo valor que al ser hasheado entregue el mismo hash que otro valor.

En algunos casos, hasta el día de hoy se siguen usando funciones de hash rotas (como MD5 y SHA1) para realizar este cálculo, lo que hace que esta verificación pierda valor.

_([Acá](https://alf.nu/SHA1) puedes ver un ejemplo que utiliza la vulnerabilidad SHAttered, descubierta el 2017, para generar 2 PDF distintos con el mismo valor de hash)_

Sin embargo, incluso usando funciones de hash seguras, nada asegura que el usuario comparará letra por letra el hash calculado sobre el archivo con el hash publicado. Muchas veces, uno se conforma comparando los primeros y/o últimos caracteres de ambos hashes.

Se les pide modificar `script_virus.sh`, de tal forma que sus primeros 2 y últimos 2 caracteres del hash _SHA256_ sean los mismos que los del archivo `script_bueno.sh`, y siga ejecutando el código que ejecuta actualmente.

Para determinar cómo leer y calcular el hash de un archivo, basarse en el script `calculate_hash.py` adjunto en el material de la clase.

### Generación de números aleatorios

Usando la versión modificada del generador pseudoaleatorio definido en clases, disponible en el archivo `prng.py`, generar una clave de 32 bytes. Luego, intentar adivinar la clave generada aprovechándose de un problema de implementación fundamental de este generador.

* ¿Cómo varía la efectividad la estrategia según el tamaño de `SEED_SIZE`?
* ¿Cómo varía la efectividad de la estrategia según el tamaño de `KEY_SIZE`?
