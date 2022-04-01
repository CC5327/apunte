---
title: "Tarea 1: Criptografía"
date: 2020-04-24T10:00:00-03:00
draft: false
weight: 1
---

## Instrucciones

Antes de partir esta tarea, lea la sección de [reglas de las tareas](../reglas).

## Preguntas

### P1: Criptografía Asimétrica

En [este repositorio](https://github.com/cc5327/lab1-p1) encontrará un cifrador de mensajes usando RSA, con problemas de implementación importantes. Al ejecutar este programa, se crean 2 archivos. Uno de ellos es un mensaje aleatorio de largo variable en texto plano, el otro es el mismo mensaje, pero cifrado con una llave generada por el código.

Recuerde instalar los requerimientos de librerías usando el comando `pip install -r requirements.txt`. (La librería `criptography` )

a. (30%) Comente el/los problemas de implementación que encuentre en las funciones que implementan RSA. Explique por qué usted encuentra que son problemas.

**Hint**: Puede ser un solo problema. Lo importante es que sea el mismo problema que les permita realizar la parte siguiente.

**Hint 2**: Revise en específico las funciones que implementan la generación de llaves RSA en utils.

b. (30%) Desarrolle y explique una estrategia que se aproveche de los problemas encontrados en la parte anterior, de forma de **siempre** ser capaz de descifrar los mensajes cifrados generados por la implementación anterior.

c. (40%) Implemente en código la estrategia desarrollada en la sección anterior.

**Hint:** Las funciones en `utils.py` y la librería built-in [Decimal](https://docs.python.org/3/library/decimal.html) debiesen bastarle para resolver esta pregunta. Para encontrar la llave privada no es necesario usar directamente la librería `Criptography`, pero necesitará instalarla para que las funciones de `utils.py` funcionen y para descifrar el mensaje cifrado.

**Otro Hint:** La librería **Decimal** les servirá para realizar operaciones como `raíz cuadrada`. Para pasar un número de `int` a `Decimal`, simplemente pásenlo como primer argumento al constructor de la clase `Decimal`. También les recomendamos cambiar la `precisión` de los números en Decimal a un valor grande.:

```python
import decimal
decimal.getcontext().prec = 4096
x = decimal.Decimal(5)
```

Si desean pasar de `Decimal` a `int` (las funciones de utils reciben `int`), basta con que lo casteen como `int`:
```python
x_int = int(x)
```

### P2: Criptografía Simétrica

En esta pregunta, programaremos un ataque conocido como `Padding Oracle`, el cual consiste en aprovecharse del `padding` que se agrega a mensajes para obtener información del mensaje cifrado, sin necesidad de conocer la llave usada para cifrar.

Le recomendamos revisar el [anexo](../../auxiliares/anexos/padding-oracle.html) para entender mejor cómo llevar a cabo el ataque.

Les recomendamos partir la tarea con el código base de [este repositorio](https://github.com/cc5327/lab1-p2).

#### Servicios a atacar

Dejamos dos servicios **accesibles usando la VPN del CEC** corriendo en la IP `172.17.69.107`. El servicio A (puerto 5312) y el servicio B (puerto 5313). Pueden comunicarse directamente a estos servicios usando el código base proporcionado.

* El servicio en el puerto A recibirá un mensaje de ustedes y les entregará directamente un texto en hexadecimal correspondiente al mensaje cifrado en AES-CBC, el cual contiene su mensaje y una contraseña.
* El servicio en el puerto B recibirá el texto hexadecimal correspondiente al mensaje cifrado, y les indicará el mensaje que ustedes le entregaron, o un error en caso de fallo.

#### Preguntas

a. (10%) Comente qué ocurre en cada servicio con distintos tipos de entradas. En el caso del servicio A. ¿Qué pasa con el largo de la salida al variar el largo de la entrada? En el caso del servicio B, ¿Qué pasa si varía caracteres del valor recibido de A al enviarlos a B? ¿Qué pasa si envía algo completamente distinto? 

b. (10%) Utilice el código base para crear un programa que envíe un texto al servicio A, reciba su respuesta, envíe esa respuesta al servicio B y reciba la respuesta de este servicio.

c. (20%) Usando el código anterior, determine una estrategia para calcular el tamaño del bloque usado en el texto cifrado a través de una función que lo calcule de forma automática. Explique justificadamente por qué ocurre esto.

**Hint**: Probablemente ya conozca el tamaño del bloque (más arriba dice que el cifrador es AES el cual tiene un bloque de tamaño fijo). Sin embargo, se busca que expliquen cómo podrían aprovechar las funcionalidades del sistema para determinarlo en casos genéricos, en los que no manejan ese dato.)

d. (20%) Cree una función que permita descifrar el último carácter del texto cifrado.

e. (20%) Modifique la función de la parte anterior para descifrar un bloque completo.

f. (20%) Ejecute satisfactoriamente el ataque descrito en la parte anterior sobre todo el texto cifrado y registre su contenido en el informe de la pregunta.
