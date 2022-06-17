---
title: "Auxiliar 5"
date: 2020-06-10T10:10:32-03:00
draft: false
weight: 5
menu:
  auxiliares:
    parent: "enunciados"
---

En esta auxiliar veremos cómo ejecutar algunos ataques sobre aplicaciones web inseguras. Pero antes, nos conectaremos a la VPN del CEC para poder acceder a las páginas que atacaremos.

## VPN del CEC

Configurar la VPN del CEC es bastante fácil, más aún desde que permiten conectarse usando [OpenVPN](https://openvpn.net).

Sigan el tutorial respectivo para su sistema operativo en [esta página](https://www.cec.uchile.cl/vpn/). Puede usar también la máquina virtual de la tarea pasada si lo desea.


{{< alert icon="👉" >}}
Si no logran conectarse, prueben con alguno de los siguientes métodos:

* La red interna del DCC comparte el mismo espacio de IPs locales por defecto que Docker, por lo tanto, les recomendamos que deshabiliten el servicio Docker temporalmente (En Ubuntu, lo pueden hacer con `systemctl disable docker` y luego reiniciando)
* Diego Vargas pudo conectarse usando un Túnel SSH:

```bash
sudo ssh -L localhost:443:ecursos.cc5312.xor.cl:443 <user>@anakena.dcc.uchile.cl
```

Con esto, pueden meterse a la página escribiendo `localhost:443`.

{{< /alert >}}



### Ver mi IP en la red del CEC

Luego de conectarse a la VPN, puede revisar la IP asignada en la red del CEC usando el comando `ifconfig` o `ip addr` en Linux (es la dirección que aparece bajo una interfaz llamada `tunX`, con `X` uno o más dígitos), o `ipconfig` en Windows. Esta IP cambiará cada vez que se conecte.

## Navegador para este laboratorio

Los ejemplos se ejecutarán utilizando Firefox, sin embargo, potencialmente todos ellos son replicables usando algún navegador basado en Chromium (Chrome, Edge, Vivaldi, Brave, Opera, etc.)


## XSS

Partiremos revisando una versión insegura de U-Cursos, la cual denominaremos E-Cursos.

Pueden entrar a ella en la siguiente URL, solo visible desde la VPN: http://ecursos.cc5312.xor.cl. Probablemente recibirán un mensaje de advertencia por certificado autofirmado. Como la página es interna, no es posible conseguir certificados TLS gratuitos a través de Let's Encrypt, por lo que deberemos conformarnos con esto. 

Al contrario de U-Cursos, E-Cursos no revisa potenciales entradas maliciosas en las publicaciones de Novedades. Un usuario de malas intenciones con permisos para sbir de novedades acaba de publicar una noticia. Sin embargo, la publicación contiene un script que extrae información de los usuarios que visitan la página. Revisaremos en detalle el script durante la auxiliar.

El script de bash utilizado para correr un servidor HTTP simple es el siguiente:

```bash
while true; do
    printf 'HTTP/1.1 200 OK\r\nContent-Type: application/json\r\nAccess-Control-Allow-Origin: *\r\n\r\n{"ok": true}' | netcat -l -w 1 5312;
done
```

(Requiere tener BSD-Netcat instalado)

En caso que usen macOS, pueden probar con esta versión:


```bash
while true; do
    printf 'HTTP/1.1 200 OK\r\nContent-Type: application/json\r\nAccess-Control-Allow-Origin: *\r\n\r\n{"ok": true}' | netcat -l -G 1 -p 5312;
done
```

En caso que no les funcione el método anterior, python3 tiene un módulo para levantar servidores http de forma rápida y simple:

* Crear una carpeta vacía y abrir un terminal dentro de ella. **Esto es importante para evitar exponer archivos de su computador.**
* Ejecutar `python3 -m http.server 5312`. Esto levantará un servidor HTTP en el puerto 5312 que mostrará los archivos de esa carpeta (es decir, ninguno), pero además logueará en el terminal las consultas realizadas (con sus parámetros GET). La funcionalidad del terminal les sirve tanto para el ejemplo de la auxiliar como para el tarea.

En Windows, también pueden intentar con el primer script a través de [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10), o usando la máquina virtual.


A cada consulta realizada, este pequeño servidor contestará un JSON pequeño, además de setear la cabecera de la consulta de forma que no sea rechazada por CORS de parte del navegador. Es importante comentar que este código requiere tener instalada la versión BSD de Netcat.

### Modificación local del DOM

Para poder probar el script ustedes mismos, es necesario que el valor del campo `hidden` en el que está la IP de destino de las cookies y contraseña sea igual a la IP de sus computadores, de tal modo que las consultas del script inyectado viajen a nuestros equipos. Si bien no pueden cambiar ese valor en el servidor, pueden modificar el DOM de la página web local de forma bastante simple. En el video de la clase se mostrará en detalle como usar el inspector de DOM del navegador y como hacer esta modificación.


### Revision y modificación de cookies

En este ejemplo vamos a ver cómo se exploran y modifican las cookies en el navegador. Para ello, les recomendamos revisar el video de esta auxiliar.


## Inyección SQL Ciega

Por último, veremos un pequeño ejemplo de juguete que nos enseñará a implementar una inyección SQL ciega en [PostgreSQL](https://www.postgresql.org/). Pueden revisarlo [acá](https://ecursos.cc5312.xor.cl/query.php) si sus computadores se encuentran conectados a la VPN.

En el caso de Postgres, existe la función `pg_sleep(t)`, con `t` igual al tiempo en segundos en que se detiene la consulta. Esta función se puede utilizar en conjunto con el operador condicional `CASE`, de forma que si se encuentra una coincidencia en una consulta SQL, ésta se quede dormida una cantidad perceptible de segundos, y si no, termine inmediatamente.

El ejemplo de juguete cuenta con una tabla de nombre `users`, la cual define tres columnas: `id`, `name` y `pin`. Se nos informa que el `id` = 0 es el usuario administrador, y queremos obtener su PIN (numérico de 4 dígitos) usando inyecciones SQL. El formulario entregado es vulnerable a inyecciones, sin embargo no muestra en ninguna parte el resultado de la consulta inyectada, ya que ésta corresponde a una actualización y no a una selección. En estos casos, las inyecciones SQL ciegas son bastante útiles.

Usaremos el siguiente código en la consulta:

```sql
SELECT CASE when (SELECT 1 FROM users WHERE id = 0 and pin like 'X%') = 1 then pg_sleep(N) else pg_sleep(0) end
```

cambiaremos X por cada uno de los posibles valores por los que puede comenzar el PIN (de 0 a 9). Al mismo tiempo, colocaremos como tiempo `N` un valor aproximadamente igual a 5 veces el tiempo promedio que demora la página en cargar, de forma que la demora sea notoria. En el momento en que la consulta se demore harto, es bastante probable que hayamos encontrado un acierto, por lo que el PIN del usuario con ID 0 parte con ese dígito. La búsqueda continúa manteniendo los dígitos encontrados y cambiando el valor de los nuevos, hasta completar el largo total del valor.

## Instalar OpenVPN en la máquina virtual

* Descarguen el archivo [.ovpn](https://www.cec.uchile.cl/download/OPENVPN/CEC-fcfm.ovpn) de la página del CEC

* Ejecuten estos comandos en la terminal

```bash
sudo apt install openvpn
cd ~/Descargas # si lo guardaron en Descargas
sudo openvpn --config CEC-fcfm.ovpn
```

* Mantengan la terminal abierta para mantenerse conectados a la VPN.

## Miniejercicios recomendados para antes del laboratorio

* Desarrollen un script en Python que realice esta acción de forma automática. Para ello, le recomendamos usar la librería [requests](https://requests.readthedocs.io/en/latest/). La URL de consulta debe ser la misma que la definida en el `action` del form vulnerable, y se debe usar el verbo `POST` (para más información sobre cómo mandar requests POST, revisar [esta parte de la documentación](https://requests.readthedocs.io/en/latest/user/quickstart/#more-complicated-post-requests)).

* Prueben consiguiendo más información de la base de datos entregada: Use [esta referencia](http://pentestmonkey.net/cheat-sheet/sql-injection/postgres-sql-injection-cheat-sheet) para conseguir ideas. No se preocupen si rompen algo, pero avísenme para restaurarlo.
