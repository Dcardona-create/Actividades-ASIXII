# Actividad 2 — Resolución DNS

## Índice

1. [Introducción y objetivos](#1-introducción-y-objetivos)
2. [Comparativa de servidores DNS](#2-comparativa-de-servidores-dns)
3. [Investigación de dominios y autoridades DNS](#3-investigación-de-dominios-y-autoridades-dns)
   - [Consulta WHOIS del dominio ifp.es](#31-consulta-whois-del-dominio-ifpes)
   - [Información de dominios en IANA](#32-información-de-dominios-en-iana)
4. [Configuración local del servicio DNS](#4-configuración-local-del-servicio-dns)
   - [Cambio del servidor DNS en Linux](#41-cambio-del-servidor-dns-en-linux)
   - [Gestión de la caché DNS con NSCD](#42-gestión-de-la-caché-dns-con-nscd)
5. [Resolución de nombres con dig](#5-resolución-de-nombres-con-dig)
   - [Consulta de registro A](#51-consulta-de-registro-a)
   - [Consulta de registros MX y NS](#52-consulta-de-registros-mx-y-ns)
   - [Consulta del TTL](#53-consulta-del-ttl)
   - [Trazado completo de resolución DNS](#54-trazado-completo-de-resolución-dns)
6. [Análisis de tráfico DNS con Wireshark](#6-análisis-de-tráfico-dns-con-wireshark)
   - [Consulta y respuesta DNS](#61-consulta-y-respuesta-dns)
   - [Puertos UDP utilizados](#62-puertos-udp-utilizados)
   - [ID de transacción y flags](#63-id-de-transacción-y-flags)
   - [Registros MX en la respuesta](#64-registros-mx-en-la-respuesta)
7. [Conclusiones](#7-conclusiones)
8. [Capturas](#8-capturas)

## 1. Introducción y objetivos

Esta actividad estudia el funcionamiento de la resolución de nombres mediante DNS. Se realizan consultas sobre dominios, se analiza la configuración local del resolutor, se revisa el uso de la caché DNS y se inspecciona el tráfico generado con Wireshark.

Los objetivos son identificar las entidades responsables de los dominios, consultar registros DNS con `dig`, comprender el proceso de resolución y analizar los campos principales de los paquetes DNS.

## 2. Comparativa de servidores DNS

Se realiza una comparativa de rendimiento entre distintos servidores de nombres para observar sus tiempos de respuesta.

![Comparativa de servidores DNS](evidencias/01-benchmark-nameservers.jpg)

## 3. Investigación de dominios y autoridades DNS

### 3.1 Consulta WHOIS del dominio ifp.es

La consulta WHOIS permite consultar la información de registro asociada a un dominio, como la entidad responsable y otros datos administrativos disponibles.

![Consulta WHOIS del dominio ifp.es](evidencias/02-whois-ifp-es.jpg)

### 3.2 Información de dominios en IANA

IANA mantiene información de referencia sobre los dominios de nivel superior. A continuación se muestran las consultas realizadas para los dominios `.es`, `.cat` y `.edu`.

#### Dominio .es

![Información IANA del dominio .es](evidencias/03-iana-es.png)

#### Dominio .cat

![Información IANA del dominio .cat](evidencias/04-iana-cat.jpg)

#### Dominio .edu

![Información IANA del dominio .edu](evidencias/05-iana-edu.jpg)

## 4. Configuración local del servicio DNS

### 4.1 Cambio del servidor DNS en Linux

Se modifica la configuración del equipo Linux para utilizar el servidor DNS indicado en la actividad.

![Cambio del servidor DNS en Linux](evidencias/06-cambio-dns-linux.jpg)

### 4.2 Gestión de la caché DNS con NSCD

NSCD permite almacenar en caché información de servicios de nombres. Se comprueba el estado de la caché y se realiza su vaciado para evitar que consultas anteriores afecten a nuevas pruebas.

#### Estado de la caché DNS

![Caché DNS con NSCD](evidencias/07-cache-dns-nscd.jpg)

#### Vaciado de la caché DNS

![Vaciado de la caché DNS](evidencias/08-vaciado-cache-dns.jpg)

## 5. Resolución de nombres con dig

### 5.1 Consulta de registro A

Se realiza una consulta de tipo `A` para obtener la dirección IPv4 asociada al dominio analizado.

```bash
dig A aliexpress.com
```

![Consulta de registro A](evidencias/09-dig-a-aliexpress.jpg)

### 5.2 Consulta de registros MX y NS

Se consultan los registros de correo y los servidores de nombres autoritativos asociados al dominio.

```bash
dig +short MX aliexpress.com
dig +short NS aliexpress.com
```

![Consulta de registros MX y NS](evidencias/10-dig-short-mx-ns-aliexpress.jpg)

### 5.3 Consulta del TTL

El TTL indica durante cuánto tiempo puede conservarse una respuesta DNS en caché antes de volver a consultarse.

```bash
dig aliexpress.com
```

![Consulta del TTL](evidencias/11-dig-ttl-aliexpress.jpg)

### 5.4 Trazado completo de resolución DNS

La opción `+trace` permite visualizar el recorrido de la resolución, desde los servidores raíz hasta el servidor autoritativo del dominio.

```bash
dig +trace aliexpress.com
```

![Trazado completo de resolución DNS](evidencias/12-dig-trace-aliexpress.jpg)

## 6. Análisis de tráfico DNS con Wireshark

### 6.1 Consulta y respuesta DNS

Se captura una consulta DNS de tipo MX y la respuesta proporcionada por el servidor DNS.

![Consulta y respuesta DNS](evidencias/13-wireshark-query-response-mx.png)

### 6.2 Puertos UDP utilizados

El tráfico DNS se realiza normalmente mediante UDP. En la captura se identifica el puerto efímero de origen del cliente y el puerto 53 de destino del servidor DNS.

![Puertos UDP utilizados](evidencias/14-wireshark-udp-ports.png)

### 6.3 ID de transacción y flags

El ID de transacción permite asociar una respuesta con su consulta. Los flags indican características y estado del mensaje DNS, como si se trata de una respuesta o si la resolución fue correcta.

![ID de transacción y flags DNS](evidencias/15-wireshark-transaction-id-flags.png)

### 6.4 Registros MX en la respuesta

La sección de respuestas contiene los registros MX devueltos para el dominio consultado, que indican los servidores responsables de recibir correo electrónico.

![Registros MX en la respuesta](evidencias/16-wireshark-mx-answers.png)

## 7. Conclusiones

La actividad permite comprobar cómo se resuelven los nombres de dominio mediante DNS, desde la consulta de registros y el uso de caché local hasta el análisis de los paquetes intercambiados entre cliente y servidor. El uso de `dig` y Wireshark facilita la interpretación tanto de las respuestas DNS como de los campos que intervienen en el protocolo.

## 8. Capturas

Todas las capturas se guardan en `evidencias/`.

| Nº | Descripción | Captura |
|---:|---|---|
| 1 | Comparativa de servidores DNS | [Ver captura](evidencias/01-benchmark-nameservers.jpg) |
| 2 | Consulta WHOIS del dominio ifp.es | [Ver captura](evidencias/02-whois-ifp-es.jpg) |
| 3 | Información de IANA sobre .es | [Ver captura](evidencias/03-iana-es.png) |
| 4 | Información de IANA sobre .cat | [Ver captura](evidencias/04-iana-cat.jpg) |
| 5 | Información de IANA sobre .edu | [Ver captura](evidencias/05-iana-edu.jpg) |
| 6 | Cambio del servidor DNS en Linux | [Ver captura](evidencias/06-cambio-dns-linux.jpg) |
| 7 | Estado de la caché DNS con NSCD | [Ver captura](evidencias/07-cache-dns-nscd.jpg) |
| 8 | Vaciado de la caché DNS | [Ver captura](evidencias/08-vaciado-cache-dns.jpg) |
| 9 | Consulta del registro A de aliexpress.com | [Ver captura](evidencias/09-dig-a-aliexpress.jpg) |
| 10 | Consulta de registros MX y NS de aliexpress.com | [Ver captura](evidencias/10-dig-short-mx-ns-aliexpress.jpg) |
| 11 | Consulta del TTL de aliexpress.com | [Ver captura](evidencias/11-dig-ttl-aliexpress.jpg) |
| 12 | Trazado de resolución DNS de aliexpress.com | [Ver captura](evidencias/12-dig-trace-aliexpress.jpg) |
| 13 | Consulta y respuesta MX en Wireshark | [Ver captura](evidencias/13-wireshark-query-response-mx.png) |
| 14 | Puertos UDP en Wireshark | [Ver captura](evidencias/14-wireshark-udp-ports.png) |
| 15 | ID de transacción y flags DNS en Wireshark | [Ver captura](evidencias/15-wireshark-transaction-id-flags.png) |
| 16 | Registros MX de la respuesta en Wireshark | [Ver captura](evidencias/16-wireshark-mx-answers.png) |
