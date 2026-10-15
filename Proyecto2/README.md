# **_Proyecto 2_**

## Proyecto Conectividad XXI: Integración de Operadores Nacionales

### 2do Semestre del 2026

```js
Universidad San Carlos de Guatemala
Programador: César Rolando Hernández Palacios
Carne: 202000806
Correo: cesarhernandez0019@gmail.com
```

---

## Manual Técnico

### 1. **Datos generales**

#### 1.1 **Redes asignadas según el carné (202000806)**

| Uso               | Regla del enunciado                         | Red resultante                  |
| ----------------- | ------------------------------------------- | ------------------------------- |
| Interconexión BGP | `192.168.XX.0/16` (XX = últimos 2 dígitos)  | 192.168.6.0 (subneteada en /30) |
| FibraNet GT       | `172.16.1X.0/24` (X = último dígito: 6)     | **172.16.16.0/24**              |
| AndinaCom         | `172.16.2X.0/24` (X = último dígito: 6)     | **172.16.26.0/24**              |
| MoonLink          | `172.16.3X.0/24` (X = **primer** dígito: 2) | **172.16.32.0/24**              |

---

### 2. **Topología completa**

![imagen](/Proyecto2/img/topologia1.png)

| ISP         | Topología            | Protocolo interno | Departamentos                       | Servicios / extras |
| ----------- | -------------------- | ----------------- | ----------------------------------- | ------------------ |
| FibraNet GT | Árbol                | OSPF              | Administración, Atención al cliente | DNS y HTTP/HTTPS   |
| AndinaCom   | Jerárquica (3 capas) | OSPF              | Ventas, Facturación                 | DHCP, HSRP         |
| MoonLink    | Hub-and-spoke        | EIGRP             | Soporte, Seguridad                  | Red inalámbrica    |

Los tres ISPs se interconectan mediante BGP entre sus switches multicapa de borde.

---

### 3. **ISP 1: FibraNet GT**

#### 3.1 **Requisitos**

- Topología en árbol.
- Protocolo de enrutamiento interno: OSPF (área 0).
- Mínimo 5 routers, 5 hosts y 2 enlaces LACP.
- Dos departamentos (Administración y Atención al cliente), con subredes para hasta 75 hosts.
- Provee el servicio DNS y HTTP/HTTPS a toda la topología.

#### 3.2 **Topología**

La raíz del árbol es el switch multicapa F-CORE (3650-24PS), que además es el equipo de borde BGP. De él cuelgan tres ramas, y cada nodo tiene un único padre, lo que mantiene la topología en árbol.

![imagen](/Proyecto2/img/topologia3.png)

| Dispositivo                 | Modelo    | Función                                      |
| --------------------------- | --------- | -------------------------------------------- |
| F-CORE (Multilayer Switch0) | 3650-24PS | Raíz del árbol, borde BGP                    |
| R5                          | 2911      | Gateway del segmento de servidores           |
| R1                          | 2911      | Agregación de la rama de Administración      |
| R3                          | 2911      | Gateway de Administración                    |
| R2                          | 2911      | Agregación de la rama de Atención al cliente |
| R4                          | 2911      | Gateway de Atención al cliente               |
| Switch1 / Switch3           | 2960-24TT | Acceso Administración (par LACP)             |
| Switch2 / Switch0           | 2960-24TT | Acceso Atención al cliente (par LACP)        |
| Switch de servidores        | 2960-24TT | Acceso del segmento de servicios (VLAN 10)   |
| Server0                     | Server-PT | DNS y HTTP/HTTPS                             |

Total: 5 routers (R1 a R5), 5 hosts (PC0 a PC4) y 2 enlaces LACP.

#### 3.3 **Subneteo de departamentos (172.16.16.0/24)**

Se requieren subredes para hasta 75 hosts. Se necesitan 7 bits de host (2⁷ − 2 = 126 útiles), por lo que corresponde una máscara /25.

| Departamento        | Red              | Máscara         | Rango útil  | Broadcast     | Gateway            |
| ------------------- | ---------------- | --------------- | ----------- | ------------- | ------------------ |
| Administración      | 172.16.16.0/25   | 255.255.255.128 | .1 – .126   | 172.16.16.127 | 172.16.16.1 (R3)   |
| Atención al cliente | 172.16.16.128/25 | 255.255.255.128 | .129 – .254 | 172.16.16.255 | 172.16.16.129 (R4) |

#### 3.4 **Tabla de direccionamiento de enlaces**

Los dos /25 consumen todo el /24 asignado, por lo que no queda espacio para los enlaces punto a punto. Se utiliza un rango privado adicional, 10.6.0.0/24, subneteado en /30, y un /29 para el segmento de servidores.

| Enlace                      | Red          | Extremo A        | Extremo B              |
| --------------------------- | ------------ | ---------------- | ---------------------- |
| F-CORE ↔ R5                 | 10.6.0.0/30  | .1 (F-CORE)      | .2 (R5)                |
| F-CORE ↔ R1                 | 10.6.0.4/30  | .5 (F-CORE)      | .6 (R1)                |
| F-CORE ↔ R2                 | 10.6.0.8/30  | .9 (F-CORE)      | .10 (R2)               |
| R1 ↔ R3                     | 10.6.0.12/30 | .13 (R1)         | .14 (R3)               |
| R2 ↔ R4                     | 10.6.0.16/30 | .17 (R2)         | .18 (R4)               |
| R5 ↔ Segmento de servidores | 10.6.1.0/29  | .1 (R5, gateway) | .2 (Server0, estática) |

El servidor usa IP estática, ya que el requisito de DHCP aplica a los dispositivos finales (hosts).

#### 3.5 **Justificación de decisiones de diseño**

- **Árbol:** cada nodo tiene un solo padre. Los EtherChannel entre pares de switches cuentan como un único enlace lógico y no rompen la topología.
- **LACP en acceso:** los 2 enlaces LACP se colocan entre switches 2960, porque en Packet Tracer el EtherChannel solo está disponible en switches, no en routers.
- **Switch 2960-24TT:** es el switch administrable más económico que soporta LACP, suficiente para la capa de acceso.
- **Router R5 entre el core y el servidor:** crea un segmento de servicios aislado (estilo DMZ), con un punto donde aplicar políticas de acceso hacia el servidor. En producción se sustituiría por un firewall y se duplicaría para evitar un punto único de falla.
- **Switch de servidores con VLAN 10 (SERVIDORES):** permite agregar más servidores sin modificar el direccionamiento (la subred 10.6.1.0/29 tiene 5 direcciones libres además del gateway).
- **Routers R1 y R2:** capa de agregación por rama, que permite aislar cada departamento y crecer por ramas independientes.
- **Un departamento por rama:** simplifica la aplicación de las ACLs.

#### 3.6 **Configuración por dispositivo**

_(Pendiente: un bloque de código por dispositivo.)_

- [ ] LACP en Switch1/Switch3 y Switch2/Switch0
- [ ] VLAN 10 en el switch de servidores
- [ ] IPs de enlaces y OSPF área 0 en F-CORE y R1 a R5
- [ ] `ip helper-address` en R3 y R4 hacia el servidor DHCP de AndinaCom

#### 3.7 **Servicios (DNS y HTTP/HTTPS)**

_(Pendiente: configuración del servidor y dominio `www.proyecto2_202000806.com`.)_

#### 3.8 **Verificación**

_(Pendiente: capturas de `show ip ospf neighbor`, `show etherchannel summary`, ping y acceso HTTP.)_

---

### 4. **ISP 2: AndinaCom**

#### 4.1 **Requisitos**

- Topología de modelo jerárquico.
- Protocolo de enrutamiento interno: OSPF (área 0).
- Mínimo 5 routers, 5 hosts y 2 enlaces LACP.
- Dos departamentos (Ventas y Facturación), con subredes de tamaño a discreción.
- Redundancia de capa 3 obligatoria (HSRP).
- Provee el servicio DHCP a toda la topología.

#### 4.2 **Topología**

La red se organiza en un core, un bloque de servicios y dos bloques de usuarios independientes (uno por departamento). Cada bloque tiene gateway redundante con HSRP, agregación de capa 2 y acceso con enlaces LACP.

![imagen](/Proyecto2/img/topologia4.png)

| Nivel               | Equipos                        | Función                                         |
| ------------------- | ------------------------------ | ----------------------------------------------- |
| Core / borde        | Multilayer Switch1 (3650-24PS) | BGP hacia los otros ISPs y enrutamiento central |
| Distribución L3     | R8, R9, R10                    | Gateways de los departamentos con HSRP          |
| Bloque de servicios | R6, R7, SW6, Server-PT         | Gateway redundante (HSRP) y servidor DHCP       |
| Agregación L2       | SW7, SW8 (2960-24TT)           | Reúnen los switches de acceso hacia los routers |
| Acceso              | SW9, SW10 (2960-24TT)          | Conectan los hosts de cada departamento         |

Total: 5 routers (R6 a R10), 6 hosts (PC6 a PC11) y 2 enlaces LACP (SW7↔SW9 y SW8↔SW10).

**Justificación de decisiones de diseño**

- **Bloques independientes por departamento:** aíslan el tráfico y simplifican las ACLs. Cada departamento crece sin afectar al otro.
- **R9 compartido:** es el router de respaldo de ambos departamentos (standby en Ventas y en Facturación), con una interfaz por subred.
- **Bloque de servicios dedicado:** el servidor DHCP tiene su propio segmento con gateway redundante (R6 y R7), conectado directo al core.
- **LACP en el acceso:** los EtherChannel se colocan entre switches 2960, porque en Packet Tracer solo están disponibles en switches.
- **Limitaciones reconocidas del prototipo:** el core, los switches de agregación y el servidor son puntos únicos de falla. En producción el core sería un par, la agregación estaría duplicada y el servidor tendría doble tarjeta de red, con DHCP en failover.

#### 4.3 **Subneteo de departamentos (172.16.26.0/24)**

El tamaño de las subredes queda a discreción. Se usan subredes /26 (62 hosts útiles) para los departamentos y un /29 para el segmento de servidores, y se reservan los últimos bloques para los enlaces punto a punto.

| Uso                    | Red                        | Máscara         | Rango útil  | Broadcast     |
| ---------------------- | -------------------------- | --------------- | ----------- | ------------- |
| Ventas                 | 172.16.26.0/26             | 255.255.255.192 | .1 – .62    | 172.16.26.63  |
| Facturación            | 172.16.26.64/26            | 255.255.255.192 | .65 – .126  | 172.16.26.127 |
| Servidores             | 172.16.26.128/29           | 255.255.255.248 | .129 – .134 | 172.16.26.135 |
| Enlaces core ↔ routers | 172.16.26.192/28 (5 × /30) | 255.255.255.252 | ver 4.4     | —             |

#### 4.4 **Tabla de direccionamiento**

**Enlaces entre el core y los routers**

| Enlace     | Red              | Core | Router |
| ---------- | ---------------- | ---- | ------ |
| Core ↔ R6  | 172.16.26.192/30 | .193 | .194   |
| Core ↔ R7  | 172.16.26.196/30 | .197 | .198   |
| Core ↔ R8  | 172.16.26.200/30 | .201 | .202   |
| Core ↔ R9  | 172.16.26.204/30 | .205 | .206   |
| Core ↔ R10 | 172.16.26.208/30 | .209 | .210   |

**Gateways redundantes (HSRP)**

| Segmento    | Red              | IP virtual (gateway) | Router | IP real       |
| ----------- | ---------------- | -------------------- | ------ | ------------- |
| Ventas      | 172.16.26.0/26   | 172.16.26.1          | R8     | 172.16.26.2   |
| Ventas      | 172.16.26.0/26   | 172.16.26.1          | R9     | 172.16.26.3   |
| Facturación | 172.16.26.64/26  | 172.16.26.65         | R10    | 172.16.26.66  |
| Facturación | 172.16.26.64/26  | 172.16.26.65         | R9     | 172.16.26.67  |
| Servidores  | 172.16.26.128/29 | 172.16.26.129        | R6     | 172.16.26.130 |
| Servidores  | 172.16.26.128/29 | 172.16.26.129        | R7     | 172.16.26.131 |

**Servidor**

| Dispositivo      | IP                       | Máscara         | Gateway       |
| ---------------- | ------------------------ | --------------- | ------------- |
| Server-PT (DHCP) | 172.16.26.132 (estática) | 255.255.255.248 | 172.16.26.129 |

El servidor usa IP estática, ya que el requisito de DHCP aplica a los dispositivos finales.

**Enlace BGP del core de AndinaCom**

| Enlace               | Red            | Core de AndinaCom | Vecino         |
| -------------------- | -------------- | ----------------- | -------------- |
| AndinaCom ↔ FibraNet | 192.168.6.0/30 | .2                | .1 (FibraNet)  |
| AndinaCom ↔ MoonLink | 192.168.6.8/30 | .9                | .10 (MoonLink) |

Sistema autónomo: AS 65002 (Router ID 2.2.2.2).

#### 4.5 **Configuración por dispositivo**

_(Pendiente.)_

#### 4.6 **DHCP y HSRP**

_(Pendiente: DHCP centralizado y prueba de conmutación HSRP.)_

#### 4.7 **Verificación**

_(Pendiente.)_

---

### 5. **ISP 3: MoonLink**

#### 5.1 **Requisitos**

- Topología hub-and-spoke.
- Protocolo de enrutamiento interno: EIGRP (AS 100).
- Mínimo 5 routers, 5 hosts y 2 enlaces LACP.
- Dos departamentos (Soporte y Seguridad), con subredes para hasta 45 hosts.
- Al menos un router inalámbrico que proporcione servicio WiFi (puede contarse dentro de los 5 routers).

#### 5.2 **Topología**

La red usa una topología hub-and-spoke con **dos hubs redundantes**. Cada spoke se conecta a ambos hubs y no existen enlaces directos entre spokes, por lo que todo el tráfico entre ellos pasa por un hub. El switch multicapa M-CORE es el equipo de borde BGP.

![imagen](/Proyecto2/img/topologia5.png)

| Dispositivo | Modelo    | Función                                                      |
| ----------- | --------- | ------------------------------------------------------------ |
| M-CORE      | 3650-24PS | Borde BGP y enlace hacia ambos hubs                          |
| R11, R12    | 2911      | Hubs redundantes de la topología                             |
| R13         | 2911      | Spoke y gateway de Soporte                                   |
| R14         | 2911      | Spoke y gateway de Seguridad                                 |
| R15         | WRT300N   | Router inalámbrico (punto de acceso en la subred de Soporte) |
| SW11 / SW12 | 2960-24TT | Switches de Soporte (par LACP)                               |
| SW13 / SW14 | 2960-24TT | Switches de Seguridad (par LACP)                             |

Total: 5 routers (M-HUB1, M-HUB2, M-R1, M-R2 y M-WIFI), 6 PCs más un cliente WiFi, y 2 enlaces LACP (SW-S1↔SW-S2 y SW-G1↔SW-G2).

**Conexiones de los routers**

| Router | Gi0/0  | Gi0/1  | Gi0/2                 |
| ------ | ------ | ------ | --------------------- |
| M-HUB1 | M-CORE | M-R1   | M-R2                  |
| M-HUB2 | M-CORE | M-R1   | M-R2                  |
| M-R1   | M-HUB1 | M-HUB2 | LAN Soporte (SW-S1)   |
| M-R2   | M-HUB1 | M-HUB2 | LAN Seguridad (SW-G1) |

**Justificación de decisiones de diseño**

- **Dos hubs:** eliminan el punto único de falla del hub. EIGRP reparte la carga entre ambos y converge por el otro si uno falla.
- **Spokes sin enlaces entre sí:** mantienen la topología hub-and-spoke, ya que los spokes solo se comunican a través de un hub.
- **`eigrp stub connected` en los spokes:** evita que sirvan de tránsito entre los hubs y acelera la convergencia.
- **Router inalámbrico como punto de acceso:** se conecta por un puerto LAN al switch de Soporte, con su DHCP desactivado, de modo que los clientes WiFi reciben IP del servidor DHCP de AndinaCom. Se eligió el WRT300N por su compatibilidad con los clientes inalámbricos de Packet Tracer (2.4 GHz).
- **LACP en el acceso:** los EtherChannel se colocan entre switches 2960, porque en Packet Tracer solo están disponibles en switches.
- **Limitaciones reconocidas del prototipo:** el core, cada spoke y los switches de cada departamento son puntos únicos de falla. En producción el core sería un par y cada departamento tendría gateway redundante.

#### 5.3 **Subneteo de departamentos (172.16.32.0/24)**

Se requieren subredes para hasta 45 hosts. Se necesitan 6 bits de host (2⁶ − 2 = 62 útiles), por lo que corresponde una máscara /26. El WiFi comparte la subred de Soporte.

| Departamento           | Red              | Máscara         | Rango útil  | Broadcast     | Gateway             |
| ---------------------- | ---------------- | --------------- | ----------- | ------------- | ------------------- |
| Soporte (incluye WiFi) | 172.16.32.0/26   | 255.255.255.192 | .1 – .62    | 172.16.32.63  | 172.16.32.1 (M-R1)  |
| Seguridad              | 172.16.32.64/26  | 255.255.255.192 | .65 – .126  | 172.16.32.127 | 172.16.32.65 (M-R2) |
| Libre (crecimiento)    | 172.16.32.128/26 | 255.255.255.192 | .129 – .190 | 172.16.32.191 | —                   |

Los enlaces punto a punto usan /30 dentro del mismo /24, a partir de 172.16.32.192.

#### 5.4 **Tabla de direccionamiento**

**Enlaces entre dispositivos**

| Enlace          | Red              | Extremo A     | Extremo B     |
| --------------- | ---------------- | ------------- | ------------- |
| M-CORE ↔ M-HUB1 | 172.16.32.192/30 | .193 (M-CORE) | .194 (M-HUB1) |
| M-CORE ↔ M-HUB2 | 172.16.32.196/30 | .197 (M-CORE) | .198 (M-HUB2) |
| M-HUB1 ↔ M-R1   | 172.16.32.200/30 | .201 (M-HUB1) | .202 (M-R1)   |
| M-HUB1 ↔ M-R2   | 172.16.32.204/30 | .205 (M-HUB1) | .206 (M-R2)   |
| M-HUB2 ↔ M-R1   | 172.16.32.208/30 | .209 (M-HUB2) | .210 (M-R1)   |
| M-HUB2 ↔ M-R2   | 172.16.32.212/30 | .213 (M-HUB2) | .214 (M-R2)   |

**Interfaces LAN y dispositivos de infraestructura**

| Dispositivo          | Red             | IP           | Observación                                    |
| -------------------- | --------------- | ------------ | ---------------------------------------------- |
| M-R1 (LAN Soporte)   | 172.16.32.0/26  | 172.16.32.1  | Gateway de Soporte                             |
| M-WIFI (WRT300N)     | 172.16.32.0/26  | 172.16.32.62 | Punto de acceso, IP de administración estática |
| M-R2 (LAN Seguridad) | 172.16.32.64/26 | 172.16.32.65 | Gateway de Seguridad                           |

Los hosts de ambos departamentos y los clientes WiFi obtienen su IP por DHCP.

**Enlaces BGP del core de MoonLink**

| Enlace               | Red            | Core de MoonLink | Vecino         |
| -------------------- | -------------- | ---------------- | -------------- |
| MoonLink ↔ FibraNet  | 192.168.6.4/30 | .6               | .5 (FibraNet)  |
| MoonLink ↔ AndinaCom | 192.168.6.8/30 | .10              | .9 (AndinaCom) |

Sistema autónomo: AS 65003 (Router ID 3.3.3.3). Redes anunciadas: 172.16.32.0/24, apoyada en una ruta estática de resumen hacia `Null0`.

#### 5.5 **Configuración por dispositivo**

_(Pendiente.)_

#### 5.6 **Red inalámbrica y EIGRP**

_(Pendiente.)_

#### 5.7 **Verificación**

_(Pendiente.)_

---

### 6. **Interconexión de ISPs (BGP)**

#### 6.1 **Descripción**

Los tres ISPs (FibraNet GT, AndinaCom y MoonLink) se interconectan mediante BGP entre sus switches multicapa de borde (Cisco 3650-24PS), usando fibra óptica. Los tres equipos forman un triángulo, de modo que cada ISP tiene un vecino BGP directo con los otros dos.

![imagen](/Proyecto2/img/topologia2.png)

#### 6.2 **Subneteo de la red de interconexión**

| Enlace               | Red            | Extremo A      | Extremo B      |
| -------------------- | -------------- | -------------- | -------------- |
| FibraNet ↔ AndinaCom | 192.168.6.0/30 | .1 (FibraNet)  | .2 (AndinaCom) |
| FibraNet ↔ MoonLink  | 192.168.6.4/30 | .5 (FibraNet)  | .6 (MoonLink)  |
| AndinaCom ↔ MoonLink | 192.168.6.8/30 | .9 (AndinaCom) | .10 (MoonLink) |

#### 6.3 **Sistemas autónomos**

| ISP         | AS    | Router ID |
| ----------- | ----- | --------- |
| FibraNet GT | 65001 | 1.1.1.1   |
| AndinaCom   | 65002 | 2.2.2.2   |
| MoonLink    | 65003 | 3.3.3.3   |

#### 6.4 **Redes anunciadas**

Como las LAN internas están en subredes más específicas dentro de OSPF/EIGRP, se agrega una ruta estática de resumen hacia `Null0` para que el comando `network` de BGP encuentre la ruta exacta en la tabla de enrutamiento.

| ISP         | Redes anunciadas            |
| ----------- | --------------------------- |
| FibraNet GT | 172.16.16.0/24, 10.6.0.0/23 |
| AndinaCom   | 172.16.26.0/24              |
| MoonLink    | 172.16.32.0/24              |

#### 6.5 **Configuración y verificación**

_(Pendiente: configuración de cada 3650 de borde y salida de `show ip bgp summary`.)_

---

### 7. **Reglas de comunicación (ACLs)**

#### 7.1 **Tabla de reglas**

_(Pendiente: ACLs extendidas con la interfaz y el sentido (in/out) donde se aplica cada una, y su justificación.)_

#### 7.2 **Verificación**

_(Pendiente: pings y accesos HTTP entre todos los pares de departamentos.)_

---

### 8. **Pruebas integrales**

_(Pendiente: pings entre ISPs, acceso al dominio desde cualquier dispositivo, conmutación HSRP y DHCP en toda la topología.)_

---

### 9. **Análisis en modo simulación (Packet Tracer)**

_(Pendiente: DHCP DORA, DNS, HTTP/HTTPS, OSPF/EIGRP y BGP.)_

---

### 10. **Análisis de tráfico con Wireshark**

_(Pendiente: explicación capa por capa de al menos 3 protocolos: TCP handshake, DNS, HTTP vs HTTPS e ICMP.)_

---

### 11. **Anexos**

- Configuraciones completas por dispositivo
- Estructura del repositorio
- Costos estimados (opcional)
