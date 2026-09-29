# Reto 3 - Fuga de información

Un antiguo colaborador se llevó el diseño del nuevo producto.

**Equipo:** Rubén, Pau, Alex y Daniel

## 1. ¿Qué ha pasado?

* Empresa de ingeniería con oficina, wifi y colaboradores que trabajan en remoto.
* Seis meses de trabajo en el diseño de un nuevo producto.
* Una startup lanza un diseño idéntico justo antes que nosotros.
* Los ex-colaboradores conservaban acceso remoto: no se les revocó.
* Los registros de acceso (con copia de seguridad) identifican al usuario y el momento de la copia.

## 2. Fallos detectados

* Accesos remotos sin revocar a colaboradores que se fueron.
* Información confidencial sin cifrar: el procedimiento existía, pero no se aplicaba.
* RRHH (gestoría) e informática sin coordinación al producirse las bajas.
* Muchos cambios de colaboradores externos en un año, sin control de accesos.
* Punto a favor: logs activos y con backup, y acuerdos de confidencialidad firmados.

## 3. Nuestras ideas

### Rubén

* Quitar el acceso a los ex-trabajadores.
* Margen de 24 h para retirar datos personales.
* RRHH avisa a informática para revocar permisos.

### Pau

* Demandar al ex-colaborador y a la otra empresa si sabía del robo.
* Protocolo de bajas: quitar el acceso al instante.
* Opina que no era evitable por errores previos.

### Alex

* Mejorar el cifrado de datos como red de seguridad.
* Nunca dejar de borrar usuarios inactivos.
* Hay casos en que un fallo puede ocurrir.

### Daniel

* Revocar el acceso de forma automática.
* Notificar los términos legales; demanda si hay filtración.
* Avisar antes a los administradores y reportar a RRHH.

## 4. Medidas de prevención (lo que coincide)

* Revocar accesos de inmediato, idealmente de forma automática, al producirse la baja.
* Coordinar RRHH e informática: cada baja genera un aviso a sistemas.
* Cifrar la información confidencial y aplicar el procedimiento siempre.
* Mantener y revisar los registros de acceso y sus copias.
* Auditar periódicamente usuarios activos y colaboradores externos.

## 5. Vía legal y consecuencias

* Los acuerdos de confidencialidad estaban firmados: se pueden emprender acciones legales.
* Demanda al ex-colaborador y a la nueva empresa si sabía que la información era robada.
* Los logs sirven como prueba: usuario y momento de la copia.
* Pérdidas económicas importantes y pérdida de confianza de los inversores.
* No hay datos personales afectados, así que no hay problema con la AEPD/LOPD.

## 6. Conclusión

* La fuga se pudo evitar con un proceso de baja claro y automático.
* La tecnología (cifrado, logs) solo funciona si se aplica.
* Punto de debate: 24 h de margen para retirar datos (Rubén) frente a revocación inmediata (Pau, Daniel).
* Recomendación: revocación inmediata y entrega de datos personales por un canal controlado.
