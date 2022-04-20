---
title: "Padding Oracle Attack a CBC"
date: 2020-03-24T10:00:00-03:00
draft: false
menu:
  auxiliares:
    parent: "anexos"
---

Los ataques de tipo _Padding Oracle_ se aprovechan de la estructura de mensajes cifrados en esquemas que utilizan _padding_ para obtener el texto plano de un mensaje cifrado, realizando una cantidad de intentos considerablemente menor que los necesarios para obtenerlo mediante fuerza bruta y sin revelar la clave de cifrado.

El caso específico de _padding oracle attack_ que veremos ahora aplica si se cumplen estas condiciones:
 - Se usa criptografía simétrica, con cualquier tipo de cifrador de bloque y el modo de operación _CBC_.
 - El bloque utiliza algún tipo de _padding_ predecible, como por ejemplo, [`PKCS#7`](https://tools.ietf.org/html/rfc5652#section-6.3).
 - El servicio que recibe el mensaje cifrado no tiene forma de autentificarlo, ya que de lo contrario, se podrían detectar las modificaciones al texto cifrado.
 - El servicio que recibe el mensaje cifrado entrega feedback de algún tipo, como un mensaje de error, cuando el padding está mal formado.

## El modo CBC

Antes de hablar del ataque, conviene recordar cómo funciona el cifrado y descifrado al usar el modo de operación _CBC_.

![Cifrar en modo CBC](https://upload.wikimedia.org/wikipedia/commons/thumb/8/80/CBC_encryption.svg/902px-CBC_encryption.svg.png)

La imagen anterior muestra que para un bloque \\(B_i\\), \\(C(B_i) = BlockCipher(B_i \oplus C(B_{i-1}))\\) (Asumiendo \\(B_0 = IV\\) y contando los bloques de texto plano a partir del 1).

![Descifrar en modo CBC](https://upload.wikimedia.org/wikipedia/commons/thumb/2/2a/CBC_decryption.svg/902px-CBC_decryption.svg.png)

Por otro lado, el proceso inverso muestra que para un bloque cifrado \\(C_i\\), \\(D(C_i) = BlockDecipher(C_i) \oplus C_{i-1}) = B_i\\). Le llamaremos \\(I_i\\) al estado intermedio del bloque descifrado antes de reaizar el \\(\oplus\\), por lo que la relación se puede escribir como \\(B_i = I_i \oplus C_{i-1}\\). Una propiedad interesante de \\(I_i\\) con respecto a  \\(B_i\\) es que cada byte de \\(C_{i-1}\\) afecta de forma directa al byte en la misma posición de \\(I_i\\).

![Estados intermedios](./CBC_intermediate.svg)

## Padding PKCS#7

El _padding_ _PKCS#7_ consiste en agregar entre 1 y \\(BlockSize\\) bytes al final de un mensaje, de tal forma que el valor de todos estos bytes debe ser igual a el número de bytes que le faltan al mensaje para completar el bloque. En caso que el tamaño del mensaje sea múltiplo del tamaño del bloque, se agrega al mensaje un bloque extra en el que todos los bytes tienen valor igual a \\(BlockSize\\).

Por ejemplo, si el mensaje a paddear es `Hola mundo` y el \\(BlockSize\\) es 16, el mensaje con _padding_ es `Hola mundo[0x06][0x06][0x06][0x06][0x06][0x06]` (Los valores entre corchetes representan a un byte con ese valor hexadecimal). Mientras que si el mensaje a paddear es `Mensaje Completo` (largo 16) y el \\(BlockSize\\) es 16 bytes, el mismo mensaje con _padding_ sería `Mensaje Completo[0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10][0x10]` (`0x10` es 16 en hexadecimal). Como se observa, este método de padding tiene sentido para bloques de tamaño de hasta 256 bytes.

Antes de cifrar un mensaje, se calcula y anexa el padding correspondiente. Luego, es el texto con padding el que se cifra. En el caso del proceso de descifrado, primero se descifra el mensaje usando el procedimiento habitual, y antes de entregarlo descifrado, se revisa su último byte y se remueven tantos bytes como la cantidad indicada por este valor.

## El ataque

Un detalle importante que conviene notar de la imagen que muestra los estados intermedios es que el texto descifrado del último bloque depende solamente del bloque intermedio (desconocido para nosotros, ya que no tenemos la llave del cifrador de bloque) y del bloque cifrado anterior (dato conocido, porque manejamos el texto cifrado). Por lo tanto, lo que haremos será crear una estrategia para poder encontrar el valor de los bloques intermedios, ya que con esa información podremos descifrar el mensaje sin necesidad de conocer la llave.

Si volvemos a revisar esta misma imagen, podremos notar notar que podríamos obtener el estado intermedio del bloque si manejáramos el texto plano y el valor del bloque cifrado anteriormente. Esto puede sonar un poco como un problema circular (dado que son dos los valores que no conocemos, el texto plano y el bloque intermedio), sin embargo, la estructura del modo CBC nos permite modificar el texto cifrado para que al descifrarse resulte en un texto plano distinto de forma controlada. 

Lo anterior es posible dado que el texto descifrado depende del bloque anterior y el bloque intermedio final, si quisiéramos cambiar el valor del texto descifrado final, bastaría con cambiar el valor del bloque cifrado anterior. Este cambio al bloque cifrado anterior nos impediría descifrar de forma correcta ese bloque (ya que un pequeño cambio en un bloque cifrado con cifrador de bloque genera un gran cambio en su resultado descifrado), pero al mismo tiempo afectaría de forma controlada al resultado del bloque siguiente (ya que es operado a través de `xor` con el resultado del bloque intermedio), lo que repercute también en un cambio en el texto plano a descifrar.

Por lo tanto, podemos aprovechar la información que nos entrega el servidor (cuándo el padding del mensaje recibido es correcto y cuando no) para modificar el último byte del mensaje descifrado de forma tal que termine con el padding `0x01` a través de modificar el último byte del texto cifrado del bloque anterior hasta que el mensaje completo pase por padding. Si hacemos esto, podemos determinar el valor del bloque intermedio haciendo `xor` entre el byte modificado y `0x01`. 

El valor del byte final del bloque intermedio encontrado sería exactamente el mismo que antes de modificar el último byte del bloque anterior, debido a que el bloque intermedio final depende solamente del cifrador de bloque, la llave usada y el bloque cifrado, el cual no hemos modificado. Ahora que tenemos el último byte del bloque intermedio, solamente necesitamos hacer `xor` entre él y el último byte original del bloque anterior para obtener el último byte en texto plano del texto cifrado original. ¡Esto nos requirió solamente 256 intentos!

En el caso de otros bytes en el último bloque, podemos usar la información manejada hasta ahora para ajustar el padding a otros valores, de tal forma que el primer byte de padding de izquierda a derecha sea el byte que estamos buscando, ya que cuando encontremos el valor que lo deja como padding correcto, podremos repetir el experimento anterior de forma de determinar el valor descifrado de esa parte del texto.


## El algoritmo

Ahora veremos la implementación del ataque anterior, dividiéndola en 3 fases. Primero intentaremos conseguir solamente el valor en texto plano del último byte cifrado del mensaje. Posteriormente, repetiremos este ataque para obtener todo el último bloque. Finalmente, veremos cómo obtener los demás bloques.

### Obtener el último byte de un mensaje

En esta fase, el ataque consiste en abusar del padding final de un mensaje encriptado simétricamente y del feedback que nos entrega la aplicación al recibir un mensaje mal cifrado. 

En esta parte usaremos la notación \\(B_i[j]\\) para hablar del j-ésimo byte del bloque de texto plano número i. Los bloques irán numerados de \\(0\\) (el IV) a \\(n\\) (el último bloque del mensaje), mientras que los bytes en un bloque irán numerados de \\(0\\) a \\(BlockSize-1\\).

Podemos realizar la primera parte del ataque usando \\(C_n[BlockSize-1]\\) y (\\(C_{n-1}[BlockSize-1]\\)) (últimos bytes de los dos últimos bloques cifrados). Con esta información, intentaremos determinar el estado intermedio de (\\(I_n[BlockSize-1]\\)) a partir de generar un mensaje \\(D_n\\) cuyo texto plano tenga padding de `[0x01]` (es decir, \\(D_n[BlockSize-1] = [0x01]\\)), ya que este estado al validarse no generaría error de validación de padding y solamente depende del valor de 1 byte de padding. Recordemos que \\(D_n[BlockSize-1]\\) depende tanto de \\(I_n[BlockSize-1]\\) (no lo podemos tocar) como de \\(C_{n-1}[BlockSize-1]\\) (lo conocemos y podemos modificar), por lo que podemos ir iterando entre **los 256 posibles valores** de \\(C_{n-1}[BlockSize-1]\\) (al bloque modificado lo llamaremos \\(M_{n-1}\\)) hasta llegar a un \\(M_{n-1}\\) que haga que \\(D_n[BlockSize-1]\\).ea igual a `[0x01]`.

Sin embargo, existe la remota posibilidad de conseguir un resultado en el que el texto cifrado valida, pero su último byte en texto plano no es `[0x01]`. Esto puede ocurrir solamente si el penúltimo byte del último bloque es igual a `[0x02]`. ya que en este caso, el mensaje con padding validaría si el último byte también es `[0x02]`. Para descartar este caso, podemos modificar el valor del penúltimo byte de \\(M_{n-1}\\) por uno distinto al usado anteriormente. Si el mensaje sigue validando, significa que transformamos el último byte en `[0x01]`.

¿Qué hacemos con esta información? Conociendo \\(M_{n-1}[BlockSize-1]\\) y \\(D_n[BlockSize-1]\\) (`[0x01]`), podemos calcular \\(I_n[BlockSize-1]\\) haciendo \\(\oplus\\) entre ambos valores (Revisar gráfico de descifrado CBC para entender por qué es así). Además, como \\(I_n[BlockSize-1]\\) depende solamente de \\(C_n[BlockSize-1]\\) y no hemos modificado \\(C_n\\) en todo el proceso, estamos seguros que es el mismo valor que el obtenido antes de modificar \\(C_{n-1}\\).

Finalmente, para obtener \\(B_n[BlockSize-1]\\). basta con ejecutar el algoritmo de descifrado regular de CBC, es decir, hacer \\(\oplus\\) entre \\(I_n[BlockSize-1]\\) (valor obtenido recién) y \\(C_{n-1}[BlockSize-1]\\) (último bloque del mensaje original). Con esto, pudimos descubrir 1 byte del texto cifrado en solo 256 intentos. Si mantenemos esta eficiencia para los otros caracteres, podemos encontrar el texto plano sin conocer la llave en tiempo lineal.

En resumen, el algoritmo se podría describir de la forma siguiente:
 * Copiar \\(C_{n-1}\\) a una nueva variable llamada \\(M_{n-1}\\). y cambiar \\(M_{n-1}[BlockSize-1]\\) a `[0x00]`.
 * Enviar mensaje cifrado modificado a validar. (Esto es, el mismo mensaje que antes, pero cambiando \\(C_{n-1}\\) por \\(M_{n-1}\\)).
 * Mientras mensaje no valide (\\(D_n[BlockSize-1] !=\\) `[0x01]`), aumentar \\(M_{n-1}[BlockSize-1]\\) en 1.
 * En el momento en que valide, significa que creamos un bloque \\(D_n\\) con padding correcto (casi seguramente su texto plano termina en `[0x01]`, pero podemos asegurarnos de aquello si repetimos el proceso cambiando \\(M_{n-1}[BlockSize-2]\\) por otro valor y la validación sigue pasando. Recuerda devolver el valor del byte luego de cambiarlo).
 * Hacemos \\(\oplus\\) entre el valor \\(M_{n-1}[BlockSize-1]\\) obtenido en el paso pasado y `[0x01]` (El texto plano que adivinamos), lo que nos entregará \\(I_n[BlockSize-1]\\)  
 * Hacemos \\(\oplus\\) entre \\(C_{n-1}[BlockSize-1]\\) y \\(I_n[BlockSize-1]\\). Esto nos entregará \\(B_n[BlockSize-1]\\).

### Obtener el último bloque completo

Hablemos en específico de la obtención del penúltimo byte antes de explicar el caso genérico. Conociendo el último byte del mensaje, podemos usar esta información para saber cómo modificar \\(C_{n-1}[BlockSize-1]\\). de forma que el valor de \\(D_n[BlockSize-1]\\) sea `[0x02]`. Esto se hace cambiando nuevamente el bloque \\(C_{n-1}\\) original por un bloque \\(M_{n-1}\\), y asignando a \\(M_{n-1}[BlockSize-1]\\) el valor de \\(I_{n}[BlockSize-1] \oplus\\)  `0x02`, es decir, el valor intermedio del byte obtenido en el paso anterior operado con el valor del nuevo padding. El último término anulará el byte original (dejándolo en `[0x00]`) y el del medio lo dejará tal cual como queremos.

Ahora, repetimos los pasos de la sección anterior sobre el penúltimo byte, buscando eso sí un padding de `[0x02]`. Esto se puede generalizar para todos los bytes del bloque, cambiando el padding en cada ocasión por el correspondiente.

En otras palabras, este sería el algoritmo genérico para cualquier byte del último bloque:
 * Si queremos descifrar el byte \\(i\\) del último bloque, y asumiendo que conocemos todos los bytes en \\(B_{n}[i+1:BlockSize]\\), para cada \\(j \in [i+1,BlockSize)\\) asignamos \\(M_{n-1}[j] = I_{n-1}[j] \oplus PaddingByte\\), donde \\(PaddingByte = BlockSize - i\\). Esto hará que los últimos bytes del mensaje de texto plano correspondan a un padding correcto cuando encontremos el valor del bloque número \\(i\\).
 * Ejecutamos la misma estrategia que para obtener el último byte, pero ahora asumiendo que si la validación pasa, el valor descifrado en este caso será igual a \\(PaddingByte\\). Es recomendable hacer la misma revisión que en el caso anterior para descartar paddings correctos debido a los datos del mensaje. (Cambiar el byte (i-1)-ésimo de \\(M_{n-1}\\) y verificar si sigue pasando la validación.)

 ### Obtener los demás bloques

 Dado que el valor de un bloque encriptado en modo CBC depende solamente de los bloques anteriores, podemos truncar del mensaje todos los bloques que ya conocemos, de forma de que el bloque desconocido más lejano del inicio del mensaje tome el rol de último bloque. Luego, se repiten los mismos pasos de las secciones anteriores. Es importante recordar que el bloque \\(B_0\\) corresponderá al vector de inicialización usado.

 Es más, si facilita la implementación del ataque, en algunos casos es posible enviar solamente 2 bloques al oráculo a la vez (el anterior o el IV, y el que queremos descifrar).

