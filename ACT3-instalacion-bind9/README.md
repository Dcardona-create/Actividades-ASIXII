# ACT3 — Instalación y configuración de BIND9

> Práctica de despliegue de un servidor DNS autoritativo con BIND9, configuración de zonas directa e inversa y validación de la resolución local.

## Índice

- [Objetivo](#objetivo)
- [Introducción teórica](#introducción-teórica)
- [Escenario y arquitectura](#escenario-y-arquitectura)
- [Preparación del entorno](#1-preparación-del-entorno)
- [Instalación y servicio BIND9](#2-instalación-y-servicio-bind9)
- [Configuración de zonas](#3-configuración-de-zonas)
- [Validación DNS](#4-validación-dns)
- [Análisis de incidencias técnicas](#análisis-de-incidencias-técnicas)
- [Operación y mantenimiento](#operación-y-mantenimiento)
- [Conclusiones](#conclusiones)
- [Capturas](#capturas)

---

## Objetivo

El objetivo de esta actividad es instalar y dejar operativo **BIND9** como servidor DNS en una máquina virtual GNU/Linux. La configuración incluye una zona de búsqueda directa —nombre a dirección IP— y una zona inversa —dirección IP a nombre—, comprobando finalmente que ambas consultas son respondidas por el servicio local.

## Introducción teórica

### Función del DNS

El Domain Name System (DNS) es el servicio encargado de traducir nombres de dominio legibles, como `www.ejemplo.com`, a direcciones IP que utilizan los equipos para comunicarse. Es esencial para el funcionamiento de Internet porque permite acceder a servidores y servicios mediante nombres, sin necesidad de conocer o memorizar sus direcciones numéricas.

### Jerarquía del DNS

DNS se organiza de forma jerárquica. En la parte superior se encuentran los servidores raíz, que derivan las consultas hacia los servidores de dominio de nivel superior o TLD, como `.com`, `.es` o `.org`. Los servidores TLD indican cuáles son los servidores autoritativos de cada dominio. Estos servidores autoritativos almacenan los registros definitivos de una zona DNS. Por último, el resolutor recursivo realiza las consultas necesarias y devuelve la respuesta al cliente.

### Consultas iterativas y recursivas

En una consulta recursiva, el cliente solicita una respuesta final y el servidor DNS se encarga de consultar otros servidores si no dispone de la información solicitada. En una consulta iterativa, cada servidor devuelve la mejor información que posee, normalmente una referencia al siguiente servidor que debe consultarse para continuar la resolución.

### Registros de recursos DNS

Los registros de recursos, también llamados RR, almacenan la información de una zona DNS. Los principales son:

- `SOA`: identifica la autoridad de la zona e incluye parámetros como el número de serie.
- `NS`: indica los servidores de nombres autoritativos de la zona.
- `A`: asocia un nombre de host con una dirección IPv4.
- `AAAA`: asocia un nombre de host con una dirección IPv6.
- `CNAME`: crea un alias hacia otro nombre de dominio.
- `MX`: define los servidores responsables de recibir correo electrónico para un dominio.
- `PTR`: permite la resolución inversa, asociando una dirección IP a un nombre de dominio.

### Seguridad en DNS

El servicio DNS puede verse afectado por ataques como la suplantación de respuestas DNS, la contaminación de caché o *cache poisoning*, transferencias de zona no autorizadas y ataques de amplificación DDoS. Para reducir estos riesgos se pueden aplicar medidas como DNSSEC, la limitación de recursión, restricciones de transferencia de zona mediante listas de control de acceso, la actualización periódica de BIND9 y la monitorización de registros y eventos del servicio.

## Escenario y arquitectura

La máquina virtual actúa como servidor DNS. BIND9 carga la configuración global y las declaraciones de zona, consulta los archivos de zona y atiende las peticiones DNS en el puerto 53.

```text
┌─────────────────────────┐        consultas DNS        ┌─────────────────────────────┐
│ Cliente / terminal      │ ─────────────────────────▶ │ Servidor BIND9              │
│ dig · nslookup · host   │ ◀───────────────────────── │ zona directa + zona inversa │
└─────────────────────────┘        respuestas DNS       └──────────────┬──────────────┘
                                                                        │
                                                       ┌────────────────┴────────────────┐
                                                       │ /etc/bind/named.conf.local       │
                                                       │ /etc/bind/zones/db.deivid.test   │
                                                       │ /etc/bind/zones/db.6.168.192     │
                                                       └─────────────────────────────────┘
```

| Elemento | Función en la práctica |
|---|---|
| BIND9 | Servicio DNS que procesa las consultas. |
| Zona directa `deivid.test` | Asocia nombres de host con registros `A`. |
| Zona inversa `6.168.192.in-addr.arpa` | Asocia direcciones IP con registros `PTR`. |
| `dig` / `nslookup` | Herramientas usadas para validar la resolución. |

---

## 1. Preparación del entorno

Antes de configurar el servicio se revisa la máquina virtual y su conectividad de red. La interfaz `enp0s3` proporciona salida a Internet mediante NAT, mientras que la interfaz `enp0s8` pertenece a la red local del laboratorio y se utiliza para el servidor DNS autoritativo.

### Características de hardware y software

| Elemento | Configuración utilizada |
|---|---|
| Hostname estático | `Deivid` |
| Sistema operativo | Debian GNU/Linux 13 (Trixie) |
| Kernel | Linux `6.12.111+deb13-amd64` |
| Arquitectura | `x86-64` |
| Entorno de virtualización | Oracle VirtualBox |
| Interfaz NAT | `enp0s3` |
| IP NAT | `10.0.2.15/24` (asignación DHCP) |
| Puerta de enlace predeterminada | `10.0.2.2` |
| Interfaz de laboratorio | `enp0s8` |
| IP del servidor DNS | `192.168.6.100/24` |
| Red local del laboratorio | `192.168.6.0/24` |

### Entorno de la máquina virtual y configuración de red

![Entorno de la máquina virtual y configuración de red](evidencias/03-entorno-vm-y-red.png)


### Pasos

1. Iniciar la máquina virtual y comprobar que las interfaces de red tienen una dirección IP válida.
2. Verificar la conectividad mediante la puerta de enlace `10.0.2.2` y, si procede, con una dirección externa.
3. Confirmar que la interfaz `enp0s8` mantiene la dirección `192.168.6.100/24`, utilizada por las zonas DNS locales.

---

## 2. Instalación y servicio BIND9

BIND9 se instala desde los repositorios de la distribución. Tras la instalación, se comprueba que el demonio queda habilitado y en ejecución antes de declarar las zonas propias.

```bash
sudo apt update
sudo apt install bind9 bind9utils dnsutils
sudo systemctl enable --now bind9
sudo systemctl status bind9
```

### Resolución externa y estado del servicio BIND9

![Resolución externa y estado del servicio BIND9](evidencias/02-resolucion-externa-y-estado-bind9.png)


### Validación esperada

- `systemctl status bind9` muestra el servicio como `active (running)`.
- El sistema mantiene conectividad para consultas DNS externas a través de `enp0s3`.
- El puerto DNS queda disponible para las consultas configuradas en el host.

---

## 3. Configuración de zonas

La configuración se centraliza en `/etc/bind/`. El archivo `/etc/bind/named.conf.local` declara las zonas que administra el servidor y cada zona apunta a su correspondiente archivo de registros en `/etc/bind/zones/`.

```conf
zone "deivid.test" {
    type master;
    file "/etc/bind/zones/db.deivid.test";
};

zone "6.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.6.168.192";
};
```

### Configuración de BIND9 y archivos de zona

![Configuración de BIND9 y archivos de zona](evidencias/04-configuracion-bind9-y-zonas.png)


### Zona directa

La zona directa `deivid.test` asocia el servidor DNS `ns1.deivid.test.` con la dirección IPv4 `192.168.6.100`.

```dns
$TTL 86400
@   IN  SOA ns1.deivid.test. hostmaster.deivid.test. (
        1       ; Serial
        86400   ; Refresh
        7200    ; Retry
        3600000 ; Expire
        86400   ; Negative Cache TTL
)

@       IN  NS  ns1.deivid.test.
ns1     IN  A   192.168.6.100
```

### Zona inversa

La zona inversa `6.168.192.in-addr.arpa` permite recuperar el nombre de host a partir de la dirección IP `192.168.6.100`.

```dns
$TTL 86400
@   IN  SOA ns1.deivid.test. hostmaster.deivid.test. (
        1       ; Serial
        86400   ; Refresh
        7200    ; Retry
        3600000 ; Expire
        86400   ; Negative Cache TTL
)

@       IN  NS  ns1.deivid.test.
100     IN  PTR ns1.deivid.test.
```

### Comprobación sintáctica

Antes de reiniciar el servicio conviene validar archivos y configuración:

```bash
sudo named-checkconf
sudo named-checkzone deivid.test /etc/bind/zones/db.deivid.test
sudo named-checkzone 6.168.192.in-addr.arpa /etc/bind/zones/db.6.168.192
sudo systemctl restart bind9
```

---

## 4. Validación DNS

Una vez cargadas las zonas, las consultas directas e inversas confirman que BIND9 responde como servidor autoritativo para los registros definidos.

### Consulta directa

```bash
dig @127.0.0.1 ns1.deivid.test A
```

La consulta devuelve el estado `NOERROR`, una respuesta autoritativa mediante el flag `aa` y el siguiente registro:

```text
ns1.deivid.test. 86400 IN A 192.168.6.100
```

### Consulta inversa

```bash
dig @127.0.0.1 -x 192.168.6.100
```

La consulta devuelve el estado `NOERROR`, una respuesta autoritativa mediante el flag `aa` y el siguiente registro:

```text
100.6.168.192.in-addr.arpa. 86400 IN PTR ns1.deivid.test.
```

### Verificación de la resolución directa e inversa

![Verificación de la resolución directa e inversa](evidencias/01-verificacion-zonas-directa-inversa.png)


### Criterios de éxito

| Prueba | Resultado observado |
|---|---|
| Consulta directa | `ns1.deivid.test.` devuelve el registro `A` con la IP `192.168.6.100`. |
| Consulta inversa | `192.168.6.100` devuelve el registro `PTR` `ns1.deivid.test.` |
| Estado del servicio | BIND9 activo y sin errores de sintaxis o carga de zonas. |

---

## Análisis de incidencias técnicas

Durante la configuración de BIND9, los errores más relevantes pueden aparecer al declarar una zona, definir sus registros o recargar el servicio. Antes de aplicar cambios se debe comprobar la sintaxis de la configuración global y de cada archivo de zona:

```bash
sudo named-checkconf
sudo named-checkzone deivid.test /etc/bind/zones/db.deivid.test
sudo named-checkzone 6.168.192.in-addr.arpa /etc/bind/zones/db.6.168.192
```

Una incidencia habitual es modificar un archivo de zona sin incrementar el serial del registro `SOA`. Tras cada cambio se debe aumentar ese valor y recargar el servicio para que BIND9 utilice la versión actualizada de la zona.

Si el servicio no inicia o una zona no se carga, se debe revisar el estado y los eventos recientes:

```bash
sudo systemctl status bind9
sudo journalctl -u bind9 -n 50 --no-pager
```

La resolución de incidencias consiste en corregir el error detectado, volver a ejecutar las comprobaciones sintácticas y recargar BIND9. Finalmente, se deben repetir las consultas `dig` directa e inversa para verificar que la respuesta sigue siendo autoritativa y no presenta errores.

---

## Operación y mantenimiento

### Después de modificar una zona

1. Incrementar el **serial** del registro SOA.
2. Revisar la sintaxis con `named-checkconf` y `named-checkzone`.
3. Recargar BIND9 sin interrumpir el servicio:

```bash
sudo rndc reload
```

4. Repetir las consultas `dig` para confirmar que los cambios se han cargado.

### Diagnóstico rápido

```bash
sudo systemctl status bind9
sudo journalctl -u bind9 -n 50 --no-pager
sudo ss -lntup | grep ':53'
dig @127.0.0.1 deivid.test SOA
```

- Si BIND9 no inicia, revisar primero la salida de `named-checkconf`.
- Si una zona no carga, comprobar el nombre del archivo, los permisos y el serial SOA.
- Si una consulta devuelve `NXDOMAIN`, verificar el registro solicitado y que se consulta el servidor DNS correcto.

---

## Conclusiones

La práctica ha permitido desplegar un servidor DNS autoritativo con BIND9 en Debian GNU/Linux 13 y configurar una zona directa junto con su zona inversa para la red `192.168.6.0/24`.

Las consultas realizadas con `dig` han confirmado la resolución directa de `ns1.deivid.test.` a `192.168.6.100` y la resolución inversa de `192.168.6.100` a `ns1.deivid.test.`. Ambas respuestas devolvieron el estado `NOERROR` y el flag `aa`, confirmando que el servidor BIND9 responde de forma autoritativa para las zonas configuradas.

También se ha comprobado la importancia de validar la configuración con `named-checkconf` y `named-checkzone`, incrementar el serial SOA después de cada modificación y utilizar los registros del servicio para diagnosticar incidencias antes de recargar BIND9.

---

## Capturas

Todas las capturas se guardan en `evidencias/`.

| Descripción | Captura |
|---|---|
| Verificación de la resolución directa e inversa | [Ver captura](evidencias/01-verificacion-zonas-directa-inversa.png) |
| Resolución externa y estado del servicio BIND9 | [Ver captura](evidencias/02-resolucion-externa-y-estado-bind9.png) |
| Entorno de la máquina virtual y configuración de red | [Ver captura](evidencias/03-entorno-vm-y-red.png) |
| Configuración de BIND9 y archivos de zona | [Ver captura](evidencias/04-configuracion-bind9-y-zonas.png) |
