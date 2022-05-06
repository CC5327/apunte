---
title: "Auxiliar 4"
date: 2020-05-21T10:10:32-03:00
draft: false
weight: 4
menu:
  auxiliares:
    parent: "enunciados"
---

En esta auxiliar probaremos unos ejemplos de código de vulnerabilidades y bugs vistos en clases, usando una máquina virtual con Lubuntu 20.04.

## Usar la máquina virtual

Para usar la máquina virtual necesitan ejecutar las siguientes tareas:

* Descargar e instalar [VirtualBox](https://virtualbox.org)
* Descargar [la máquina virtual](https://drive.google.com/open?id=1W9Mz843KbC1PympEOwSzeER9FE-6DftR)
* Importar la máquina virtual con VirtualBox (File -> Import Appliance)

{{< alert icon="👉" >}}
Si tienen problemas al ejecutar la máquina virtual (por la falta de un driver USB) hagan lo siguiente:

 * Hagan click derecho en la máquina virtual y luego hagan click en "settings".
 * Vayan al tab USB, y luego hagan click en la opción "USB 1.1 (OHCI) Controller".
 * Intenten iniciar nuevamente la máquina virtual.
{{< /alert >}}

{{< alert icon="👉" >}}
Si el error lanzado es de virtualización, necesitan activar la "Virtualización de Hardware" en la BIOS de su computador. Comuníquense con el equipo docente si necesitan ayuda para hacer esto.
{{< /alert >}}


Tanto el usuario como la contraseña de la máquina son el código del curso, en minúsculas.


## Configuración actual de la máquina virtual

La máquina virtual viene instalada con el sistema operativo [Lubuntu 20.04](https://lubuntu.me), y se encuentra configurada con las siguientes opciones:

* ASLR desactivado: `echo 0 > /proc/sys/kernel/randomize_va_space` como usuario `root`
* `build-essential` instalado: `sudo apt install build-essential`, con GCC 9.3.0
* `gdb` instalado: `sudo apt install gdb`
* NX desactivado: Opciones `noexec=off` y `noexec32=off` agregadas como parámetros de inicialización de kernel 
* [vscodium](https://vscodium.com) instalado (VSCode, pero sin la telemetría de Microsoft)
* VirtualBox Guest Additions (permite interacción más fluída con el sistema anfitrión)

## Códigos de ejemplo

Pueden bajar los ejemplos de este [repositorio en Github](https://github.com/cc5312/cfexamples).

### El `Makefile`

El proyecto integra un archivo con el nombre `Makefile`, el cual declara las flags necesarias para la compilación de cada proyecto.

Los archivos `Makefile` contienen una lista de `targets` a construir, cada uno declarando un nombre, otros `targets` requeridos y el/los comandos a ejecutar para completarse:

```makefile
target: requirement1 requirement2
  echo "completing target"

requirement1:
  echo "completing requirement 1"

requirement2:
  echo "completing requirement 2"
```

### Compilar ejemplos

Para ejecutar el primer `target` (el cual en este proyecto compila todos los ejemplos), basta con correr `make` en la carpeta del makefile. Si se quiere ejecutar un target en específico, basta con correr `make <nombre_target>`, cambiando `<nombre_target>` por el nombre del target a ejecutar.

Les recomendamos revisar el archivo `Makefile` para ver cuáles flags se están usando en cada ejemplo.


### Flags utilizadas

Para activar/desactivar ciertas opciones de protección del stack, se usan las siguientes flags:

#### Canarios

* `-fstack-protector`: , Agrega un __canario_ a funciones y a buffers de tamaño mayor o igual a 8
* `-fstack-protector-strong`: (Activada por defecto en Lubuntu) Agrega un __canario_ a funciones y a buffers de tamaño mayor o igual a 4, definiciones de arreglos locales y referencias a `frames` locales.
* `-fstack-protector-all`: agrega un _canario_ a todas las funciones y buffers
* `-fnostack-protector`: Desactiva las protecciones de stack usando canarios


#### Protecciones contra ejecución de código en stack

* `-z execstack`: Desactiva la protección de ejecución de código en el stack.

#### Protecciones contra ROP

* `-fcf-protection=[full|branch|return|none]`: Define protecciones contra ataques de control de flujo, como ROP y Return-To-Libc. La flag `return` corresponde a revisar la integridad de las protecciones al momento de volver de un `jmp` o `call`. La flag `branch` corresponde a revisar la integridad de las protecciones al momento de realizar un `jmp` o `call`. La flag `full` utiliza ambas medidas anteriores y la flag `none` no utiliza ninguna.

### Usando GDB

Revisen el [anexo correspondiente](../../anexos/gdb) para entender mejor cómo usar GDB al momento de correr un programa.

### Ejecutando los ejemplos

Luego de compilarlos, podrán ejecutar los binarios producidos a partir de los siguientes códigos fuente:

#### `format_string.c`

Este código genera solamente un binario: `format_string`, el cual recibe un string que es impreso directamente como único argumento de `printf`.

* Prueba ejecutar este código con el mismo input tanto con ASLR desactivado (por defecto en la máquina) como activándolo:
  * Puedes activar ASLR ejecutando `echo 2 > /proc/sys/kernel/randomize_va_space` como usuario `root`. Recuerda desactivarlo después (también puedes reiniciar la máquina virtual para desactivarlo). ¿Qué ocurre de distinto en ambos casos?
* Intenta distintos tipos de string con directrices de formato. En especial, te recomendamos intentar con `%p %p %p %p %p...` hasta encontrar un patrón. Intenta identificar a qué corresponde este patrón.
* En la rutina "main" hay una variable `password` con un texto largo. Intenta mostrarla con un string de formato especial con ayuda de GDB.

#### `buffer_overflow.c`

Este código fuente genera dos binarios distintos: 

* `buffer_overflow`: el cual se compila con flags que permiten realizar un buffer overflow de forma correcta (desactivando canarios)
* `buffer_overflow_protected`: el cual se compila con las flags por defecto de gcc en Lubuntu

Ambos programas reciben de input un string de tamaño arbitrario, el cual intentan copiar en un buffer de tamaño 8

Recomendamos que experimenten con distintos inputs y vean qué ocurre con el flujo del programa. Además, tambien es buena idea ejecutar las instrucciones usando GDB para ver el estado del stack.

En la tarea 2, vamos a intentar ejecutar _shellcode_ a través del _buffer overflow_ del programa vulnerable.


