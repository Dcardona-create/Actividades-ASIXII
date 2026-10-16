# ACT4 — Configuración de un servidor DNS secundario con BIND9

> Práctica de despliegue de un servidor DNS secundario con BIND9, transferencia de zonas desde un servidor primario y validación de la redundancia del servicio DNS.

## Índice

- [Objetivo](#objetivo)
- [Introducción teórica](#introducción-teórica)
- [Escenario y arquitectura](#escenario-y-arquitectura)
- [Plan de direccionamiento](#plan-de-direccionamiento)
- [Preparación del servidor secundario](#1-preparación-del-servidor-secundario)
- [Configuración del servidor primario](#2-configuración-del-servidor-primario)
- [Configuración del servidor secundario](#3-configuración-del-servidor-secundario)
- [Transferencia y sincronización de zonas](#4-transferencia-y-sincronización-de-zonas)
- [Validación del servicio](#5-validación-del-servicio)
- [Análisis de incidencias técnicas](#análisis-de-incidencias-técnicas)
- [Operación y mantenimiento](#operación-y-mantenimiento)
- [Conclusiones](#conclusiones)
- [Evidencias](#evidencias)

---

## Objetivo

El objetivo de esta actividad es desplegar un servidor DNS secundario con BIND9 para proporcionar redundancia al servidor DNS primario configurado en la actividad anterior. El secundario recibirá una copia de las zonas directa e inversa mediante transferencia de zona y podrá responder de forma autoritativa a las consultas DNS.

## Introducción teórica

Un servidor DNS secundario, también denominado esclavo, mantiene una copia de las zonas administradas por un servidor primario. Su función principal es aportar disponibilidad y tolerancia a fallos: si el primario no está disponible, los clientes pueden consultar el secundario.

Las transferencias de zona permiten copiar los registros DNS desde el primario al secundario. Una transferencia completa se denomina AXFR y una transferencia incremental se denomina IXFR. El valor serial del registro SOA permite al secundario detectar si la zona del primario ha sido actualizada.

En BIND9, el primario declara una zona con `type master` y autoriza transferencias hacia el secundario mediante `allow-transfer`. El secundario declara esa misma zona con `type slave`, indica la IP del primario en la directiva `masters` y guarda localmente la copia recibida en un directorio escribible por el usuario `bind`.

## Escenario y arquitectura

El servidor primario ya configurado administra las zonas `deivid.test` y `6.168.192.in-addr.arpa`. El servidor secundario se conectará a la misma red de laboratorio y obtendrá las zonas desde el primario.

```text
                     Red de laboratorio: 192.168.6.0/24

┌────────────────────────────────┐       Transferencia de zona       ┌────────────────────────────────┐
│ DNS primario                   │ ───────────────────────────────▶ │ DNS secundario                 │
│ ns1.deivid.test.               │            AXFR / IXFR            │ ns2.deivid.test.               │
│ IP: 192.168.6.100              │ ◀─────────────────────────────── │ IP: [pendiente de confirmar]   │
│ BIND9: type master             │        consultas y notificación   │ BIND9: type slave              │
└────────────────────────────────┘                                   └────────────────────────────────┘
          │                                                                      │
          └──────────── Zona directa e inversa autoritativas ───────────────────┘
```

## Plan de direccionamiento

| Elemento | Valor |
|---|---|
| Red de laboratorio | `192.168.6.0/24` |
| DNS primario | `ns1.deivid.test.` |
| IP del primario | `192.168.6.100` |
| DNS secundario | `ns2.deivid.test.` |
| IP del secundario | Pendiente de confirmar |
| Zona directa | `deivid.test` |
| Zona inversa | `6.168.192.in-addr.arpa` |

> La IP del secundario sigue pendiente de confirmar en esta documentación. Los marcadores `IP_DEL_SECUNDARIO` no deben ejecutarse literalmente ni sustituirse por una IP supuesta.

## 1. Preparación del servidor secundario

En una segunda máquina virtual Debian se debe comprobar el hostname, la interfaz de red y la IP asignada en la red `192.168.6.0/24`.

```bash
hostnamectl
ip -4 addr show
ip route
```

### Evidencia 1 — Hostname del servidor DNS2

Captura disponible para documentar la identificación del servidor secundario. La dirección IP se mantendrá pendiente en el texto hasta contrastarla con la salida real del sistema.

![Evidencia 1: Hostname del servidor DNS2](evidencias/01-hostname-dns2.png)

Después se instala BIND9:

```bash
sudo apt update
sudo apt install bind9 bind9utils dnsutils
sudo systemctl enable --now bind9
sudo systemctl status bind9
```

## 2. Configuración del servidor primario

El servidor primario debe autorizar al secundario para transferir las zonas. La IP de ejemplo siguiente deberá sustituirse por la IP real de `ns2.deivid.test`.

Archivo `/etc/bind/named.conf.local` del primario:

```conf
zone "deivid.test" {
    type master;
    file "/etc/bind/zones/db.deivid.test";
    allow-transfer { IP_DEL_SECUNDARIO; };
    also-notify { IP_DEL_SECUNDARIO; };
};

zone "6.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.6.168.192";
    allow-transfer { IP_DEL_SECUNDARIO; };
    also-notify { IP_DEL_SECUNDARIO; };
};
```

### Evidencia 3 — Autorización de transferencia en el maestro

Captura de la configuración de autorización de transferencias del servidor primario. Se conserva el número 3 para que coincida con el nombre del archivo existente.

![Evidencia 3: Autorización de transferencia en el maestro](evidencias/03-autorizacion-transferencia-maestro.png)

En la zona directa se añadirá el segundo servidor de nombres:

```dns
@       IN  NS  ns1.deivid.test.
@       IN  NS  ns2.deivid.test.
ns1     IN  A   192.168.6.100
ns2     IN  A   IP_DEL_SECUNDARIO
```

En la zona inversa se añadirá el PTR correspondiente:

```dns
ULTIMO_OCTETO_SECUNDARIO  IN  PTR ns2.deivid.test.
```

Después de modificar las zonas, se debe incrementar el serial SOA, validar los archivos y recargar BIND9:

```bash
sudo named-checkconf
sudo named-checkzone deivid.test /etc/bind/zones/db.deivid.test
sudo named-checkzone 6.168.192.in-addr.arpa /etc/bind/zones/db.6.168.192
sudo rndc reload
```

## 3. Configuración del servidor secundario

El servidor secundario no crea manualmente los archivos de zona: los recibe desde el primario. Primero se prepara un directorio donde BIND9 pueda almacenar las transferencias:

```bash
sudo mkdir -p /var/cache/bind/slaves
sudo chown bind:bind /var/cache/bind/slaves
```

Archivo `/etc/bind/named.conf.local` del secundario:

```conf
zone "deivid.test" {
    type slave;
    masters { 192.168.6.100; };
    file "/var/cache/bind/slaves/db.deivid.test";
};

zone "6.168.192.in-addr.arpa" {
    type slave;
    masters { 192.168.6.100; };
    file "/var/cache/bind/slaves/db.6.168.192";
};
```

### Evidencia 2 — Configuración del servidor esclavo

Captura de la configuración del servidor secundario. Se conserva el número 2 para que coincida con el nombre del archivo existente.

![Evidencia 2: Configuración del servidor esclavo](evidencias/02-configuracion-esclavo.png)

A continuación se valida la sintaxis y se inicia o recarga BIND9:

```bash
sudo named-checkconf
sudo systemctl restart bind9
sudo systemctl status bind9
```

## 4. Transferencia y sincronización de zonas

Tras iniciar BIND9 en el secundario, este solicita las zonas al primario. Los archivos transferidos deben aparecer en el directorio configurado:

```bash
sudo ls -la /var/cache/bind/slaves/
sudo journalctl -u bind9 -n 50 --no-pager
```

La transferencia será correcta si aparecen los archivos de zona y los logs no muestran errores de permisos, conectividad o autorización.

### Evidencia 4 — Transferencia de zonas

Captura disponible de la comprobación de transferencia de zonas. La validación final mediante consultas DNS se documentará por separado en la Evidencia 5.

![Evidencia 4: Transferencia de zonas](evidencias/04-transferencia-zonas.png)

## 5. Validación del servicio

Se comprobará que ambos servidores responden de manera autoritativa para la zona directa e inversa.

```bash
# Consulta contra el primario
dig @192.168.6.100 ns1.deivid.test A
dig @192.168.6.100 -x 192.168.6.100

# Consulta contra el secundario
dig @IP_DEL_SECUNDARIO ns1.deivid.test A
dig @IP_DEL_SECUNDARIO -x 192.168.6.100
```

La validación será correcta si ambas consultas devuelven los mismos registros `A` y `PTR`, el estado `NOERROR` y el flag `aa`.

También se puede comparar el SOA en ambos servidores:

```bash
dig @192.168.6.100 deivid.test SOA
dig @IP_DEL_SECUNDARIO deivid.test SOA
```

### Evidencia 5 — Consultas DNS contra primario y secundario

Pendiente de captura y documentación de los resultados. No se marca como completada únicamente por disponer de una captura de transferencia de zonas.

## Análisis de incidencias técnicas

Los problemas más habituales son los siguientes:

- El primario no autoriza la IP del secundario en `allow-transfer`.
- El secundario no puede escribir en `/var/cache/bind/slaves/`.
- El puerto TCP 53 está bloqueado: las transferencias de zona utilizan TCP, además del UDP habitual para consultas DNS.
- El serial SOA no se ha incrementado después de modificar una zona.
- El secundario no tiene conectividad con `192.168.6.100`.
- La zona en el secundario tiene una IP errónea en `masters`.

Para diagnosticar estos errores se utilizarán:

```bash
sudo named-checkconf
sudo systemctl status bind9
sudo journalctl -u bind9 -n 50 --no-pager
sudo ss -lntup | grep ':53'
```

## Operación y mantenimiento

Después de modificar una zona en el primario:

1. Incrementar el serial del SOA.
2. Validar el archivo de zona con `named-checkzone`.
3. Recargar BIND9 en el primario con `sudo rndc reload`.
4. Comprobar en el secundario que recibe la actualización.
5. Consultar el SOA en ambos servidores para verificar que muestran el mismo serial.

## Conclusiones

La configuración de un servidor DNS secundario permite mantener la disponibilidad de las zonas DNS si el servidor primario deja de estar disponible. La transferencia de zona y la comprobación del serial SOA garantizan que el secundario conserve una copia sincronizada de los registros autoritativos.

## Evidencias

Las capturas se guardan en la carpeta `evidencias/` y se muestran en su apartado correspondiente. La numeración sigue los nombres de las imágenes, aunque el apartado del primario aparezca antes que el del secundario en la guía.

| Nº | Evidencia | Captura |
|---:|---|---|
| 1 | Hostname del servidor DNS2 | [Ver captura](evidencias/01-hostname-dns2.png) |
| 2 | Configuración del servidor esclavo | [Ver captura](evidencias/02-configuracion-esclavo.png) |
| 3 | Autorización de transferencia en el maestro | [Ver captura](evidencias/03-autorizacion-transferencia-maestro.png) |
| 4 | Transferencia de zonas | [Ver captura](evidencias/04-transferencia-zonas.png) |
| 5 | Consultas DNS contra primario y secundario | Pendiente de captura |

## Estructura del proyecto

```text
ACT4-dns-esclavo/
├── README.md
└── evidencias/
    ├── 01-hostname-dns2.png
    ├── 02-configuracion-esclavo.png
    ├── 03-autorizacion-transferencia-maestro.png
    └── 04-transferencia-zonas.png
```

Las nuevas capturas deben subirse a `ACT4-dns-esclavo/evidencias/`.
