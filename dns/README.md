# Análisis y Resolución del Sistema DNS

| Datos de la Práctica   | Análisis y Resolución del Sistema DNS          |
| ---------------------- | ---------------------------------------------- |
| **Módulo**             | 0375 Servicios de red                          |
| **Ciclo Formativo**    | ASIX                                           |
| **Alumno**             | **Daniel Valero**                              |
| **Formato de Entrega** | URL al informe documentado en Gitbook personal |

***

## 1. El ecosistema DNS (OSINT y Web) \[2p]

El sistema DNS es una estructura jerárquica a nivel mundial. Para administrar redes, primero debemos entender quién gestiona cada parte del pastel.

### 1.1. Investigación de Jerarquía

**¿Qué organismo internacional coordina y asigna los parámetros a nivel global del sistema de nombres de dominio e IPs?**

La **ICANN** (Corporación de Internet para la Asignación de Nombres y Números), actuando específicamente a través de su departamento **IANA** (Autoridad de Números Asignados en Internet), es quien coordina los parámetros globales, direcciones IP y el sistema de dominios raíz.

**¿Qué empresa u organismo gestiona (Registry) cada uno de los siguientes dominios de nivel superior (TLD)?**

* **.es:** Gestionado por **Red.es** (Entidad Pública Empresarial adscrita al gobierno de España).
* **.cat:** Gestionado por la **Fundació puntCAT**.
* **.edu:** Gestionado por **EDUCAUSE** (con la infraestructura técnica operada por Verisign).
* **.ifp.es:** A nivel técnico, .ifp.es no es un TLD (Top Level Domain), sino un dominio de segundo nivel (SLD). El TLD sigue siendo .es (gestionado por Red.es). El dominio ifp.es en sí fue registrado y es gestionado por la propia institución académica titular.

### 1.2. Herramientas OSINT (Whois y DNS Lookup)

Utilizando herramientas online (como Dominios.es, whois.com, nslookup.io) se responde a lo siguiente:

**a. ¿Qué información te brinda una consulta Whois sobre un dominio?**

Una consulta _Whois_ proporciona los metadatos y el registro público de un nombre de dominio. Esta información incluye:

* **Datos del registrador:** La empresa donde se registró el dominio.
* **Fechas clave:** Fecha de creación, última actualización y fecha de expiración del dominio.
* **Servidores de nombres (DNS):** Los servidores autoritativos delegados para resolver el dominio.
* **Estado del dominio:** Si está activo, está bloqueado para transferencias.
* **Datos de contacto:** Información del titular, contacto administrativo y técnico (frecuentemente ofuscados por normativas de privacidad como el RGPD).

**b. Define brevemente la diferencia entre el Registry de la base de datos y el Registrar (Registrador) del dominio.**

* **Registry (Registro):** Es la organización central que posee y administra la base de datos global de un Dominio de Nivel Superior (TLD) específico. Por ejemplo, Red.es gestiona el TLD .es. No venden dominios directamente al público.
* **Registrar (Registrador):** Es la empresa comercial (ejm DonDominio, GoDaddy, Hostinger) acreditada por el _Registry_ para vender los dominios. Son los intermediarios que interactúan con los clientes finales y se encargan de actualizar los datos en la base de datos del _Registry_.

**c. Investiga: ¿Qué es DNSSEC y qué problema de seguridad intenta resolver en las resoluciones DNS?**

**¿Qué es DNSSEC?** Las Domain Name System Security Extensions (DNSSEC) son un conjunto de protocolos que añaden firmas criptográficas a los registros del sistema DNS.

**Problema que resuelve:** Su objetivo es prevenir infección en la caché (DNS Spoofing / Cache Poisoning) en las resoluciones DNS. DNSSEC garantiza la autenticidad del origen y la integridad de los datos, asegurando que la respuesta DNS recibida proviene legítimamente del servidor autorizado y no ha sido alterada por un atacante para redirigir el tráfico hacia un sitio web malicioso.

### 1.3. Rendimiento DNS (GRC DNS Benchmark)

