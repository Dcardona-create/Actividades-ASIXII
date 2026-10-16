# Actividad 1 — DNS Tunneling

## Índice

1. [Introducción y objetivos](#1-introducción-y-objetivos)
2. [Funcionamiento del DNS Tunneling](#2-funcionamiento-del-dns-tunneling)
3. [Implementación del cliente DNS](#3-implementación-del-cliente-dns)
4. [Implementación del servidor DNS](#4-implementación-del-servidor-dns)
5. [Ejecución de la petición DNS](#5-ejecución-de-la-petición-dns)
6. [Análisis de la comunicación con Wireshark](#6-análisis-de-la-comunicación-con-wireshark)
7. [Conclusiones](#7-conclusiones)
8. [Capturas](#8-capturas)

## 1. Introducción y objetivos

Esta actividad muestra una implementación básica de DNS Tunneling mediante un cliente y un servidor desarrollados en Python. El objetivo es comprender cómo una petición DNS puede utilizarse para transportar información entre un cliente y un servidor, así como observar el tráfico generado durante la comunicación.

Los objetivos principales son:

- Implementar un cliente que envíe una petición DNS.
- Implementar un servidor que reciba y procese la petición.
- Comprobar la comunicación entre ambos extremos.
- Analizar los paquetes intercambiados mediante Wireshark.

## 2. Funcionamiento del DNS Tunneling

DNS Tunneling es una técnica que utiliza consultas y respuestas DNS como canal de comunicación. En esta práctica, el cliente genera una petición DNS y el servidor la recibe para procesar la información enviada.

El flujo de trabajo es el siguiente:

1. Se inicia el servidor DNS y queda a la espera de peticiones.
2. El cliente construye y envía una consulta DNS.
3. El servidor recibe la petición y procesa el contenido.
4. Se observa la comunicación desde el cliente, el servidor y Wireshark.

## 3. Implementación del cliente DNS

El cliente se ha desarrollado en Python y se encarga de crear y enviar la petición DNS al servidor.

Archivo utilizado: `dns_client_1_peticion.py`.

```bash
python3 dns_client_1_peticion.py
```

## 4. Implementación del servidor DNS

El servidor se ha desarrollado en Python y permanece a la espera de recibir solicitudes DNS enviadas por el cliente.

Archivo utilizado: `dns_server_1_peticion.py`.

```bash
python3 dns_server_1_peticion.py
```

## 5. Ejecución de la petición DNS

Para realizar la prueba, primero se ejecuta el servidor y, a continuación, el cliente. La siguiente captura muestra el envío de la petición desde el cliente.

### Petición enviada desde el cliente

![Petición enviada desde el cliente](evidencias/cliente_peticion.jpg)

El servidor recibe y procesa la petición DNS enviada por el cliente.

### Petición recibida en el servidor

![Petición recibida en el servidor](evidencias/server.peticion.jpg)

## 6. Análisis de la comunicación con Wireshark

Wireshark permite verificar que la comunicación se ha realizado mediante paquetes DNS. En la captura se puede observar el tráfico generado durante la petición entre el cliente y el servidor.

### Análisis de la comunicación DNS con Wireshark

![Análisis de la comunicación DNS con Wireshark](evidencias/wireshark.jpg)

## 7. Conclusiones

La actividad permite comprobar el funcionamiento básico de una comunicación mediante DNS Tunneling. La implementación cliente-servidor demuestra cómo se puede enviar información dentro de una petición DNS y cómo el servidor puede recibirla y procesarla. El análisis con Wireshark confirma el intercambio de tráfico DNS durante la prueba.

## 8. Capturas

Todas las capturas se guardan en `evidencias/`.

| Nº | Descripción | Captura |
|---:|---|---|
| 1 | Petición enviada desde el cliente | [Ver captura](evidencias/cliente_peticion.jpg) |
| 2 | Petición recibida en el servidor | [Ver captura](evidencias/server.peticion.jpg) |
| 3 | Análisis de la comunicación DNS con Wireshark | [Ver captura](evidencias/wireshark.jpg) |
