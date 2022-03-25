---
title: "Laboratorio 2: Buffer Overflows"
date: 2020-06-05T9:00:00-03:00
draft: true
weight: 2
---

## Instrucciones

La siguiente actividad está pensada para su desarrollo en un bloque de auxiliar y un poco más (~3 horas), en grupos de dos personas como máximo. Sin embargo, se les la posibilidad de entregarla hasta el **próximo viernes 12 de junio, a las 10:00 hrs, sin descuento por ello**.

Al igual que en el laboratorio anterior, antes de partir, recuerden leer la sección de [reglas de laboratorios](reglas).

### Pregunta única: Shellcode

El objetivo de esta pregunta es explotar manualmente una vulnerabilidad de tipo _buffer overflow_ en [el binario de este archivo comprimido](buffer_overflow.zip) entregado con ayuda de GDB. Lo anterior deben realizarlo para que corra de forma apropiada en la [máquina virtual](https://drive.google.com/open?id=1W9Mz843KbC1PympEOwSzeER9FE-6DftR) del curso. Para resolver esta pregunta de forma más cómoda, puede apoyarse con scripts de Python que le ayuden a generar automáticamente las entradas.

El binario y el fuente que deben atacar lo pueden descargar desde [este enlace](buffer_overflow.zip). Su única función es recibir el texto entregado como primer argumento, copiarlo a un buffer de un tamaño limitado, y luego terminar.

{{< alert icon="👉" >}}

Si tienen problemas ejecutando `buffer_overflow`, deben activar el permiso de ejecución con este comando:

```bash
sudo chmod +x buffer_overflow
```

Las credenciales de la máquina las pueden encontrar en la [auxiliar 4](../../auxiliares/auxiliar-4).

{{< /alert >}}

#### Obtener la dirección de memoria del inicio del buffer y determinar la ubicación en la pila de `ret` que usa el frame f  (3 puntos)

a. (1,5 puntos) Utilizando GDB, depuren el binario para determinar la dirección de memoria del inicio del buffer. Adjunten una captura GDB mostrando la dirección de inicio del buffer e indiquen ya sea en la imagen o en texto cuál es la dirección de memoria buscada en cada caso.

b. (1,5 puntos) Determinen el espacio de memoria en el cual se guarda la dirección de retorno que usa el frame `f` al entrar a la función que copia el buffer utilizando GDB. Para ello, les recomendamos que ejecuten el programa sin overflow, ubiquen la línea en que se ejecuta `strcpy` al buffer, revisen el estado de la pila y finalmente identifiquen el valor que ésta contiene al momento de salir del frame (es decir, qué dirección de memoria se apunta). Adjunten en el informe la dirección de memoria que determinaron en cada caso con una captura de pantalla de GDB mostrándola.


#### Crear NOP Slide y saltar al inicio del buffer (2 puntos)

Desde esta parte, formaremos el payload necesario para ejecutar el ataque de desborde de buffer.

Sabiendo la dirección el inicio del buffer y la dirección de `ret`, calculen la cantidad de bytes entre ambas direcciones. Compruebe que una cadena de texto de prueba de ese tamaño pasada como entrada al programa lo hace caerse (ya que está sobreescribiendo `ret` con parte de su texto).

Luego de realizar lo anterior, reemplacen algunos de los caracteres de su cadena de prueba por un NOP slide de un tamaño razonable (>=24 bytes) utilizando la función de bash `printf` de la siguiente forma:

`printf "\x90\x90\x90"`

Lo anterior imprime tres veces el caracter de código hexadecimal `0x90`

Para hacer el reemplazo, les recomendamos usar interpolación de valores en bash. A continuación mostramos un ejemplo:

```bash
./lab2 $(printf "\x90\x90\x90\x90\x90\x90<...>")aaaaaaaaaaaaaaaa...
```

Esto imprimirá el byte \0x90 (NOP) una cantidad de veces determinada, seguido del caracter `a`. Tengan cuidado con no poner espacios entre `$()` y el texto porque serán considerados como entradas distintas.

Por último, reemplacen los últimos bytes del payload que está armando por una dirección de memoria cercana al inicio del buffer, en la que se encuentre algún byte del _NOP Slide_. 

Es importante tener en cuenta que no es posible ingresar el byte \x00. Esto es fácil de solucionar si es que el valor original en la posición que queremos reemplazar ya tenía estos \x00, ya que bastaría con dejarlos intactos. Les recomendamos que hagan varias pruebas con distintos largos y vayan viendo cómo queda la pila en cada caso

También es importante tener en cuenta que los valores se guardan en x86_64 en formato little endian, es decir, la dirección de memoria `0x0123456789abcdef` se escribiría como `$(printf "\xef\xcd\xab\x89\x\x67\x45\x23\x01")`

Para la revisión de esta sección, es necesario que envíen una captura de la pila en GDB al momento de terminar la copia de la variable externa. En esta captura deben destacar tanto el NOP slide como la dirección de retorno (que debe estar en el mismo lugar que la mostrada en la parte 1).

#### Inyectar Shellcode (1 punto)

Finalmente, queda agregar el shellcode al payload generándose.A continuación les adjuntamos un shellcode válido para su uso en x64. Este shellcode levanta un terminal al ejecutarse.

```bash
\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05
```

[Fuente](https://www.exploit-db.com/shellcodes/46907)


Agregue el shellcode inmediatamente a continuación del NOP slide **pero dejando una cantidad de bytes de relleno entre él y el final del buffer**. Comente las razones por las que cree que el shellcode no se puede colocar directamente al final del buffer.

Finalmente, ejecute el programa y compruebe que el shellcode se ejecuta, levantándose un terminal al correr el programa con el input alterado. Para probar el buen funcionamiento del terminal, ejecute algunos comandos en él (ejemplo: `whoami`, `echo "hola"`, `ls`). Agregue una captura del shellcode ejecutado y el terminal funcionando y adjunte el texto de entrada completo (con NOP slide, shellcode, relleno y dirección de retorno) que generó.

#### Pregunta Extra 1: repetir el experimento, pero sin ayuda de GDB (hasta 0.5 puntos extra para cualquier laboratorio, se puede entregar de forma individual o grupal)

Probablemente, si ejecutan el payload sin GDB, éste no funcione. La razón de esto es que las variables de ambiente cargadas con GDB y sin GDB son distintas, y éstas ocupan un espacio en la memoria del proceso que hace que cambien ligeramente las direcciones de memoria de la pila. En el anexo de GDB hay un enlace a Stack Overflow explicando mejor estas razones y cómo solucionar el problema.

Para obtener esta bonificación, es necesario que expliquen detalladamente un método para poder encontrar las direcciones de memoria correctas al ejecutar el programa con un payload sin GDB, además de por qué funcionaría (no es necesario que sea la más eficiente). Para la evaluación, deben adjuntar los pasos a seguir para replicar su método y una captura de pantalla que muestre que su método funciona. La respuesta no puede usar el script del enlace de Stack Overflow del anexo de GDB (aunque si la citan pueden entregar una explicación basada en ella).


#### Pregunta Extra 2: Usar otros shellcode (hasta 0.5 puntos extra para cualquier laboratorio, se puede entregar de forma individual o grupal)

Busque otros shellcode para Linux x86/64 que le interesen e intente agregarlos como payload a su ataque. Para completar este objetivo, debe adjuntar el enlace a la fuente del shellcode usado, el texto de entrada completo y la captura del shellcode ejecutándose.

#### Pregunta Extra 3: Crear su propio shellcode (hasta 1 punto extra para cualquier laboratorio, se puede entregar solamente de forma individual)

Explique detalladamente cómo crear shellcode para Linux x86_64 y entregue un `shellcode` creado por usted siguiendo ese método (no tiene que ser necesariamente complejo, basta con que ejecuten código no presente en el programa original). Puede usar librerías y programas externos existentes siempre y cuando los cite y explique en qué aporta cada uno de ellos. Para completar este objetivo, debe adjuntar la descripción de cómo hacerlo, el shellcode junto con una descripción de qué hace, el comando completo y una captura del shellcode funcionando.



### Créditos y más información

* Esta pregunta está inspirada en el contenido del curso online de desarrollo de exploits de [samclass.info](https://samsclass.info/127/127_F15.shtml). Si quieren adentrarse en la parte práctica de estos ataques, es un buen punto para partir.

* Si quieren conocer más de shellcodes, les recomendamos el libro `A Shellcoder's Handbook`. A pesar de ser algo antiguo, es una lectura entretenida para entender cómo funcionan los shellcodes.
