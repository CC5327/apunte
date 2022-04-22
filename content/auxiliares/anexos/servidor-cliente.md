---
title: "Cliente y Servidor Eco"
date: 2022-03-24T10:00:00-03:00
draft: false
menu:
  auxiliares:
    parent: "anexos"
---

Un *Echo Server* es un tipo de servidor que envía la misma información que recibe, de la misma manera, un *Echo Client* es un tipo de cliente que está preparado para recibir exactamente lo mismo que envía y luego cerrar la conexión.

A continuación se muestra una implementación en *Python* de un Echo Server en protocolo TCP.

``` python
import socket

##################### SERVER SETTINGS ######################
TCP_IP = 'localhost'
TCP_PORT = 5005
BUFFER_SIZE = 16
MAX_CLIENT = 1 # N° maximo de clientes paralelos
############################################################

# 'with' para no tener que usar sock.close()
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    server_address = (TCP_IP, TCP_PORT)
    s.bind(server_address)
    s.listen(MAX_CLIENT)
    
    print(f'Server started on {TCP_IP} port {TCP_PORT}')
    
    while True:
        # Esperar por un cliente
        print('waiting for a connection')
        conn, addr = s.accept()

        with conn: # Para cada cliente se realiza lo siguiente
            print('New client! connection address:', addr)
            while True:
                try:
                    data = conn.recv(BUFFER_SIZE)
                    print('received {!r}'.format(data))
                    
                    if data:
                        print('sending data back to the client')
                        conn.sendall(data)
                    else:
                        print('no data from', addr)
                        break
                    
                except:
                    print("Can't receive data from client")
                    break

        print('Connection closed')
```

La siguiente es una implementación de un Echo Client en *Python* usando el protocolo TCP.

``` python
import socket

####################### SERVER INFO ########################
TCP_IP = 'localhost'
TCP_PORT = 5005
############################################################
BUFFER_SIZE = 16

# 'with' para no tener que usar sock.close()
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:

    # Conectarse al puerto e IP donde el servidor se encuentra escuchando
    server_address = (TCP_IP, TCP_PORT)
    s.connect(server_address)
    print(f'Connected to server on {TCP_IP} port {TCP_PORT}')
    
    try:

        # Enviar el mensaje
        message = b'This is the message.  It will be repeated.'
        print('sending {!r}'.format(message))
        s.sendall(message)

        # Esperar la respuesta
        amount_received = 0
        amount_expected = len(message)

        while amount_received < amount_expected:
            data = s.recv(BUFFER_SIZE)
            amount_received += len(data)
            print('received {!r}'.format(data))
    
    except:
        print("Error sending")
```

**Importante**: estas implementaciones sirven para conectarse dentro de la misma máquina, para conectar dos maquinas diferentes en una misma red es necesario levantar el servidor usando la IP privada de la máquina.
