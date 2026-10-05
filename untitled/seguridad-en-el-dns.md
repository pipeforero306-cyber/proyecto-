# Seguridad en el DNS

> **Módulo:** 037 Seguridad en servicios · **Ciclo:** ASIX · **Alumno:** Daniel Felipe Valero Forero

## 1. Introducción y objetivo

El objetivo de esta práctica es analizar la seguridad del servicio DNS y entender los riesgos que trae su uso, estudiando a fondo una técnica de **exfiltración de datos** a través del protocolo DNS (puerto **53 UDP**).

La idea de fondo es sencilla pero peligrosa: el DNS casi nunca se bloquea en los cortafuegos, porque sin él te quedas sin internet. Eso lo convierte en un camino perfecto para sacar información de una red sin levantar sospechas. En la prueba se esconde un mensaje en formato hexadecimal dentro del subdominio de una consulta DNS, y se usan un cliente y un servidor en Python más **Wireshark** para capturar y auditar el tráfico. A esta técnica se la conoce como **canal encubierto** o **DNS tunneling**.

## 2. Instalación de dependencias

Para que el cliente y el servidor pudieran construir y deserializar las tramas DNS con los scripts de Python, instalé las librerías necesarias en los dos entornos.

**En Windows (cliente — 192.168.111.22):**

```powershell
pip install dnspython dnslib
```

**En Debian Trixie 13 (servidor — 192.168.111.53):**

```bash
sudo apt update && sudo apt install python3-dnspython python3-dnslib -y
```

## 3. Arquitectura del entorno

Algo importante: esto **no** está hecho en localhost (127.0.0.1). La práctica pedía demostrarlo entre dos sistemas de verdad, con IPs distintas, para que se viera el tráfico cruzando la red.

| Rol                               | Sistema                          | IP             |
| --------------------------------- | -------------------------------- | -------------- |
| Cliente (emisor / quien exfiltra) | Equipo físico con Windows        | 192.168.111.22 |
| Servidor (receptor)               | Máquina virtual Debian Trixie 13 | 192.168.111.53 |

En la máquina Debian compruebo sus interfaces con `ip a`. Se ve la interfaz `ens33` con la 192.168.111.53, que es justo la IP que después recibe la petición en Wireshark.

> 📸 **Captura 1** — arrastra aquí `01-ips-servidor.png` (salida de `ip a` en Debian)

## 4. Desarrollo paso a paso

### Paso 1 — Liberar el puerto 53 en Debian

La Debian ya tenía BIND9 ocupando el puerto 53 de otra práctica. Como no puede haber dos programas escuchando el mismo puerto, primero paro el servicio:

```bash
sudo systemctl stop bind9
```

Para asegurarme de que el 53 queda libre:

```bash
sudo ss -tulnp | grep :53
```

### Paso 2 — Arrancar el servidor

En Linux hace falta ser administrador para escuchar en puertos por debajo del 1024, así que lo lanzo con `sudo`:

```bash
sudo python3 dns_server_1_peticion.py
```

El servidor se queda esperando con el mensaje `Esperando 1 peticion DNS...`.

### Paso 3 — Configurar y ejecutar el cliente

En el script del cliente, en Windows, cambio la IP de destino para que apunte a la Debian:

```python
servidor = "192.168.111.53"
```

Y lo ejecuto desde PowerShell:

```powershell
python dns_client_1_peticion.py
```

El cliente lanza una consulta de tipo `A` con la carga oculta en el subdominio: `6461746f73206f63756c746f73.secreto.com`.

> 📸 **Captura 2** — arrastra aquí `03-cliente-windows.png` (cliente en PowerShell)

Esa ristra de hex no es un dominio real, es el mensaje disfrazado. Si lo vamos descodificando de dos en dos:

```
64 61 74 6f 73 20 6f 63 75 6c 74 6f 73
 d  a  t  o  s     o  c  u  l  t  o  s
```

O sea, "datos ocultos".

### Paso 4 — El resultado en el servidor

En cuanto llega la petición, la Debian la procesa: recibe el paquete, aísla el subdominio, lo convierte de hexadecimal a texto y recupera el mensaje. La línea clave es `Mensaje oculto recibido: datos ocultos`.

> 📸 **Captura 3** — arrastra aquí `02-servidor-resultado.png` (Debian mostrando "datos ocultos")

La exfiltración ha funcionado. Fíjate además en que el `id` de la petición coincide con el que muestra el cliente, así que es la misma conversación.

## 5. Análisis del tráfico en Wireshark

Dejé Wireshark capturando durante la prueba y apliqué el filtro de protocolo DNS:

```
dns
```

Se ven las dos tramas que forman el intercambio: la petición saliendo de 192.168.111.22 (Windows) hacia 192.168.111.53 (Debian), y la respuesta de vuelta.

### La petición

Es una query de tipo `A` normal a ojos de la red. Lo interesante está en el nombre consultado, el subdominio con los datos en hexadecimal.

> 📸 **Captura 4** — arrastra aquí `04-wireshark-peticion.png` (detalle de la petición)

Incluso en el volcado en hexadecimal del paquete se puede señalar la cadena `6461746f73...` viajando tal cual por el cable.

> 📸 **Captura 5** — arrastra aquí `05-wireshark-peticion-hex.png` (datos ocultos en el hex)

### La respuesta

La segunda trama es la respuesta del servidor, que devuelve un registro `A` con la IP `4.3.2.1`. El campo **Request In** apunta a la petición, confirmando que las dos van juntas.

> 📸 **Captura 6** — arrastra aquí `06-wireshark-respuesta.png` (detalle de la respuesta, trama 36147)

## 6. Desenlace y conclusiones

El script del servidor en Debian completó su ejecución correctamente:

1. Deserializó la trama DNS.
2. Extrajo el nombre de dominio completo de la consulta.
3. Aisló el subdominio y lo tradujo de hexadecimal a texto.
4. Reveló el mensaje oculto exfiltrado: **`datos ocultos`**.
5. Devolvió al cliente una respuesta con el registro `A` apuntando a `4.3.2.1`.

Lo que monté parece una tontería con un mensaje de prueba, pero el concepto es serio: un atacante podría sacar contraseñas o documentos troceándolos en muchas peticiones DNS, y para un cortafuegos normal todo eso parece tráfico de resolución de nombres de lo más inocente.

**Cómo se detectaría:** subdominios muy largos o con pinta aleatoria (mucha entropía), un volumen anormal de consultas hacia un mismo dominio, o tipos de registro poco habituales (TXT, NULL) usados para meter más datos por paquete.

**Cómo se mitiga:** filtrado de DNS obligando a resolver solo contra el servidor interno, inspección de consultas con un IDS que traiga reglas de tunneling, y limitar la longitud y frecuencia de las peticiones hacia dominios externos.

**Conclusión:** esta práctica demuestra una debilidad de diseño del DNS. Al funcionar sobre UDP, sin cifrado y estar permitido por defecto en casi cualquier cortafuegos, puede convertirse en un canal encubierto para exfiltrar información de forma silenciosa. Montarlo entre dos máquinas reales (Windows y Debian Trixie 13) y auditarlo con Wireshark es lo que permite verlo y entenderlo de verdad.