Se descarga e inicia la aplicación [GRC DNS Benchmark](https://www.grc.com/dns/benchmark.htm) (no requiere instalación), se ejecuta el test para comprobar cuáles son los servidores DNS más rápidos desde mi ubicación y se seleccionan los **3 servidores** más rápidos, anotando sus IPs y de qué empresas son.

> 📷 **Captura:** GRC DNS Benchmark, pestaña _Nameservers_ → _Name_ (156.154.71.22, 208.67.222.220 y 208.67.222.222).

> 📷 **Captura:** GRC DNS Benchmark, pestaña _Nameservers_ → _Owner_ (VERCARA – Vercara, LLC, US / CISCO-UMBRELLA – Cisco OpenDNS, LLC, US).

**IP: 156.154.71.22**

* **Nombre mostrado:** Aparece sin nombre oficial (... no official Internet DNS name ...).
* **Empresa:** Pertenece a **Vercara** (empresa que actualmente opera la infraestructura antes conocida como Neustar / UltraDNS), una compañía especializada en servicios de resolución DNS gestionados y ciberseguridad corporativa.

**IP: 208.67.222.220**

* **Nombre mostrado:** resolver3.opendns.com
* **Empresa:** Pertenece a **OpenDNS**. Es un servicio público muy utilizado que ofrece resolución rápida y opciones de filtrado de contenido para mejorar la seguridad.

**IP: 208.67.222.222**

* **Nombre mostrado:** dns.sse.cisco.com
* **Empresa:** Pertenece a **Cisco** (empresa que compró OpenDNS). Forma parte de su plataforma de seguridad en la nube (Cisco Umbrella) diseñada para bloquear dominios malintencionados a nivel de DNS.

***

## 2. Configuración y Caché \[2p]

Todo sistema operativo guarda las resoluciones DNS para no saturar la red.

### 2.1. Cambio de servidores DNS

**¿Cómo puedes ver mediante consola (CLI) qué servidores DNS tienes asignados actualmente en Windows y en Linux?**

**En Windows:** Se visualizan abriendo el Símbolo del sistema (CMD) o PowerShell y ejecutando el comando `ipconfig /all`. Los servidores asignados aparecerán detallados en la línea que dice "Servidores DNS" dentro de la configuración del adaptador de red activo.

> 📷 **Captura:** salida de `ipconfig /all` (adaptador Wi-Fi) con los servidores DNS 1.1.1.2 y 8.8.4.4.

**En Linux:** Abriendo la terminal y ejecutando el comando `cat /etc/resolv.conf`. Los servidores asignados aparecen junto a la palabra `nameserver`.

> 📷 **Captura:** salida de `cat /etc/resolv.conf` en WSL (nameserver 10.255.255.254).

_Nota técnica:_ Al estar trabajando en un entorno WSL, la consola muestra la IP del conmutador virtual 10.255.255.254, el cual hereda y utiliza automáticamente los DNS que ya hemos configurado en el Windows anfitrión.

**Cambia la configuración de red de tu equipo principal (Windows o Linux) poniendo como DNS primario y secundario los que obtuviste en el Benchmark de la Fase 1. Muestra captura del cambio.**

> 📷 **Captura:** `ipconfig /all` tras el cambio, con los servidores DNS 156.154.71.22 y 208.67.222.220.

**¿En qué menú de tu dispositivo móvil (Android/iOS) podrías forzar el uso de unos DNS específicos para tu conexión Wi-Fi?**

En Android (HyperOS / MIUI): Se debe acceder a **Ajustes > Wi-Fi**, tocar la flecha o el icono de detalles que aparece junto a la red conectada, ir a la opción **Ajustes de IP**, cambiar el modo de "DHCP" a "Estática" y modificar los campos "DNS 1" y "DNS 2". Como se puede evidenciar en la siguiente comprobación en mi dispositivo:

> 📷 **Captura:** ajustes Wi-Fi del móvil (IP estática 192.168.0.30, DNS 1 y DNS 2).

### 2.2. Gestión de la caché DNS (ipconfig / resolvectl)

Para visualizar las resoluciones DNS que mi sistema operativo Windows ha guardado en memoria, he abierto el Símbolo del sistema (CMD) y he ejecutado el comando `ipconfig /displaydns`. A continuación, se muestran algunas de las direcciones y registros almacenados localmente en el equipo.

**Mostrar direcciones almacenadas en caché:**

> 📷 **Captura:** salida de `ipconfig /displaydns` (registros PTR y A de Daniel.mshome.net).

**Vaciar la caché y utilidad para un administrador de sistemas:**

> 📷 **Captura:** salida de `ipconfig /flushdns` ("Se vació correctamente la caché de resolución de DNS").

Ejecución del vaciado de caché: Mediante el comando `ipconfig /flushdns` en la consola de Windows, se ha forzado al sistema operativo a borrar de forma inmediata todo su historial temporal de resoluciones DNS. Con esta acción concreta, la memoria local queda limpia de registros antiguos y se obliga al equipo a realizar peticiones nuevas a los servidores DNS configurados para cualquier futura conexión.

***

## 3. Administración - Troubleshooting con DIG y CLI \[3p]

En un entorno profesional, especialmente servidores Linux, la herramienta `nslookup` se considera obsoleta, siendo `dig` (Domain Information Groper) el estándar de la industria.

### 3.1. Consultas específicas de registros (dig en WSL)

Consultas sobre el dominio aliexpress.com.

#### Registro A: `dig aliexpress.com`

> 📷 **Captura:** salida de `dig aliexpress.com`.

Al ejecutar el comando sin modificadores, por defecto se realiza una consulta del registro A (dirección IPv4). **Análisis de la ANSWER SECTION:**

La terminal nos devuelve dos líneas, lo que indica que aliexpress.com tiene asignadas al menos dos direcciones IP públicas (47.246.111.53 y 47.246.75.137) para balancear el tráfico y tener redundancia. Cada registro devuelto se compone de los siguientes campos:

* **Nombre:** aliexpress.com. (el dominio consultado).
* **TTL (Time to Live):** 5 (indica que el registro puede permanecer en caché durante 5 segundos antes de tener que volver a consultarse).
* **Clase:** IN (hace referencia a Internet).
* **Tipo:** A (indica que el registro es una dirección de host IPv4).
* **Dato:** La dirección IP destino final (47.246.111.53 y 47.246.75.137).

#### Short: `dig +short aliexpress.com`

> 📷 **Captura:** salida de `dig +short aliexpress.com`.

Al utilizar el modificador `+short`, la herramienta omite toda la estructura, cabeceras y metadatos de la consulta DNS, devolviendo de forma limpia y exclusiva el valor solicitado (las direcciones IP 47.246.75.137 y 47.246.111.53).

**¿Por qué es útil este formato en scripts de Bash?** Es extremadamente útil para la automatización porque evita tener que usar herramientas adicionales de filtrado de texto (como grep, awk o sed) para extraer la IP de en medio de todo el código. Un administrador de sistemas puede asignar el resultado directamente a una variable dentro de un script (por ejemplo, `IP=$(dig +short dominio.com)`) de forma directa, rápida y sin margen de error para automatizar tareas, bloqueos en firewalls o monitorización.

#### MX: `dig MX aliexpress.com`

> 📷 **Captura:** salida de `dig MX aliexpress.com`.

El parámetro MX especifica que queremos consultar los registros de intercambio de correo (Mail Exchange). Estos registros indican a qué servidores se deben enviar los correos electrónicos dirigidos al dominio @aliexpress.com.

**Identificación de la Prioridad/Preference:** Analizando la ANSWER SECTION devuelta por la consulta (`aliexpress.com. 600 IN MX 10 mx2.mail.aliyun.com.`), identificamos que el campo de Prioridad es el número **10**. Este valor numérico le indica a los agentes de transferencia de correo (MTA) el orden de preferencia. Si existieran varios registros MX, el correo siempre intentará entregarse primero al servidor con el número de prioridad más bajo (el de mayor preferencia).

#### NS: `dig NS aliexpress.com`

> 📷 **Captura:** salida de `dig NS aliexpress.com`.

El parámetro NS (Name Server) se utiliza para identificar a los servidores de nombres delegados de un dominio.

**Servidores con autoridad sobre la zona:** Analizando la ANSWER SECTION devuelta por el comando, podemos ver que los servidores que tienen la autoridad (es decir, los servidores primarios que contienen la base de datos oficial con todos los registros de esa zona DNS) para el dominio aliexpress.com son dos:

* `ns1.alibabadns.com.`
* `ns2.alibabadns.com.`

(Como detalle técnico extra, la sección ADDITIONAL SECTION nos adelanta las direcciones IP de estos servidores autoritativos, ahorrando al sistema tener que hacer una segunda consulta DNS para encontrarlos).

### 3.2. Autoridad y Caché (TTL)

**¿Qué diferencia existe entre un registro SOA (Start of Authority) y un registro NS (Name Server)?**

* **SOA (Start of Authority):** Es el registro principal y único de una zona. Define quién es el administrador y los parámetros de control (tiempos de refresco, número de serie).
* **NS (Name Server):** Son los registros de delegación. Simplemente listan qué servidores DNS están autorizados para responder consultas sobre ese dominio.

**Realiza una consulta a un dominio cualquiera. Observa el valor TTL (Time To Live). Vuelve a realizar la consulta a los 5 segundos. ¿Qué ha pasado con el valor numérico del TTL? ¿Qué nos demuestra esto sobre el origen de la respuesta (Caché vs Servidor Autoritativo)?**

> 📷 **Captura 1:** primera consulta `dig amazon.com` (TTL 700).

> 📷 **Captura 2:** segunda consulta `dig amazon.com` (TTL 687).

Al realizar la primera consulta DNS al dominio amazon.com, el sistema nos devuelve un tiempo de vida (TTL) de 700 segundos. Al repetir exactamente la misma consulta en la terminal unos instantes después, observamos que el valor del TTL ha bajado a 687.

Esta cuenta atrás demuestra de forma práctica que la segunda respuesta proviene de la memoria caché de nuestro servidor DNS local. Si la segunda respuesta viniera directamente del servidor autoritativo de Amazon, el contador de TTL se habría reiniciado mostrándonos su valor máximo original en lugar de continuar restando segundos.

### 3.3. Trazabilidad Completa (Trace)

Comando ejecutado: `dig +trace aliexpress.com`

> 📷 **Captura:** salida de `dig @8.8.8.8 +trace aliexpress.com`.

Al ejecutar el comando `dig +trace aliexpress.com`, obligamos a nuestro equipo a ignorar la caché local y realizar una resolución DNS iterativa completa. Como se puede observar en la captura, el proceso sigue estos tres pasos exactos:

1. **Consulta a los Root Servers (.):** Al no saber dónde está el dominio, mi ordenador contacta primero con los servidores raíz de Internet (representados por el punto `.`). Estos servidores no tienen la IP final, pero nos indican qué servidores gestionan las extensiones .com.
2. **Consulta a los servidores del TLD (.com):** A continuación, mi equipo pregunta a los servidores Top-Level Domain (TLD) encargados de la zona .com. Estos tampoco tienen la respuesta final, pero nos redirigen a los servidores que gestionan específicamente el dominio aliexpress.com.
3. **Consulta a los Servidores Autoritativos:** Finalmente, contactamos con los servidores autoritativos del propio dominio (como ns1.alibabadns.com y ns2.alibabadns.com). Al ser los dueños de la zona, tienen la autoridad y nos devuelven, ahora sí, las direcciones IP (registros A) correctas de AliExpress.

***

## 4. Análisis de Tráfico de Red (Wireshark) \[3p]

Vamos a comprobar qué viaja realmente por el cable físico cuando resolvemos un nombre.

### 4.1. Inicio de la captura

> 📷 **Captura:** pantalla de inicio de Wireshark con la interfaz _Ethernet_ seleccionada.

En esta primera fase, iniciamos Wireshark y seleccionamos la interfaz de red principal del equipo (en este caso, Ethernet). Al iniciar la captura en esta interfaz, el programa comienza a registrar todo el tráfico de datos físico que entra y sale del pc para su análisis.

### 4.2. Filtro DNS

> 📷 **Captura:** paquetes 79 y 80 filtrados con `dns`.

Para aislar el tráfico que nos interesa entre todo el ruido de la red, aplicamos el filtro `dns` (o `udp.port == 53`). Como se observa en la captura, hemos localizado exactamente el intercambio generado por nuestro nslookup:

* **Paquete 79:** Es la petición o consulta (_Standard query_) que nuestro equipo hace pidiendo el registro MX de google.com.
* **Paquete 80:** Es la respuesta (_Standard query response_) del servidor DNS entregando la información solicitada.

### 4.3. Consulta forzada con nslookup

Comando ejecutado: `nslookup -type=mx google.com`

> 📷 **Captura:** PowerShell con la salida de `nslookup -type=mx google.com`.

A través de la consola de Windows PowerShell, forzamos una petición DNS manual utilizando el comando `nslookup -type=mx google.com`. Con esto, le solicitamos específicamente a nuestro servidor DNS (192.168.111.1) que nos devuelva los registros de intercambio de correo (MX) del dominio google.com. La consola muestra una respuesta 'no autoritativa', indicando que el servidor de correo es `smtp.google.com` con una preferencia de 10.

### 4.4. Análisis del intercambio Petición / Respuesta

> 📷 **Captura:** panel de detalles de Wireshark, Paquete 79 (Frame, Ethernet II, IPv4, UDP, DNS).

Al detener la captura y seleccionar el **Paquete 79**, el panel de detalles nos muestra cómo se encapsula la petición de red capa por capa: vemos la trama Ethernet física, el Protocolo de Internet (IPv4) con las direcciones de origen y destino, el Protocolo de Datagramas de Usuario (UDP) apuntando al puerto 53 (específico de DNS), y finalmente la capa de aplicación del Sistema de Nombres de Dominio (DNS) que contiene nuestra consulta.

**Capa de Transporte: ¿Qué protocolo se utiliza (TCP o UDP)? ¿Por qué DNS utiliza este protocolo por defecto en lugar del otro?**

El DNS utiliza **UDP** ya que sus consultas son más rápidas y ligeras para resolver la dirección IP a partir del nombre de dominio.

El **TCP** se descarta para esta función y se reserva para protocolos como **HTTPS**, ya que está orientado a conexiones fiables, seguras y pesadas que aseguran la transferencia completa de toda la página web sin pérdida de paquetes.

**Puertos: Identifica el puerto de origen (dinámico) del cliente y el puerto de destino (conocido) del servidor.**

El puerto de origen es el **53952**, un puerto lógico dinámico asignado de forma aleatoria por el sistema operativo del cliente únicamente para esta consulta. El puerto de destino es el **53**, que corresponde al puerto que escucha el servidor DNS.

**Identificador: Expande la sección Domain Name System. ¿Qué identificador de transacción (Transaction ID) vincula la respuesta del servidor con la petición de tu cliente?**

El identificador de transacción es **0x0002**. Este código es fundamental porque permite vincular de forma única la respuesta del servidor DNS con la petición enviada por el cliente, evitando confusiones si hay múltiples peticiones DNS simultáneas en la red.

**Flags: En el paquete de Respuesta, despliega la sección Flags. Busca la opción Authoritative Answer. ¿Está a 0 o a 1? ¿Qué significa esto?**

En el paquete de respuesta, la opción _Authoritative Answer_ está a **0**. Esto significa que el servidor DNS que nos está respondiendo no es el servidor autoritativo oficial y principal del dominio, sino que nos ha proporcionado la información a partir de su memoria caché o actuando como un servidor recursivo intermediario.

**Respuestas (Answers): Despliega el bloque de respuestas. ¿Qué servidor de correo de Google tiene la prioridad (preference) más alta (el número más bajo)?**

> 📷 **Captura:** bloque _Answers_ del paquete de respuesta (google.com: type MX, preference 10, mx smtp.google.com).

Al desplegar el bloque de respuestas (_Answers_), se observa que el servidor de correo de Google con la prioridad más alta (identificada por el número de preferencia más bajo, que es 10) es **smtp.google.com**.

***

## Documento original

Informe original de la práctica en PDF, con todas las capturas de pantalla:
