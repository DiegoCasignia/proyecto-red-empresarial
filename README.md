# Proyecto Redes de Comunicaciones

## Resumen
El presente proyecto consiste en el diseño, documentación e implementación (simulada en Cisco Packet Tracer) de una infraestructura de red empresarial para una organización con sede central (**Matriz**) y dos sucursales (**Guayaquil** y **Cuenca**). 

La solución integra tecnologías de segmentación de red (VLANs), enrutamiento dinámico (OSPF y EIGRP), conectividad WAN segura mediante **VPN Site-to-Site con GRE sobre IPsec** y servicios esenciales como DHCP, DNS, HTTP, FTP y correo electrónico. El diseño sigue el estándar de **arquitectura jerárquica de tres capas** con **núcleo colapsado** y utiliza **VLSM** para optimizar el direccionamiento IP.

---

## Topología Física y Conexiones (Detalle por Puerto)

A continuación se enumeran **todas las conexiones físicas** entre los dispositivos, organizadas por sede y por red de proveedor.

### 1. MATRIZ – Quito (UIO)
La matriz cuenta con un router de borde, un switch multicapa (núcleo colapsado) y cuatro switches de acceso.

| **Origen** | **Puerto Origen** | **Destino** | **Puerto Destino** | **Descripción** |
| :--- | :--- | :--- | :--- | :--- |
| R1-UIO | Gig0/0/1 | MLS-UIO | Gig1/0/1 | Enlace troncal (routing) entre el router y el núcleo. |
| MLS-UIO | Gig1/0/2 | SW1-UIO | Gig0/1 | Enlace trunk a switch de acceso 1. |
| MLS-UIO | Gig1/0/3 | SW2-UIO | Gig0/1 | Enlace trunk a switch de acceso 2. |
| MLS-UIO | Gig1/0/4 | SW3-UIO | Gig0/1 | Enlace trunk a switch de acceso 3. |
| MLS-UIO | Gig1/0/5 | SW4-UIO | Gig0/1 | Enlace trunk a switch de acceso 4. |

> **Nota:** Los switches SW1-UIO a SW4-UIO son **switches L2 (2960)**. Sus puertos FastEthernet restantes se asignan a las VLANs correspondientes para conectar estaciones de trabajo y servidores.

---

### 2. SUCURSAL 1 – Guayaquil (GYE)
La sucursal de Guayaquil cuenta con un router, un switch multicapa y dos switches de acceso.

| **Origen** | **Puerto Origen** | **Destino** | **Puerto Destino** | **Descripción** |
| :--- | :--- | :--- | :--- | :--- |
| R1-GYE | Gig0/0/1 | MLS-GYE | Gig1/0/1 | Enlace troncal (routing) entre el router y el núcleo. |
| MLS-GYE | Gig1/0/2 | SW1-GYE | Gig0/1 | Enlace trunk a switch de acceso 1. |
| MLS-GYE | Gig1/0/3 | SW2-GYE | Gig0/1 | Enlace trunk a switch de acceso 2. |

---

### 3. SUCURSAL 2 – Cuenca (CUE) *(Con corrección STP)*
La sucursal de Cuenca cuenta con un router, un switch multicapa y **tres switches L2**. 

> - **SW1-CUE** y **SW2-CUE** son los switches destinados a la conexión de usuarios finales y hosts.
> - **SW3-CUE** no está destinado a usuarios finales; su función principal es **generar la redundancia y el bucle L2** necesario para implementar y verificar las **modificaciones de STP (Spanning Tree Protocol)**, cumpliendo así con el requerimiento del proyecto de evitar bucles L2 mediante la configuración de prioridades y bloqueo de puertos.

| **Origen** | **Puerto Origen** | **Destino** | **Puerto Destino** | **Descripción** |
| :--- | :--- | :--- | :--- | :--- |
| R1-CUE | Gig0/0/1 | MLS-CUE | Gig1/0/1 | Enlace troncal (routing) entre el router y el núcleo. |
| MLS-CUE | Gig1/0/2 | SW1-CUE | Gig0/1 | Enlace trunk a switch de acceso 1 (usuarios). |
| MLS-CUE | Gig1/0/3 | SW2-CUE | Gig0/1 | Enlace trunk a switch de acceso 2 (usuarios). |
| MLS-CUE | Gig1/0/4 | SW3-CUE | Gig0/2 | Enlace trunk hacia el switch dedicado a STP. |
| SW2-CUE | Gig0/2 | SW3-CUE | Gig0/1 | Enlace directo entre switches L2 para crear el bucle STP. |

---

### 4. RED WAN – PROVEEDOR DE SERVICIOS (ISP)
Se implementa una nube de **7 routers ISP** que simulan la red pública de transporte con topología de malla parcial.

#### 4.1. Conexiones de las sedes hacia el ISP

| **Origen** | **Puerto Origen** | **Destino** | **Puerto Destino** | **Descripción** |
| :--- | :--- | :--- | :--- | :--- |
| R1-GYE | Se0/1/0 | R1-ISP | Se0/1/0 | Enlace WAN de Guayaquil. |
| R1-UIO | Se0/1/0 | R3-ISP | Se0/1/0 | Enlace WAN de Quito. |
| R1-CUE | Se0/1/0 | R5-ISP | Se0/1/0 | Enlace WAN de Cuenca. |

#### 4.2. Interconexiones entre routers ISP (Backbone)

| **Origen** | **Puerto Origen** | **Destino** | **Puerto Destino** |
| :--- | :--- | :--- | :--- |
| R1-ISP | Se0/2/0 | R2-ISP | Se0/1/0 |
| R2-ISP | Se0/1/1 | R3-ISP | Se0/1/1 |
| R3-ISP | Se0/2/0 | R4-ISP | Se0/1/0 |
| R4-ISP | Se0/1/1 | R5-ISP | Se0/1/1 |
| R5-ISP | Se0/2/0 | R6-ISP | Se0/1/1 |
| R6-ISP | Se0/1/0 | R1-ISP | Se0/1/1 |
| R7-ISP | Se0/1/0 | R2-ISP | Se0/2/0 |
| R7-ISP | Se0/1/1 | R4-ISP | Se0/2/0 |
| R7-ISP | Se0/2/0 | R6-ISP | Se0/2/0 |

---

## Requerimientos Técnicos Implementados

| **Tecnología / Servicio** | **Descripción** |
| :--- | :--- |
| **VLANs** | Segmentación lógica de la red. Matriz: 7 departamentos. Sucursales: 4 departamentos cada una. |
| **Inter-VLAN Routing** | Mediante SVIs (Switch Virtual Interfaces) en los switches multicapa (MLS). |
| **STP (Spanning Tree)** | Configuración para evitar bucles L2; el MLS de cada sede es el **Root Bridge** primario. En Cuenca, SW3-CUE se utiliza para probar el bloqueo de puertos y redundancia. |
| **Enrutamiento Dinámico** | **OSPF** (Área 0) para el *underlay* (redes públicas de los ISP). <br> **EIGRP** (AS 100) para el *overlay* (redes privadas y túneles). |
| **Diseño Jerárquico** | Modelo de **tres capas** (Acceso, Distribución, Núcleo) con **núcleo colapsado** en el MLS de cada sede. |
| **VPN Site-to-Site** | **GRE** (Generic Routing Encapsulation) sobre **IPsec** (AES-256, SHA) para la conectividad privada entre las 3 sedes a través de la WAN. |
| **NAT (Overload)** | Traducción de direcciones para salida a Internet en el router de borde de cada sede. |
| **DHCP** | Servicio de asignación automática de IPs para todas las VLANs de usuarios (configurado en los MLS). |
| **Servicios Internos** | **DNS**, **Web (HTTP)**, **FTP** y **Correo (SMTP/POP3)** alojados en el Data Center de la Matriz. |
| **Seguridad** | Acceso remoto exclusivo mediante **SSH** (sin Telnet). Contraseñas encriptadas y usuarios locales. |

---

## Esquema de Direccionamiento IP (VLSM)

Todo el direccionamiento privado se ajusta al estándar **RFC 1918**, mientras que el enlace de última milla y la red del proveedor utilizan direccionamiento público.

### A. Subredes Privadas – LAN y Enlaces Internos

| **Sede** | **VLAN / Uso** | **Red Asignada** | **Máscara** | **Rango de Hosts** |
| :--- | :--- | :--- | :--- | :--- |
| **MATRIZ (UIO)** | RH (20) | 192.168.20.0 | /26 | .1 – .62 |
| | Operaciones (30) | 192.168.30.0 | /27 | .1 – .30 |
| | Gerencia (40) | 192.168.40.0 | /27 | .1 – .30 |
| | Capacitaciones (60) | 192.168.60.0 | /28 | .1 – .14 |
| | Finanzas (70) | 192.168.70.0 | /27 | .1 – .30 |
| | Marketing (80) | 192.168.80.0 | /27 | .1 – .30 |
| | Data Center / TI (90) | 192.168.90.0 | /28 | .1 – .14 |
| | Enlace R1 ↔ MLS | 192.168.0.0 | /30 | .1 (R1) / .2 (MLS) |
| **SUCURSAL 1 (GYE)** | Atención Cliente (110) | 172.168.110.0 | /26 | .1 – .62 |
| | Administrativo (120) | 172.168.120.0 | /26 | .1 – .62 |
| | Logística (130) | 172.168.130.0 | /24 | .1 – .254 |
| | Seguridad Física (140) | 172.168.140.0 | /27 | .1 – .30 |
| | Enlace R1 ↔ MLS | 172.168.0.0 | /30 | .1 (R1) / .2 (MLS) |
| **SUCURSAL 2 (CUE)** | Atención Cliente (150) | 10.0.150.0 | /26 | .1 – .62 |
| | Soporte Técnico (160) | 10.0.160.0 | /26 | .1 – .62 |
| | Administrador (170) | 10.0.170.0 | /27 | .1 – .30 |
| | Invitados (180) | 10.0.180.0 | /27 | .1 – .30 |
| | Enlace R1 ↔ MLS | 10.0.0.0 | /30 | .1 (R1) / .2 (MLS) |

### B. Túneles VPN (Overlay – GRE)

| **Túnel** | **Red Asignada** | **Extremo UIO** | **Extremo GYE** | **Extremo CUE** |
| :--- | :--- | :--- | :--- | :--- |
| UIO ↔ GYE | 192.168.200.0/30 | .1 | .2 | — |
| UIO ↔ CUE | 192.168.200.4/30 | .5 | — | .6 |
| GYE ↔ CUE | 192.168.200.8/30 | — | .9 | .10 |

### C. Direccionamiento Público (WAN e ISP)

| **Equipo** | **Interfaz** | **IP Asignada** | **Destino** |
| :--- | :--- | :--- | :--- |
| R1-UIO | Se0/1/0 | 200.0.1.1/30 | R3-ISP (Se0/1/0: 200.0.1.2) |
| R1-GYE | Se0/1/0 | 200.0.2.1/30 | R1-ISP (Se0/1/0: 200.0.2.2) |
| R1-CUE | Se0/1/0 | 200.0.3.1/30 | R5-ISP (Se0/1/0: 200.0.3.2) |
| Routers ISP | Enlaces seriales | 200.0.4.0/30 – 200.0.12.0/30 | Asignación secuencial según topología. |
| Loopbacks (Sede) | Loopback0 | 10.255.255.1/32 (UIO), .2 (GYE), .3 (CUE) | Identificación OSPF / EIGRP. |

---

## Servicios de Red (Data Center – Matriz)

Los servidores se ubican en la VLAN 90 con direccionamiento fijo y son accesibles desde cualquier sede a través de la VPN:

| **Servicio** | **Nombre DNS** | **IP Fija** | **Gateway** |
| :--- | :--- | :--- | :--- |
| DNS | `servidor.rugied.local` | 192.168.90.10 | 192.168.90.1 |
| Web | `www.rugied.local` | 192.168.90.11 | 192.168.90.1 |
| FTP | `ftp.rugied.local` | 192.168.90.12 | 192.168.90.1 |
| Correo (SMTP/POP3) | `mail.rugied.local` | 192.168.90.13 | 192.168.90.1 |

Todos los clientes reciben la dirección del servidor DNS (192.168.90.10) mediante DHCP, lo que permite la resolución de nombres interna.

---

## Pruebas de Conectividad (Validación)

Para comprobar la correcta implementación, se realizan las siguientes pruebas extremo a extremo:

1. **Ping intra-VLAN**: Cliente en VLAN 20 → Gateway (192.168.20.1).
2. **Ping inter-VLAN**: Cliente en VLAN 20 → Servidor DNS (192.168.90.10).
3. **Ping entre sedes (VPN)**: Cliente en Matriz (VLAN 20) → Cliente en Guayaquil (VLAN 110) utilizando el overlay de EIGRP sobre los túneles GRE/IPsec.
4. **Salida a Internet (NAT)**: Ping desde cualquier dispositivo interno hacia una dirección pública del ISP (ej. 200.0.4.1).
5. **Acceso a servicios**:
   - Navegador web → `http://www.rugied.local`.
   - Cliente FTP → `ftp.rugied.local`.
   - Cliente de correo → `mail.rugied.local` (SMTP/POP3).

---

## Seguridad Aplicada

- Desactivación de servicios no seguros (no se utiliza TELNET en ningún dispositivo crítico).
- Configuración de **SSH versión 2** con autenticación local en los routers y switches multicapa.
- Encriptación de contraseñas en ejecución (`service password-encryption`) y contraseñas secretas para privilegios.
- Filtrado de tráfico mediante ACLs para la protección de los túneles VPN (solo se permite tráfico GRE entre los peers autorizados).

---

Session ID: ses_0a67981d5ffeNzkt7bSog8tIgM
