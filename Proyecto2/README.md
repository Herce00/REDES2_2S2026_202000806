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

La raíz del árbol es el switch multicapa MS1 (3560-24PS), que actúa como core de la red. De MS1 cuelgan tres ramas: la frontera hacia los otros ISP (F-BORDER), la rama de MS2 y la rama de MS3. Cada nodo tiene un único padre, lo que mantiene la topología en árbol.

- **Frontera BGP:** MS1 → F-BORDER → enlace hacia los otros ISP.
- **Rama de servicios y Administración:** MS1 → MS2, de donde salen dos ramas:
  - MS2 → SW5 → Server0 (DNS y HTTP/HTTPS).
  - MS2 → MS4 → SW1 y SW2 → PC3, PC5 y PC2 (Administración).
- **Rama de Atención al cliente:** MS1 → MS3 → MS5 → SW3 y SW4 → PC0, PC4 y PC1.

![imagen](/Proyecto2/img/topologia3.png)

| Dispositivo                   | Modelo    | Función                                                                                          |
| ----------------------------- | --------- | ------------------------------------------------------------------------------------------------ |
| MS1                           | 3560-24PS | Core y raíz del árbol, une todas las ramas                                                       |
| F-BORDER (Multilayer Switch0) | 3650-24PS | Frontera de la red, único equipo que habla BGP con los otros ISP                                 |
| MS2                           | 3560-24PS | Distribución de Administración y del segmento de servidores; gateway de servidores (SVI VLAN 10) |
| MS3                           | 3560-24PS | Distribución de la rama de Atención al cliente                                                   |
| MS4                           | 3560-24PS | Gateway de Administración (SVI VLAN 20)                                                          |
| MS5                           | 3560-24PS | Gateway de Atención al cliente (SVI VLAN 30)                                                     |
| SW1 / SW2                     | 2960-24TT | Acceso Administración                                                                            |
| SW3 / SW4                     | 2960-24TT | Acceso Atención al cliente                                                                       |
| SW5                           | 2960-24TT | Acceso del segmento de servidores (VLAN 10)                                                      |
| Server0                       | Server-PT | DNS y HTTP/HTTPS                                                                                 |

Total: 6 switches multicapa que cumplen el rol de routers (MS1 a MS5 y F-BORDER), 6 hosts (PC0 a PC5) más el servidor, y 6 enlaces EtherChannel LACP.

#### 3.3 **Subneteo de departamentos (172.16.16.0/24)**

Se requieren subredes para hasta 75 hosts. Se necesitan 7 bits de host (2⁷ − 2 = 126 útiles), por lo que corresponde una máscara /25.

| Departamento        | VLAN | Red              | Máscara         | Rango útil  | Broadcast     | Gateway             |
| ------------------- | ---- | ---------------- | --------------- | ----------- | ------------- | ------------------- |
| Administración      | 20   | 172.16.16.0/25   | 255.255.255.128 | .1 – .126   | 172.16.16.127 | 172.16.16.1 (MS4)   |
| Atención al cliente | 30   | 172.16.16.128/25 | 255.255.255.128 | .129 – .254 | 172.16.16.255 | 172.16.16.129 (MS5) |

Los hosts de ambos departamentos obtienen su IP por DHCP. El servidor DHCP está en AndinaCom, por lo que MS4 y MS5 configuran `ip helper-address` en sus SVIs.

#### 3.4 **Tabla de direccionamiento de enlaces (10.1.0.0/24)**

Los dos /25 consumen todo el /24 asignado, por lo que no queda espacio para los enlaces punto a punto. Se utiliza una red privada adicional, 10.1.0.0/24, subneteada en /30 para los enlaces entre multicapa y en /29 para el segmento de servidores.

| Enlace                 | Red          | Extremo A                       | Extremo B               |
| ---------------------- | ------------ | ------------------------------- | ----------------------- |
| MS1 ↔ F-BORDER         | 10.1.0.0/30  | .1 (MS1)                        | .2 (F-BORDER)           |
| MS1 ↔ MS2              | 10.1.0.4/30  | .5 (MS1)                        | .6 (MS2)                |
| MS1 ↔ MS3              | 10.1.0.8/30  | .9 (MS1)                        | .10 (MS3)               |
| MS2 ↔ MS4              | 10.1.0.12/30 | .13 (MS2)                       | .14 (MS4)               |
| MS3 ↔ MS5              | 10.1.0.16/30 | .17 (MS3)                       | .18 (MS5)               |
| Segmento de servidores | 10.1.0.24/29 | .25 (MS2, gateway, SVI VLAN 10) | .26 (Server0, estática) |

El servidor usa IP estática, ya que el requisito de DHCP aplica a los dispositivos finales (hosts).

#### 3.5 **Enlaces LACP (EtherChannel)**

Todos los enlaces entre equipos de distribución se implementan como pares de cables agrupados con LACP (`channel-group X mode active`), lo que da más ancho de banda y tolerancia a la falla de un cable.

| Port-channel | Enlace         | Tipo                                      |
| ------------ | -------------- | ----------------------------------------- |
| Po1          | MS1 ↔ F-BORDER | Capa 3 (puerto enrutado, `no switchport`) |
| Po2          | MS1 ↔ MS2      | Capa 3                                    |
| Po3          | MS1 ↔ MS3      | Capa 3                                    |
| Po4          | MS2 ↔ MS4      | Capa 3                                    |
| Po5          | MS3 ↔ MS5      | Capa 3                                    |
| Po6          | MS2 ↔ SW5      | Capa 2 (acceso a VLAN 10)                 |

Se verifican con `show etherchannel summary`, donde cada Port-channel debe aparecer en estado `SU` y sus puertos en estado `P`.

#### 3.6 **OSPF (área 0)**

Todas las interfaces de capa 3 (Port-channels enrutados y SVIs) pertenecen al área 0. Las SVIs de usuarios (VLAN 20 y VLAN 30) se declaran como `passive-interface` para no enviar hellos hacia los hosts. F-BORDER es el equipo que conecta FibraNet GT con el resto de ISPs: anuncia por BGP las redes 172.16.16.0/24 y 10.1.0.0/24, y entrega al interior de OSPF las rutas aprendidas de los otros ISP.

#### 3.7 **Justificación de decisiones de diseño**

- **Árbol:** cada nodo tiene un solo padre. Los EtherChannel entre pares de equipos cuentan como un único enlace lógico y no rompen la topología.
- **MS1 como core:** concentra las tres ramas, por lo que todo el tráfico entre departamentos y hacia el exterior pasa por un solo punto, donde se aplican las ACLs y se controla el tráfico.
- **F-BORDER separado del core:** aísla BGP y la conexión con los otros ISP del resto de la red interna. Un problema en la frontera no afecta el enrutamiento interno y se puede reemplazar sin tocar el core.
- **Multicapa como routers:** los switches 3560/3650 enrutan en hardware y soportan OSPF, SVIs y Port-channels de capa 3, lo que reduce costos frente a tener routers y switches separados.
- **LACP en la distribución:** protege los enlaces críticos del árbol. La caída de un cable no interrumpe el servicio.
- **Switch 2960-24TT en acceso:** es el switch administrable más económico y suficiente para conectar usuarios y servidor.
- **Gateways en MS4 y MS5:** cada departamento tiene su gateway en la hoja de su rama, por lo que el tráfico entre departamentos sube hasta el core.
- **Segmento de servidores en VLAN 10:** permite agregar más servidores sin modificar el direccionamiento (la subred 10.1.0.24/29 tiene 4 direcciones libres además del gateway y Server0).
- **Un departamento por rama:** simplifica la aplicación de las ACLs.
- **Mejora en producción:** MS1 y F-BORDER son puntos únicos de falla. En un entorno real se duplicarían.

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

La red sigue el modelo jerárquico de tres capas: core, distribución y acceso. El switch multicapa A-BORDER (3650-24PS) es la frontera que conecta con los otros ISP mediante BGP. MS6 es el core. MS7 y MS8 son los bloques de distribución, uno por departamento, y cada bloque tiene dos routers 2911 que funcionan como gateway redundante con HSRP. SW7 y SW8 son la capa de acceso, y el servidor DHCP cuelga de R1 a través de SW6.

- **Ventas:** MS6 → MS7 → R1 y R2 → SW7 → PC9, PC10 y PC11.
- **Facturación:** MS6 → MS8 → R3 y R4 → SW8 → PC6, PC7 y PC8.
- **Servicios:** MS6 → SW6 → Server DHCP.

![imagen](/Proyecto2/img/topologia4.png)

| Capa          | Equipos                               | Modelo                | Función                                                                                                                    |
| ------------- | ------------------------------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Frontera      | A-BORDER (Multilayer Switch1)         | 3650-24PS             | Único equipo que habla BGP con FibraNet GT y MoonLink                                                                      |
| Core          | MS6                                   | 3560-24PS             | Une la frontera con los bloques de distribución y el bloque de servicios; gateway del segmento de servidores (SVI VLAN 10) |
| Distribución  | MS7 (Ventas), MS8 (Facturación)       | 3560-24PS             | Concentran los dos routers gateway de cada departamento                                                                    |
| Gateways HSRP | R1, R2 (Ventas), R3, R4 (Facturación) | 2911                  | Gateway redundante de cada departamento                                                                                    |
| Acceso        | SW7, SW8                              | 2960-24TT             | Conectan los hosts de cada departamento                                                                                    |
| Servicios     | SW6, Server DHCP                      | 2960-24TT / Server-PT | Bloque de servicios conectado directamente al core                                                                         |

Total: 8 equipos de capa 3 (4 routers 2911 y 4 switches multicapa), 6 hosts (PC6 a PC11) más el servidor DHCP, y 3 enlaces EtherChannel LACP (ver 4.5).

**Justificación de decisiones de diseño**

- **Modelo jerárquico:** cada capa tiene una función definida (frontera, core, distribución, acceso), lo que facilita el crecimiento por bloques y el diagnóstico de fallas.
- **Bloque de servicios en el core:** el servidor DHCP cuelga de MS6 a través de SW6, en su propia subred. Así no depende de ningún router de departamento y la falla de R1 a R4 no afecta el servicio DHCP de ninguno de los tres ISP.
- **Par de routers con HSRP por departamento:** si el router activo falla, el standby toma la IP virtual y los hosts siguen usando el mismo gateway sin reconfigurarse.
- **LACP en el troncal:** los enlaces entre A-BORDER, MS6, MS7 y MS8 son EtherChannel, por lo que la caída de un cable no interrumpe el servicio.
- **Segmento de servidores propio:** el servidor DHCP está aislado en su propia subred, detrás de R1, y atiende los pools de todos los ISP.

#### 4.3 **Subneteo de departamentos (172.16.26.0/24)**

El tamaño de las subredes queda a discreción. Se usan subredes /26 (62 hosts útiles) para los departamentos, un /29 para el segmento de servidores y un /27 para los enlaces punto a punto (7 enlaces /30).

| Uso                   | Red                  | Máscara         | Rango útil  | Broadcast     |
| --------------------- | -------------------- | --------------- | ----------- | ------------- |
| Ventas                | 172.16.26.0/26       | 255.255.255.192 | .1 – .62    | 172.16.26.63  |
| Facturación           | 172.16.26.64/26      | 255.255.255.192 | .65 – .126  | 172.16.26.127 |
| Servidores            | 172.16.26.128/29     | 255.255.255.248 | .129 – .134 | 172.16.26.135 |
| Libre                 | 172.16.26.136 – .191 | —               | —           | —             |
| Enlaces punto a punto | 172.16.26.192/27     | 255.255.255.224 | ver 4.4     | 172.16.26.223 |
| Libre                 | 172.16.26.224 – .255 | —               | —           | —             |

#### 4.4 **Tabla de direccionamiento**

**Enlaces punto a punto (172.16.26.192/27, subneteado en /30)**

| Enlace         | Red              | Extremo A       | Extremo B  |
| -------------- | ---------------- | --------------- | ---------- |
| A-BORDER ↔ MS6 | 172.16.26.192/30 | .193 (A-BORDER) | .194 (MS6) |
| MS6 ↔ MS7      | 172.16.26.196/30 | .197 (MS6)      | .198 (MS7) |
| MS6 ↔ MS8      | 172.16.26.200/30 | .201 (MS6)      | .202 (MS8) |
| MS7 ↔ R1       | 172.16.26.204/30 | .205 (MS7)      | .206 (R1)  |
| MS7 ↔ R2       | 172.16.26.208/30 | .209 (MS7)      | .210 (R2)  |
| MS8 ↔ R3       | 172.16.26.212/30 | .213 (MS8)      | .214 (R3)  |
| MS8 ↔ R4       | 172.16.26.216/30 | .217 (MS8)      | .218 (R4)  |

Queda libre el bloque 172.16.26.220/30 para crecimiento.

**Gateways redundantes (HSRP)**

| Segmento    | Red             | IP virtual (gateway) | Router | IP real      | Prioridad     |
| ----------- | --------------- | -------------------- | ------ | ------------ | ------------- |
| Ventas      | 172.16.26.0/26  | 172.16.26.1          | R1     | 172.16.26.2  | 110 (activo)  |
| Ventas      | 172.16.26.0/26  | 172.16.26.1          | R2     | 172.16.26.3  | 100 (standby) |
| Facturación | 172.16.26.64/26 | 172.16.26.65         | R3     | 172.16.26.66 | 110 (activo)  |
| Facturación | 172.16.26.64/26 | 172.16.26.65         | R4     | 172.16.26.67 | 100 (standby) |

**Segmento de servidores**

| Dispositivo                  | IP                       | Máscara         | Gateway             |
| ---------------------------- | ------------------------ | --------------- | ------------------- |
| MS6 (SVI VLAN 10, hacia SW6) | 172.16.26.129            | 255.255.255.248 | —                   |
| Server DHCP                  | 172.16.26.130 (estática) | 255.255.255.248 | 172.16.26.129 (MS6) |

El servidor usa IP estática, ya que el requisito de DHCP aplica a los dispositivos finales.

**Enlaces BGP de A-BORDER**

| Enlace               | Red            | A-BORDER | Vecino         |
| -------------------- | -------------- | -------- | -------------- |
| AndinaCom ↔ FibraNet | 192.168.6.0/30 | .2       | .1 (FibraNet)  |
| AndinaCom ↔ MoonLink | 192.168.6.8/30 | .9       | .10 (MoonLink) |

Sistema autónomo: AS 65002 (Router ID 2.2.2.2). A-BORDER anuncia por BGP la red 172.16.26.0/24. Para que BGP acepte el anuncio, se declara en A-BORDER una ruta estática de resumen hacia Null0 (`ip route 172.16.26.0 255.255.255.0 Null0`). Las rutas más específicas aprendidas por OSPF siguen teniendo prioridad.

#### 4.5 **Enlaces LACP (EtherChannel)**

Los enlaces del troncal se implementan como pares de cables agrupados con LACP (`channel-group X mode active`).

| Port-channel | Enlace         | Tipo                                      |
| ------------ | -------------- | ----------------------------------------- |
| Po1          | A-BORDER ↔ MS6 | Capa 3 (puerto enrutado, `no switchport`) |
| Po2          | MS6 ↔ MS7      | Capa 3                                    |
| Po3          | MS6 ↔ MS8      | Capa 3                                    |

Se verifican con `show etherchannel summary`, donde cada Port-channel debe aparecer en estado `SU` y sus puertos en estado `P`.

#### 4.6 **HSRP**

Cada departamento tiene un grupo HSRP configurado en las interfaces de R1 a R4 que miran hacia el switch de acceso:

- **Grupo 1 (Ventas):** R1 (activo, prioridad 110) y R2 (standby, prioridad 100), IP virtual 172.16.26.1.
- **Grupo 2 (Facturación):** R3 (activo, prioridad 110) y R4 (standby, prioridad 100), IP virtual 172.16.26.65.
- En los routers activos se configura `standby X preempt`, para que recuperen el rol al volver a estar disponibles.
- Los hosts reciben por DHCP la IP virtual como gateway.

**Verificación:** `show standby brief` en cada router. La prueba de conmutación consiste en hacer un ping continuo desde un host y apagar la interfaz del router activo (por ejemplo R3). R4 pasa a estado Active y el ping se recupera tras pocos paquetes perdidos.

#### 4.7 **OSPF (área 0)**

Todas las interfaces de capa 3 pertenecen al área 0. Las interfaces de R1 a R4 que miran hacia los switches de acceso se declaran como `passive-interface`, igual que la SVI de la VLAN 10 en MS6.

#### 4.8 **DHCP centralizado**

El servidor DHCP de AndinaCom atiende a toda la topología. Los routers que hacen de gateway (R1 a R4 aquí, y los gateways de los otros ISP) configuran `ip helper-address 172.16.26.130` en las interfaces donde hay hosts.

| Pool                          | Red              | Gateway entregado | IP inicial | Máx. usuarios | DNS       |
| ----------------------------- | ---------------- | ----------------- | ---------- | ------------- | --------- |
| Ventas                        | 172.16.26.0/26   | 172.16.26.1       | .4         | 59            | 10.1.0.26 |
| Facturación                   | 172.16.26.64/26  | 172.16.26.65      | .68        | 59            | 10.1.0.26 |
| FibraNet: Administración      | 172.16.16.0/25   | 172.16.16.1       | .2         | 125           | 10.1.0.26 |
| FibraNet: Atención al cliente | 172.16.16.128/25 | 172.16.16.129     | .130       | 125           | 10.1.0.26 |

La IP inicial deja fuera las direcciones de los gateways y de los routers HSRP. El DNS apunta al servidor de FibraNet GT. Los pools de MoonLink se agregan con el mismo formato.

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

La red usa una topología hub-and-spoke con **dos hubs redundantes** (MS9 y MS10). Cada spoke (MS11 y MS12) se conecta a ambos hubs y no existen enlaces directos entre spokes, por lo que todo el tráfico entre ellos pasa por un hub. El switch multicapa M-BORDER es la frontera de la red y el único equipo que habla BGP con los otros ISP.

- **Frontera BGP:** M-BORDER → enlaces hacia FibraNet GT y AndinaCom.
- **Hubs:** M-BORDER → MS9 y MS10.
- **Soporte:** MS9 y MS10 → MS11 → SW11 → PC12, PC13 y R5 (WRT300N) con Laptop0 por WiFi.
- **Seguridad:** MS9 y MS10 → MS12 → SW13 → PC14, PC15 y PC16.

![imagen](/Proyecto2/img/topologia5.png)

| Dispositivo | Modelo    | Función                                                          |
| ----------- | --------- | ---------------------------------------------------------------- |
| M-BORDER    | 3650-24PS | Frontera de la red, único equipo que habla BGP con los otros ISP |
| MS9, MS10   | 3560-24PS | Hubs redundantes de la topología                                 |
| MS11        | 3560-24PS | Spoke y gateway de Soporte (SVI VLAN 40)                         |
| MS12        | 3560-24PS | Spoke y gateway de Seguridad (SVI VLAN 50)                       |
| SW11        | 2960-24TT | Acceso de Soporte                                                |
| SW13        | 2960-24TT | Acceso de Seguridad                                              |
| R5          | WRT300N   | Router inalámbrico, punto de acceso WiFi en la subred de Soporte |
| Laptop0     | Laptop-PT | Cliente inalámbrico de Soporte                                   |

Total: 5 switches multicapa que cumplen el rol de routers (M-BORDER, MS9, MS10, MS11 y MS12) más el router inalámbrico R5, 6 hosts (PC12 a PC16 y Laptop0) y 2 enlaces EtherChannel LACP.

**Justificación de decisiones de diseño**

- **Dos hubs:** eliminan el punto único de falla del hub. Si uno cae, el tráfico converge por el otro. Cuando las métricas de ambos caminos coinciden, EIGRP reparte la carga entre los dos hubs.
- **Spokes sin enlaces entre sí:** mantienen la topología hub-and-spoke, ya que los spokes solo se comunican a través de un hub.
- **`eigrp stub connected` en los spokes:** evita que sirvan de tránsito entre los hubs y acelera la convergencia.
- **LACP en un enlace de cada spoke:** MS11 tiene un Port-channel hacia MS9 y MS12 uno hacia MS10, de modo que cada hub tiene un spoke con enlace reforzado y la caída de un cable no interrumpe el servicio.
- **Gateways en los spokes:** MS11 y MS12 enrutan las VLAN de cada departamento mediante SVIs, por lo que el tráfico entre departamentos sube hasta un hub, donde se aplican las políticas.
- **Router inalámbrico como punto de acceso:** se conecta por un puerto LAN al switch SW9 con su DHCP desactivado, de modo que los clientes WiFi reciben IP del servidor DHCP de AndinaCom. Se eligió el WRT300N por su compatibilidad con los clientes inalámbricos de Packet Tracer.
- **Un departamento por spoke:** simplifica la aplicación de las ACLs.
- **Limitaciones reconocidas del prototipo:** M-BORDER, los switches de acceso y cada spoke son puntos únicos de falla. En producción la frontera sería un par y cada departamento tendría gateway redundante.

#### 5.3 **Subneteo de departamentos (172.16.32.0/24)**

Se requieren subredes para hasta 45 hosts. Se necesitan 6 bits de host (2⁶ − 2 = 62 útiles), por lo que corresponde una máscara /26. El WiFi comparte la subred de Soporte.

| Departamento           | VLAN | Red              | Máscara         | Rango útil  | Broadcast     | Gateway             |
| ---------------------- | ---- | ---------------- | --------------- | ----------- | ------------- | ------------------- |
| Soporte (incluye WiFi) | 40   | 172.16.32.0/26   | 255.255.255.192 | .1 – .62    | 172.16.32.63  | 172.16.32.1 (MS11)  |
| Seguridad              | 50   | 172.16.32.64/26  | 255.255.255.192 | .65 – .126  | 172.16.32.127 | 172.16.32.65 (MS12) |
| Libre (crecimiento)    | —    | 172.16.32.128/26 | 255.255.255.192 | .129 – .190 | 172.16.32.191 | —                   |

Los enlaces punto a punto usan /30 dentro del mismo /24, a partir de 172.16.32.192.

#### 5.4 **Tabla de direccionamiento**

**Enlaces entre dispositivos (/30)**

| Enlace            | Red              | Extremo A       | Extremo B   |
| ----------------- | ---------------- | --------------- | ----------- |
| M-BORDER ↔ MS9    | 172.16.32.192/30 | .193 (M-BORDER) | .194 (MS9)  |
| M-BORDER ↔ MS10   | 172.16.32.196/30 | .197 (M-BORDER) | .198 (MS10) |
| MS9 ↔ MS11 (Po1)  | 172.16.32.200/30 | .201 (MS9)      | .202 (MS11) |
| MS9 ↔ MS12        | 172.16.32.204/30 | .205 (MS9)      | .206 (MS12) |
| MS10 ↔ MS11       | 172.16.32.208/30 | .209 (MS10)     | .210 (MS11) |
| MS10 ↔ MS12 (Po2) | 172.16.32.212/30 | .213 (MS10)     | .214 (MS12) |

Queda libre el rango 172.16.32.216 – .255 para crecimiento.

**Interfaces LAN y dispositivos de infraestructura**

| Dispositivo        | Red             | IP           | Observación                                    |
| ------------------ | --------------- | ------------ | ---------------------------------------------- |
| MS11 (SVI VLAN 40) | 172.16.32.0/26  | 172.16.32.1  | Gateway de Soporte                             |
| R5 (WRT300N)       | 172.16.32.0/26  | 172.16.32.62 | Punto de acceso, IP de administración estática |
| MS12 (SVI VLAN 50) | 172.16.32.64/26 | 172.16.32.65 | Gateway de Seguridad                           |

Los hosts de ambos departamentos y los clientes WiFi obtienen su IP por DHCP. La IP del router inalámbrico es de infraestructura y se excluye del pool.

**Enlaces BGP de M-BORDER**

| Enlace               | Red            | M-BORDER | Vecino         |
| -------------------- | -------------- | -------- | -------------- |
| MoonLink ↔ FibraNet  | 192.168.6.4/30 | .6       | .5 (FibraNet)  |
| MoonLink ↔ AndinaCom | 192.168.6.8/30 | .10      | .9 (AndinaCom) |

Sistema autónomo: AS 65003 (Router ID 3.3.3.3). M-BORDER anuncia por BGP la red 172.16.32.0/24, apoyada en una ruta estática de resumen hacia Null0 (`ip route 172.16.32.0 255.255.255.0 Null0`). Las rutas más específicas aprendidas por EIGRP siguen teniendo prioridad.

#### 5.5 **Enlaces LACP (EtherChannel)**

Los enlaces reforzados de cada spoke se implementan como pares de cables agrupados con LACP (`channel-group X mode active`).

| Port-channel | Enlace      | Tipo                                      |
| ------------ | ----------- | ----------------------------------------- |
| Po1          | MS9 ↔ MS11  | Capa 3 (puerto enrutado, `no switchport`) |
| Po2          | MS10 ↔ MS12 | Capa 3                                    |

Se verifican con `show etherchannel summary`, donde cada Port-channel debe aparecer en estado `SU` y sus puertos en estado `P`.

#### 5.6 **EIGRP (AS 100)**

- Todos los equipos de capa 3 (M-BORDER, MS9, MS10, MS11 y MS12) forman parte del AS 100, con `no auto-summary` y router-id propio.
- Las SVIs de usuarios (VLAN 40 y VLAN 50) se declaran como `passive-interface` para no enviar hellos hacia los hosts.
- MS11 y MS12 usan `eigrp stub connected`: solo anuncian sus redes conectadas y no sirven de tránsito entre hubs.
- M-BORDER entrega al interior de EIGRP una ruta por defecto (ruta estática `0.0.0.0/0` hacia Null0 redistribuida con `redistribute static`), lo que permite que los hosts de MoonLink alcancen las redes de los otros ISP aprendidas por BGP.

**Verificación:** `show ip eigrp neighbors` (cada equipo ve a sus vecinos), `show ip eigrp topology` y `show ip route eigrp`. Para probar la redundancia se apaga una interfaz hacia un hub y se verifica que el ping continúa por el otro.

#### 5.7 **Red inalámbrica**

El router inalámbrico R5 (WRT300N) se conecta por un puerto LAN a SW11, en la VLAN 40 de Soporte, y funciona como punto de acceso:

- Servidor DHCP desactivado, de modo que los clientes inalámbricos reciben IP del servidor DHCP de AndinaCom a través del `ip helper-address` de MS11.
- IP de administración estática 172.16.32.62, excluida del pool DHCP.
- Seguridad inalámbrica y SSID configurados desde la interfaz gráfica del dispositivo, ya que el WRT300N no tiene CLI.
- Laptop0 se asocia al SSID y obtiene IP, gateway (172.16.32.1) y DNS por DHCP.

#### 5.8 **DHCP centralizado**

Los hosts de MoonLink reciben su IP del servidor DHCP de AndinaCom. MS11 y MS12 configuran `ip helper-address 172.16.26.130` en sus SVIs (VLAN 40 y VLAN 50). Los pools correspondientes se agregan en el servidor:

| Pool                     | Red             | Gateway entregado | IP inicial | Máx. usuarios | DNS       |
| ------------------------ | --------------- | ----------------- | ---------- | ------------- | --------- |
| MoonLink: Soporte (WiFi) | 172.16.32.0/26  | 172.16.32.1       | .2         | 60            | 10.1.0.26 |
| MoonLink: Seguridad      | 172.16.32.64/26 | 172.16.32.65      | .66        | 61            | 10.1.0.26 |

El pool de Soporte termina en .61 para no entregar la dirección .62 del router inalámbrico. El DNS apunta al servidor de FibraNet GT.

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
