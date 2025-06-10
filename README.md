# Enrutamiento_batman

# Red ad-hoc con B.A.T.M.A.N. en Debian – Proyecto de Redes

-Este repositorio documenta el desarrollo, configuración e implementación de una red mesh ad-hoc utilizando el protocolo de enrutamiento **B.A.T.M.A.N.** (Better Approach To Mobile Adhoc Networking) en dispositivos con **Debian Linux**. El objetivo fue establecer una comunicación **multi-hop** entre 4 computadores sin infraestructura central, simulando una red autónoma y auto-organizada.

---

## ¿Qué es B.A.T.M.A.N.?

**B.A.T.M.A.N.** es un protocolo de enrutamiento diseñado para redes *mesh*, donde cada nodo actua como router y colaborador en el reenvío de paquetes. Este protocolo se destaca por su capacidad de **descubrir rutas optimas dinamicamente** sin depender de una tabla centralizada ni cálculos pesados.

En lugar de construir un mapa completo de la red, como lo hace OSPF, B.A.T.M.A.N. trabaja con información local: **cada nodo solo conoce a sus vecinos inmediatos** y elige a cual reenviar un paquete en funcion de la calidad del enlace.

---

## Hardware utilizado

- 4 computadores portátiles
- 3 USB booteables con Debian y 1 microSD con Debian
- Módulos de red Wi-Fi compatibles con modo ad-hoc
- No se utilizo infraestructura de red (sin routers, switches o puntos de acceso)

---

## Topologia del Proyecto

La red se implementó de forma **lineal**, con nodos distribuidos desde el **salon** hasta la **portería** del edificio, cada uno configurado como un nodo autónomo con una IP única. La intención era simular un entorno real donde no hay conectividad directa punto a punto, y se requiere que los mensajes viajen a través de múltiples dispositivos.

---

## Configuracion de IP de cada nodo

---

## Tipo de Red: Ad-Hoc Mesh

La red se configuró como **ad-hoc**, es decir, cada nodo se comunica directamente con otros sin necesidad de infraestructura fija. Cada nodo forma parte de la malla y coopera reenviando mensajes incluso si él no es el origen ni el destino del mensaje.

Esto fue esencial para lograr comunicación desde el primer hasta el último nodo sin conexión directa entre ellos.

---

## Proceso de Configuracioon Tecnica en Debian

### 1. Instalacion de paquetes necesarios

```bash
sudo apt update && sudo apt upgrade
sudo apt install batctl wireless-tools net-tools

```
## 2. Configuracion de red Ad-hoc 

```
sudo ip link set wlan0 down
sudo iwconfig wlan0 mode ad-hoc
sudo iwconfig wlan0 essid mesh-net
sudo iwconfig wlan0 ap 02:12:34:56:78:9A
sudo ip link set wlan0 up

```

## 3. Activar B.A.T.M.A.N.

```
sudo modprobe batman-adv
sudo batctl if add wlan0
sudo ip link set up dev bat0
sudo ip addr add 10.0.0.X/24 dev bat0  # IP única por nodo

```

## 4. Verificacion de protocolo

```
sudo batctl n  # Muestra nodos vecinos conectados
```

## Codigo de Comunicación: Chat en Python con UDP

Para validar la conexión se implementó un pequeño sistema de chat usando sockets UDP en Python. El script permite enviar mensajes de texto entre nodos ingresando la IP destino y el mensaje.

## Repositorio del script:

---

- https://github.com/Juancabarrera26/Parcial_redes/blob/main/chat_batman.py

## Explicacion del codigo:

- Usa la IP BATMAN del nodo local como origen.

- Abre un socket UDP en el puerto 5005.

- Escucha en un hilo aparte todos los mensajes entrantes.

- Permite enviar mensajes a otras IPs BATMAN activas.

```python
import socket
import threading

PORT = 5005
BUFFER_SIZE = 1024

local_ip = input("Tu IP BATMAN (ej. 10.0.0.1): ")
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind((local_ip, PORT))

def listen():
    while True:
        data, addr = sock.recvfrom(BUFFER_SIZE)
        print(f"Mensaje de {addr[0]}: {data.decode()}\n> ", end="")

thread = threading.Thread(target=listen, daemon=True)
thread.start()

print("Chat BATMAN iniciado. Escribe 'salir' para terminar.")
while True:
    dest_ip = input("> IP destino: ")
    message = input("> Mensaje: ")
    if message.lower() == "salir":
        break
    sock.sendto(message.encode(), (dest_ip, PORT))
```
## Evidencia del Funcionamiento

A continuacion se agregaran las capturas de pantalla de cada nodo, mostrando:

Terminal con ejecución del script

- IP de origen

- IP de destino

- Mensaje "Hola" transmitido

---

## Conclusiones

- El protocolo B.A.T.M.A.N. permitio una comunicacion efectiva multi-hop sin depender de routers o acceso a internet.

- Los mensajes se reenviaron dinámicamente segun la calidad del enlace entre nodos vecinos.

- Se evidencio que cada computador (nodo) puede actuar como un intermediario en la entrega de informacion.

- La red es escalable y resiliente ante fallos individuales.
