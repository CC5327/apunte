---
title: "Uso básico de GDB"
date: 2020-03-24T10:00:00-03:00
draft: false
menu:
  auxiliares:
    parent: "anexos"
---

GDB es una utilidad que permite depurar programas en varios lenguajes de programación. En el curso la usaremos para depurar programas en C.

{{< alert icon="👉" >}}
Para que el programa compilado pueda ser usado de forma adecuada en GDB, necesitan agregar la flag de compilación `-g3`, la cual almacena en el binario datos especiales para la depuración.
{{< /alert >}}

## Ejecutar un programa en GDB

Para ejecutar el código compilado en GDB, se usa el comando `env - gdb <nombre_programa>`, donde `<nombre_programa>` es el nombre del programa a depurar.

(¿Por qué partimos el comando con `env -`? Porque las variables de entorno afectarán la ubicación de nuestra pila. Pueden ver [esta pregunta de Stack Overflow](https://stackoverflow.com/a/17775966) para más información.)

Al partir GDB, el programa no se ejecuta todavía, pero GDB muestra una consola interactiva. Para conocer los comandos utilizables, es posible ejecutar `help`.

## Usar argumentos

Para ejecutar el código compilado en GDB y usar argumentos en el ejecutable, se usa el comando `gdb --args <nombre_programa> <args...>`, donde `<nombre_programa>` es el nombre del programa a depurar y `<args>` es uno o más argumentos que recibirá el programa a depurar.

## Explorar código asm

Para explorar una función en asm, se puede ejecutar el comando `disassemble <nombre_func>` o `disass <nombre_func>`, donde `<nombre_func>` es el nombre de la función que se quiere revisar. También se puede usar una dirección de memoria en vez de un nombre de función. Si no colocan nada luego de `disassemble` o `disass`, se mostrará el código asm cercano a la línea que se está ejecutando en ese momento.

## Colocar Breakpoints

Antes de la ejecución del programa, es necesario colocar _breakpoints_ o zonas en las cuales el código dejará de ejecutarse, a la espera del control de nosotros.

Los breakpoints se colocan con el comando `break <f>`, donde `<f>` corresponde a la función en la cual se detendrá la ejecución del programa, una dirección de memoria o el número de la línea en el código fuente.

Una idea para partir debuggeando en GDB es colocar un `break` en la función `main`. De esta forma podemos controlar el flujo del programa desde su inicio. Esto se hace escribiendo `break main`.

## Flujo del programa

Para partir el programa en GDB, puedes escribir `r`. El programa se detendrá tanto si espera input interactivo (función `gets()`) como si se colocó un _breakpoint_ en alguna función que se esté ejecutando.

En el caso de detención por un breakpoint, se pueden usar las siguientes opciones: 

* `frame <i>` o `f <i>`: les muestra la instrucción en la que se encuentran `i` frames más arriba del que se encuentran (conteo parte en 0). Pueden omitir el parámetro `<i>` para que GDB les muestre la línea de código del frame actual.
* `continue <i>` o `c <i>`: la cual continúa el flujo normal del programa, saltándose `i` breakpoints. Si omiten `<i>`, se continúa solamente hasta el breakpoint siguiente.
* `step <i>` o `s <i>`: la cual avanza `i` instrucciones en el programa. Si esta instrucción es una función, continúa revisando dentro del frame de la función. Es posible omitir el parámetro `<i>` para avanzar solamente una instrucción.
* `stepi <count>` o `si <count>`: les permite avanzar a `i` instrucciones en asm. Para ver en qué instrucción de asm van, pueden escribir `disass` (Una flecha en el lado izquierdo de la pantalla indica la instrucción actual). Al igual que con `step`, en caso de no colocar un número, 
* `next` o `n`: la cual pasa a la siguiente instrucción en el programa dentro del frame actual, es decir, si la instrucción es una función, la ejecuta sin pausar y se detiene justo después que ésta retorna.
* `finish` o `fin`: La cual termina el frame actual y se detiene apenas se vuelve al frame anterior.

Mientras se van ejecutando estos pasos, GDB muestra la línea de código en la que va el programa, lo que ayuda a ubicarse en su flujo.

## Información general

Existen comandos que muestran información general del estado actual del programa:

* `info args` muestra los argumentos recibidos por la función actual.
* `info all-registers` muestra el valor actual de todos los registros del procesador.
* `info frame` muestra información sobre el frame actual.
* `info locals` muestra las variables locales definidas hasta este momento.
* `x/<n><type><size> <addr>`, donde `<addr>` corresponde a una dirección de memoria o el nombre de un registro (`$rsp` por ejemplo para el tope de la pila), `n` representa a la cantidad de valores a leer desde ese punto, `type` corresponde al tipo de dato a interpretar y `size` corresponde al tamaño del dato; permite mostrar los `n` valores de tipo `type` en tamaño `size` desde la dirección `addr`. Algunas opciones de tamaño útiles son: 
   * `g`: 1 palabra grande (64 bits)
   * `w`: 1 palabra (32 bits)
   * `h`: 1 media palabra (16 bits)
   * `b`: 1 byte (8 bits)
Mientras que algunas opciones de tipo útiles son:
   * `o`: como un número octal
   * `x`: hexadecimal
   * `d`: decimal con signo
   * `u`: decimal sin signo
   * `t`: binario
   * `f`: coma flotante
   * `a`: dirección de memoria
   * `c`: caracter
   * `s`: Cadena de texto
   * `i`: Instrucción (como si fuera ASM)
* `p/<type> <var>`, donde `<var>` corresponde a una variable y `type` a un tipo al igual que en el comando anterior, permite examinar el valor de ese espacio. 
* `help <cmd>` les muestra ayuda sobre algún comando. Ejemplo: `help c`.

## Ejemplos de uso:

### ¿Cómo determinar el estado de la pila ?

* Correr el programa con GDB
* Colocar un breakpoint cerca de la variable que se quiere analizar (por ejemplo, en el nombre de la función): `b func_name`.
* Correr el programa con `r`. Se detendrá al inicio de la función.
* Avanzar con `n` hasta pasar la definición/asignación de la variable
* escribir `x/5xg $rsp` mostrará el valor de las 5 primeras palabras grandes de la pila. Al mismo tiempo, se muestra la dirección de memoria de cada valor en color azul al inicio de cada fila.
* Presionar `c` para que el programa continúe hasta su final.
