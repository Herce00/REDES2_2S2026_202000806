# **_Proyecto 1_**

## Chapin Red

### 2do Semestre del 2026

```js
Universidad San Carlos de Guatemala
Programador: César Rolando Hernández Palacios
Carne: 202000806
Correo: cesarhernandez0019@gmail.com
```

---


## Manual Tecnico



### 1.  **Implementacion de topologia**

#### 1.1 **Topologia completa**

![imagen](/Proyecto1/img/topologia1.png)

#### 1.2 **Conexión entre Edificios (Red MAN - Metropolitan Area Network)**

![imagen](/Proyecto1/img/topologia2.png)

#### 1.3 **Red del Edificio Izquierdo**

![imagen](/Proyecto1/img/topologia3.png)

#### 1.4 **Red del Edificio Derecho**

![imagen](/Proyecto1/img/topologia4.png)

---

### 2.  **Esquemas de vlans**

| VLAN ID | Nombre de la VLAN                    | Ubicación / Propósito   | Uso                                            |
| ------: | ------------------------------------ | ----------------------- | ---------------------------------------------- |
|  **10** | `VLAN_Naranja_EdificioIZQ_202000806` | Edificio Izquierdo      | Segmento Naranja                               |
|  **20** | `VLAN_Verde_EdificioIZQ_202000806`   | Edificio Izquierdo      | Segmento Verde                                 |
|  **30** | `VLAN_Naranja_EdificioDER_202000806` | Edificio Derecho        | Segmento Naranja                               |
|  **40** | `VLAN_Verde_EdificioDER_202000806`   | Edificio Derecho        | Segmento Verde                                 |
|  **99** | `VLAN_ADMIN_EdificioADM_202000806`   | Edificio Administrativo | Administración y gestión de la infraestructura |
|  **100**| `VLAN_GesitonDHCP_202000806`        | Edificio ServidoresDHCP | Administrar los servidores DHCP                |


### Criterio de numeración
La numeración fue definida de manera que exista una separación entre los identificadores, permitiendo agregar nuevas VLAN posteriormente sin necesidad de modificar las VLAN existentes.

    VLAN 10: Naranja – Edificio Izquierdo.
    VLAN 20: Verde – Edificio Izquierdo.
    VLAN 30: Naranja – Edificio Derecho.
    VLAN 40: Verde – Edificio Derecho.
    VLAN 99: Administración.
    VLAN 100: Administración ServidoresDHCP.

La VLAN 99 se mantiene separada de las VLAN destinadas a los usuarios para facilitar la administración y gestión de los dispositivos de red.


### Configuración realizada

MS1

    en
    conf t
     hostname MS1
     !Creacion de vlan
     vlan 100
      name  VLAN_GesitonDHCP_202000806
      end

MS2

    en
    conf t
     hostname MS2
     !Creacion de vlans
     vlan 30
      name  VLAN_Naranja_EdificioDER_202000806
      exit
     vlan 40
      name  VLAN_Verde_EdificioDER_202000806
      end


MS6

    en 
    conf t
     hostname MS6
     !Creacion de vlan
     vlan 99
      name  VLAN_ADMIN_EdificioADM_202000806
      end

MS7

    en
    conf t
     hostname MS7
     !Creacion de vlans
     vlan 10
      name  VLAN_Naranja_EdificioIZQ_202000806
      exit
     vlan 20
      name  VLAN_Verde_EdificioIZQ_202000806
      end

### Verificación

Después de crear las VLAN, es recomendable verificar que hayan sido registradas correctamente en el switch mediante:

    show vlan brief

---

### 3.  **Configuración de interfaces**

Una vez creadas las VLAN, se configuraron las interfaces de los switches según la función que desempeñarán dentro de la topología. Los enlaces trunk permiten transportar múltiples VLAN entre dispositivos de red, mientras que los puertos access pertenecen a una única VLAN.

### Configuración realizada

MS1

    en
    conf t

    !modo access
    interface range g1/0/3-4
     switchport mode access
     switchport access vlan 100
     no shutdown
     exit
    end


MS2

    en
    conf t

    !modo trunk
     interface range g1/0/1-6
      switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
      no shutdown
      exit
     end


MS3

    en
    conf t

    !modo trunk
     interface range fa0/1-7
      switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
      no shutdown
      exit
     end


MS4

    en
    conf t

    !modo trunk
    interface range fa0/1-6
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit
    end


MS5

    en
    conf t

    !modo trunk
    interface range fa0/1-4
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit
    end


MS6

    en
    conf t

    !modo access
    interface g1/0/3
     switchport mode access
     switchport access vlan 99
     no shutdown
     exit
    end


MS7

    en
    conf t

    !modo trunk
    interface range g1/0/1-6
    switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit
    end



MS8 y MS9

    en
    conf t

    !modo trunk  giga
    interface range g0/1-2
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit

    !modo trunk fast
    interface range fa0/1-6
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit
    end


SW1 y SW2

    en
    conf t

    !modo trunk
    interface range g0/1-2
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit

    !modo access
    interface f0/1
     switchport mode access
     switchport access vlan 10
     no shutdown
     exit

    interface f0/2
     switchport mode access
     switchport access vlan 20
     no shutdown
     exit
    end


SW3 y SW4

    en
    conf t

    !modo trunk
    interface F0/3
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit

    !modo access
    interface f0/1
     switchport mode access
     switchport access vlan 40
     no shutdown
     exit

    interface f0/2
     switchport mode access
     switchport access vlan 30
     no shutdown
     exit
    end

### Verificación

Para comprobar que las interfaces fueron configuradas correctamente como enlaces trunk, se utilizó:

    show interfaces trunk

---

### 4.  **Protocolos VTP y STP**


### 4.1 Protocolo VTP

El VLAN Trunking Protocol (VTP) se utilizó para facilitar la propagación y administración de las VLAN dentro de la infraestructura de switches.

Se definió a MS1 como servidor VTP, encargado de mantener la información de las VLAN, mientras que los demás switches fueron configurados como clientes VTP.

Se utilizó VTP versión 2, con un dominio común y una contraseña para permitir que los switches pertenecieran al mismo dominio VTP.

| Parámetro   | Configuración     |
| ----------- | ----------------- |
| Versión VTP | `2`               |
| Dominio VTP | `proyecto1Redes2` |
| Contraseña  | `redes2`          |
| MS2         | Servidor VTP      |
| MS3–MS5     | Clientes VTP      |
| MS7         | Servidor VTP      |
| MS8–MS9     | Clientes VTP      |
| SW1–SW4     | Clientes VTP      |


### 4.2 Protocolo STP

El Spanning Tree Protocol (STP) se implementó para evitar bucles de capa 2 dentro de la red conmutada.

Debido a que la topología cuenta con enlaces redundantes entre switches, existe la posibilidad de que se formen caminos circulares. Para evitar estos bucles se utilizó Rapid-PVST+, que proporciona una convergencia más rápida que STP tradicional y permite manejar una instancia de spanning-tree por VLAN.

La configuración establece a MS2 y MS7 como Root Bridge principal para las VLAN utilizadas en el proyecto.

VLAN configuradas para STP

| VLAN | Root Bridge principal |
| ---: | --------------------- |
|   10 | MS7                   |
|   20 | MS7                   |
|   30 | MS2                   |
|   40 | MS2                   |


### Configuración realizada

MS2

    en
    conf t

    !Configuracion vtp
     vtp version 2
     vtp mode server
     vtp domain proyecto1Redes2
     vtp password redes2

    !configuracion stp
    !activacion de rpvst
     spanning-tree mode rapid-pvst 
    !configuracion como root-bridge para todas las vlans despues de verlas creado
     spanning-tree vlan 30 root primary
     spanning-tree vlan 40 root primary
     exit

MS7

    en
    conf t

    !Configuracion vtp
     vtp version 2
     vtp mode server
     vtp domain proyecto1Redes2
     vtp password redes2

    !configuracion stp
    !activacion de rpvst
     spanning-tree mode rapid-pvst 
    !configuracion como root-bridge para todas las vlans despues de verlas creado
     spanning-tree vlan 10 root primary
     spanning-tree vlan 20 root primary
     exit

Para los demas switchs

    en
    conf t
    !Configuracion vtp
     vtp version 2
     vtp mode client
     vtp domain proyecto1Redes2
     vtp password redes2

    !configuracion stp
    !activacion de rpvst
     spanning-tree mode rapid-pvst
     exit


### Verificación

Para verificar el estado y configuración de VTP se recomienda utilizar:

    show vtp status

Para comprobar que las VLAN están disponibles en los switches clientes.

    show vlan brief

Para comprobar el funcionamiento de Spanning Tree se utiliza:

    show spanning-tree

---

### 5. **Agregacion de Enlaces (Etherchannel)**

### 5.1 Agregación de Enlaces — LACP (Edificio Izquierdo)


LACP (Link Aggregation Control Protocol - IEEE 802.3ad), modo `active` en ambos extremos de cada enlace.

### Enlaces configurados (5 en total)

| Enlace | Switch A | Puertos A | Switch B | Puertos B |
|---|---|---|---|---|
| 1 | MS7 | Gi1/0/1-3 | MS9 | Fa0/1-3 |
| 2 | MS7 | Gi1/0/4-6 | MS8 | Fa0/1-3 |
| 3 | MS8 | Fa0/4-6 | MS9 | Fa0/4-6 |
| 4 | MS8 | Gi0/1-2 | SW2 | Gi0/1-2 |
| 5 | MS9 | Gi0/1-2 | SW1 | Gi0/1-2 |

### Configuración realizada
MS7

    en
    conf t
    !creación del porthchannel
    !Grupo 1
     !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range GigabitEthernet1/0/1-3
      channel-protocol lacp
      channel-group 1 mode active


    !Grupo 2
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 2
     !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range GigabitEthernet1/0/4-6
      channel-protocol lacp
      channel-group 2 mode active
      end

MS9

    en
    conf t
    !creación del porthchannel
    !Grupo 1
     !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/1-3
      channel-protocol lacp
      channel-group 1 mode active
      exit

    !Grupo 2
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 2
     !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/4-6
      channel-protocol lacp
      channel-group 2 mode active
      exit

    !Grupo 3
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 3
     !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range GigabitEthernet0/1-2
      channel-protocol lacp
      channel-group 3 mode active
      end

MS8

    en
    conf t
    !creación del porthchannel
    !Grupo 1
     !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/1-3
      channel-protocol lacp
      channel-group 1 mode active
      exit

    !Grupo 2
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 2
     !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/4-6
      channel-protocol lacp
      channel-group 2 mode active
      exit

    !Grupo 3
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 3
     !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range GigabitEthernet0/1-2
      channel-protocol lacp
      channel-group 3 mode active
      end

SW1 y SW2

    en
    conf t
    !creación del porthchannel
    !Grupo 1
     !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !  switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range GigabitEthernet0/1-2
      channel-protocol lacp
      channel-group 1 mode active
      exit
      end


### 5.2 Agregación de Enlaces — PAgP (Edificio Derecho)

Protocolo PAgP (Port Aggregation Protocol - propietario de Cisco), modo `desirable` en ambos extremos de cada enlace.

### Enlaces configurados (3 en total)

| Enlace | Switch A | Puertos A | Switch B | Puertos B |
|---|---|---|---|---|
| 1 | MS2 | Gi1/0/1-3 | MS3 | Fa0/1-3 |
| 2 | MS3 | Fa0/4-7 | MS4 | Fa0/1-4 |
| 3 | MS4 | Fa0/5-6 | MS5 | Fa0/1-2 |

### Configuración realizada

MS2

    en
    conf t
    !creación del porthchannel
    !Grupo 1
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range GigabitEthernet1/0/1-3
      channel-protocol pagp
      channel-group 1 mode desirable
      end

MS3

    en
    conf t
    !creación del porthchannel
    !Grupo 1
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/1-3
      channel-protocol pagp
      channel-group 1 mode desirable
      exit

    !Grupo 2
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 2
      !switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/4-7
      channel-protocol pagp
      channel-group 2 mode desirable
      end

MS4

    en
    conf t
    !creación del porthchannel
    !Grupo 1
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/1-4
      channel-protocol pagp
      channel-group 1 mode desirable
      exit

    !Grupo 2
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 2
      !switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/5-6
      channel-protocol pagp
      channel-group 2 mode desirable
      end

MS5

    en
    conf t
    !creación del porthchannel
    !Grupo 1
    !Creacion del portchanel y configuracion del mismo
     interface port-channel 1
      !switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
    !asignamos las interfaces al portchannel creado
     interface range fastEthernet0/1-2
      channel-protocol pagp
      channel-group 1 mode desirable
      end

### Verificación

    show etherchannel summary
    show interfaces port-channel 1
    show spanning-tree active

### Prueba de tolerancia a fallos

Se realizó `shutdown` en un puerto físico


---

### 6. **Direccionamiento IP — Subnetting (VLSM y FLSM)**

### Redes base asignadas
`192.188.X.0/24` (donde X corresponde a los últimos dos dígitos de su carné) para 
VLANs de usuarios. 

`10.4.X.0/24` (donde X corresponde a los últimos dos dígitos de su carné) para enlaces 
de enrutamiento.

- `192.188.6.0/24` → VLANs de usuario (VLSM)
- `10.4.6.0/24` → Enlaces de enrutamiento punto a punto (FLSM)

Para el cálculo de VLSM se consideró la cantidad de hosts actuales en cada VLAN 
más un margen de crecimiento, evitando tanto el desperdicio excesivo de 
direcciones como un dimensionamiento demasiado ajustado que impida escalar 
la red sin rediseñar el esquema. Las subredes se ordenaron de mayor a menor 
tamaño, dejando el resto del bloque `192.188.6.0/24` sin asignar como reserva 
para la creación de nuevas VLANs en el futuro.

Para los enlaces de enrutamiento se utilizó FLSM con bloques fijos `/30`, 
ya que todo enlace punto a punto requiere únicamente 2 direcciones IP 
usables, independientemente de su ubicación en la topología.

### 6.1 VLSM — Subredes de VLANs (192.188.6.0/24)

| VLAN | Nombre | Red | Máscara | Rango usable | Broadcast | Gateway (SVI) | Switch |
|---|---|---|---|---|---|---|---|
| 10 | Naranja IZQ | 192.188.6.0 | /27 (255.255.255.224) | .1 – .30 | .31 | 192.188.6.1 | MS7 |
| 20 | Verde IZQ | 192.188.6.32 | /27 (255.255.255.224) | .33 – .62 | .63 | 192.188.6.33 | MS7 |
| 30 | Naranja DER | 192.188.6.64 | /28 (255.255.255.240) | .65 – .78 | .79 | 192.188.6.65 | MS2 |
| 40 | Verde DER | 192.188.6.80 | /28 (255.255.255.240) | .81 – .94 | .95 | 192.188.6.81 | MS2 |
| 99 | ADMIN | 192.188.6.96 | /29 (255.255.255.248) | .97 – .102 | .103 | 192.188.6.97 | MS6 |
| 100 | Gestión DHCP | 192.188.6.104 | /29 (255.255.255.248) | .105 – .110 | .111 | 192.188.6.105 | MS1 |

<!-- **Asignación de hosts fijos — VLAN 99 (ADMIN, switch MS6):**

| Dispositivo | IP | Máscara | Gateway | Tipo |
|---|---|---|---|---|
| SVI VLAN 99 (gateway) | 192.188.6.97 | /29 | — | — |
| PC Administración | 192.188.6.98 | /29 | 192.188.6.97 | Dinámica (DHCP) | -->

**Asignación de hosts fijos — VLAN 100 (Gestión DHCP, switch MS1):**

| Dispositivo | IP | Máscara | Gateway | Tipo |
|---|---|---|---|---|
| SVI VLAN 100 (gateway) | 192.188.6.105 | /29 | — | — |
| Server DHCP1 (Izquierdo) | 192.188.6.106 | /29 | 192.188.6.105 | Estática |
| Server DHCP2 (Derecho) | 192.188.6.107 | /29 | 192.188.6.105 | Estática |

**Espacio reservado para crecimiento futuro:**
El rango `192.188.6.112` en adelante (hasta `192.188.6.255`) queda sin 
asignar, disponible para la creación de nuevas VLANs o la expansión de 
las subredes existentes sin necesidad de rediseñar el esquema de 
direccionamiento actual.

### 6.2 FLSM — Enlaces de enrutamiento (10.4.6.0/24)


Todos los enlaces punto a punto que interconectan los switches multicapa 
de la Red MAN utilizan bloques fijos /30 (255.255.255.252), ya que cada 
enlace requiere únicamente 2 direcciones IP usables sin importar su 
ubicación en la topología.

| Enlace | Switch A | Switch B | Subred | IP Switch A | IP Switch B |
|---|---|---|---|---|---|
| 1 | MS1 | MS7 | 10.4.6.0/30 | 10.4.6.1 | 10.4.6.2 |
| 2 | MS1 | MS2 | 10.4.6.4/30 | 10.4.6.5 | 10.4.6.6 |
| 3 | MS7 | MS6 | 10.4.6.8/30 | 10.4.6.9 | 10.4.6.10 |
| 4 | MS7 | MS2 | 10.4.6.12/30 | 10.4.6.13 | 10.4.6.14 |
| 5 | MS2 | MS6 | 10.4.6.16/30 | 10.4.6.17 | 10.4.6.18 |

El resto del bloque 10.4.6.0/24 (desde 10.4.6.20 en adelante) queda 
disponible como reserva para enlaces de enrutamiento adicionales que 
se requieran en el futuro.

---


### 6. **Enrutamiento Dinámico — OSPF**

Según el carné del estudiante (terminación par: 06), corresponde utilizar 
**OSPF (Open Shortest Path First)** como protocolo de enrutamiento dinámico, 
tanto para la interconexión entre los 4 edificios (Red MAN) como para el 
enrutamiento inter-VLAN dentro de cada edificio.

OSPF permite que los switches multicapa aprendan automáticamente las rutas 
hacia las redes remotas a través del intercambio de información entre 
vecinos, sin necesidad de configurar rutas estáticas manualmente por cada 
subred.

Se utilizó un único proceso OSPF (`router ospf 1`) y una sola área (`area 0`, 
backbone) dado el tamaño reducido de la red.


### Dispositivos con OSPF configurado

| Switch | Rol | Redes que anuncia |
|---|---|---|
| MS1 | Tránsito Red MAN + Gestión DHCP | 10.4.6.0/30, 10.4.6.4/30, 192.188.6.104/29 |
| MS7 | Core Edificio Izquierdo | 10.4.6.0/30, 10.4.6.8/30, 10.4.6.12/30, 192.188.6.0/27, 192.188.6.32/27 |
| MS2 | Distribución Edificio Derecho | 10.4.6.4/30, 10.4.6.12/30, 10.4.6.16/30, 192.188.6.64/28, 192.188.6.80/28 |
| MS6 | Edificio Administración | 10.4.6.8/30, 10.4.6.16/30, 192.188.6.96/29 |

### Comandos de configuración

**MS1**

    en
    conf t
    !configurando puertos y asignando ips
     interface gigabitEthernet 1/0/1
      no switchport
      ip address 10.4.6.1 255.255.255.252
      no shutdown
      exit
     interface gigabitEthernet 1/0/2
      no switchport
      ip address 10.4.6.5 255.255.255.252
      no shutdown

    !activamos la funcion de enrutamiento
     ip routing

    !configuracion inter-vlan
     interface vlan 100
      ip address 192.188.6.105 255.255.255.248
      no shutdown  
      exit

    !enrutamiento dinamico OSPF
     router ospf 1
      !router-id 1.1.1.1
      network 10.4.6.0 0.0.0.3 area 0
      network 10.4.6.4 0.0.0.3 area 0
      network 192.188.6.104 0.0.0.7 area 0
      end


**MS2**

    en
    conf t
    !configurando puertos y asignando ips
     interface gigabitEthernet 1/0/5
      no switchport
      ip address 10.4.6.6 255.255.255.252
      no shutdown
      exit
     interface gigabitEthernet 1/0/6
      no switchport
      ip address 10.4.6.14 255.255.255.252
      no shutdown
      exit
     interface gigabitEthernet 1/0/4
      no switchport
      ip address 10.4.6.17 255.255.255.252
      no shutdown
      exit

    !activamos la funcion de enrutamiento
     ip routing

    !configuracion inter-vlan
     interface vlan 30
      ip address 192.188.6.65 255.255.255.240
      no shutdown
      exit
     interface vlan 40
      ip address 192.188.6.81 255.255.255.240
      no shutdown
      exit

    !enrutamiento dinamico OSPF
     router ospf 1
      !router-id 2.2.2.2
      network 10.4.6.4 0.0.0.3 area 0
      network 10.4.6.12 0.0.0.3 area 0
      network 10.4.6.16 0.0.0.3 area 0
      network 192.188.6.64 0.0.0.15 area 0
      network 192.188.6.80 0.0.0.15 area 0
      end


**MS6**

    en
    conf t
    !configurando puertos y asignando ips
     interface gigabitEthernet 1/0/1
      no switchport
      ip address 10.4.6.10 255.255.255.252
      no shutdown
      exit
     interface gigabitEthernet 1/0/2
      no switchport
      ip address 10.4.6.18 255.255.255.252
      no shutdown
      exit

    !activamos la funcion de enrutamiento
     ip routing

    !configuracion inter-vlan
     interface vlan 99
      ip address 192.188.6.97 255.255.255.248
      no shutdown
      exit

    !enrutamiento dinamico OSPF
     router ospf 1
      !router-id 6.6.6.6
      network 10.4.6.8 0.0.0.3 area 0
      network 10.4.6.16 0.0.0.3 area 0
      network 192.188.6.96 0.0.0.7 area 0
      end


**MS7**

    en
    conf t
    !configurando puertos y asignando ips
     interface gigabitEthernet 1/0/8
      no switchport
      ip address 10.4.6.2 255.255.255.252
      no shutdown
      exit
     interface gigabitEthernet 1/0/7
      no switchport
      ip address 10.4.6.9 255.255.255.252
      no shutdown
      exit
     interface gigabitEthernet 1/0/9
      no switchport
      ip address 10.4.6.13 255.255.255.252
      no shutdown
      exit

    !activamos la funcion de enrutamiento
     ip routing

    !configuracion inter-vlan
     interface vlan 10
      ip address 192.188.6.1 255.255.255.224
      no shutdown
      exit
     interface vlan 20
      ip address 192.188.6.33 255.255.255.224
      no shutdown
      exit

    !enrutamiento dinamico OSPF
     router ospf 1
      !router-id 7.7.7.7
      network 10.4.6.0 0.0.0.3 area 0
      network 10.4.6.8 0.0.0.3 area 0
      network 10.4.6.12 0.0.0.3 area 0
      network 192.188.6.0 0.0.0.31 area 0
      network 192.188.6.32 0.0.0.31 area 0
      end

### Verificación

Para ver la rutas en el protocolo de enrutamiento

    show ip route ospf

Para ver las interfaces con sus ips

    show ip interface brief

Para ver las ips

    show ip route

---


### 6. **Servicios de Red — DHCP**

### Distribución de responsabilidad por servidor

Según lo requerido en el enunciado, cada servidor DHCP atiende las VLANs 
de su edificio correspondiente.

| Servidor | IP (estática) | Edificio | VLANs que atiende |
|---|---|---|---|
| DHCP1 | 192.188.6.106 | Izquierdo | VLAN 10, VLAN 20 |
| DHCP2 | 192.188.6.107 | Derecho | VLAN 30, VLAN 40 |

Ambos servidores utilizan direccionamiento **estático**, ya que un 
servidor DHCP no puede obtener su propia IP de forma dinámica sin 
generar una dependencia circular. Esta es la única excepción a la 
regla de "sin IP estática" del proyecto, aplicable únicamente a 
infraestructura de servicio, no a dispositivos finales de usuario.

### Pools configurados — Servidor DHCP1 (Izquierdo)

| Pool | VLAN | Gateway | Rango de asignación | Máscara |
|---|---|---|---|---|
| Pool_Naranja_IZQ | 10 | 192.188.6.1 | 192.188.6.2 – 192.188.6.30 | 255.255.255.224 |
| Pool_Verde_IZQ | 20 | 192.188.6.33 | 192.188.6.34 – 192.188.6.62 | 255.255.255.224 |


### Pools configurados — Servidor DHCP2 (Derecho)

| Pool | VLAN | Gateway | Rango de asignación | Máscara |
|---|---|---|---|---|
| Pool_Naranja_DER | 30 | 192.188.6.65 | 192.188.6.66 – 192.188.6.78 | 255.255.255.240 |
| Pool_Verde_DER | 40 | 192.188.6.81 | 192.188.6.82 – 192.188.6.94 | 255.255.255.240 |

### DHCP Relay — ip helper-address

Dado que los servidores DHCP no residen dentro de las VLANs de usuario, 
se configuró DHCP Relay en las SVIs correspondientes para reenviar las 
solicitudes broadcast como tráfico unicast hacia el servidor adecuado.

### Comandos de configuración

**MS7 — VLAN 10 y 20 (apuntan a DHCP1)**

    enable
    configure terminal
     interface vlan 10
      ip helper-address 192.188.6.106
      exit
     interface vlan 20
      ip helper-address 192.188.6.106
      exit

**MS2 — VLAN 30 y 40 (apuntan a DHCP2)**

    enable
    configure terminal
     interface vlan 30
      ip helper-address 192.188.6.107
      exit
     interface vlan 40
      ip helper-address 192.188.6.107
      exit

### Configuración en dispositivos finales

Todos los PCs y laptops de las 5 VLANs (10, 20, 30, 40) se 
configuraron en modo **DHCP** (Desktop → IP Configuration → DHCP), 
sin excepción, cumpliendo con el requisito del proyecto de no utilizar 
direccionamiento estático en dispositivos finales.

### Verificación

    show ip dhcp binding      (en el router/switch, si aplica)
    ipconfig                  (en cada PC, desde Command Prompt)
    show ip interface brief   (confirmar helper-address aplicado)

### Resultado esperado
Cada PC/laptop obtiene automáticamente su dirección IP, máscara de 
subred y gateway correspondiente a su VLAN, sin necesidad de 
configuración manual, confirmando el correcto funcionamiento del 
servicio DHCP a través de las subredes mediante DHCP Relay.



---

### 6. **Seguridad y Control de Acceso — ACLs**

| VLAN | Política |
|---|---|
| Naranja (10, 30) | Comunicación permitida solo entre Naranja de ambos edificios. Bloqueada hacia Verde y ADMIN. |
| Verde (20, 40) | Comunicación permitida solo entre Verde de ambos edificios. Bloqueada hacia Naranja y ADMIN. |
| ADMIN (99) | Puede iniciar comunicación (ping) hacia todas las VLANs. Ninguna VLAN puede iniciar comunicación hacia ADMIN, pero sí se permite la respuesta (echo-reply) a sesiones iniciadas por ADMIN. |

Se utilizaron ACLs extendidas nombradas, aplicadas en dirección `in` 
sobre la SVI de origen del tráfico, en los switches donde ocurre el 
enrutamiento inter-VLAN (MS7, MS2).

### Puntos de aplicación

| Switch | VLAN | ACL aplicada | Dirección |
|---|---|---|---|
| MS7 | 10 (Naranja IZQ) | ACL_NARANJA_IZQ | in |
| MS7 | 20 (Verde IZQ) | ACL_VERDE_IZQ | in |
| MS2 | 30 (Naranja DER) | ACL_NARANJA_DER | in |
| MS2 | 40 (Verde DER) | ACL_VERDE_DER | in |


**Propósito de cada regla — ACL_NARANJA_IZQ (aplicada en VLAN 10, MS7):**

| Línea | Propósito |
|---|---|
| permit ip 192.188.6.0/27 → 192.188.6.64/28 | Permite comunicación con VLAN Naranja DER (misma VLAN, otro edificio) |
| permit icmp echo-reply → 192.188.6.96/29 | Permite respuesta a ping iniciado por VLAN ADMIN |
| deny ip → 192.188.6.32/27 | Bloquea comunicación hacia VLAN Verde IZQ |
| deny ip → 192.188.6.80/28 | Bloquea comunicación hacia VLAN Verde DER |
| deny ip → 192.188.6.96/29 | Bloquea inicio de comunicación hacia VLAN ADMIN |
| deny ip → 192.188.6.104/29 | Bloquea comunicación hacia subred de gestión DHCP |
| permit ip any any | Permite el resto del tráfico no contemplado explícitamente |

**Propósito de cada regla — ACL_VERDE_IZQ (aplicada en VLAN 20, MS7):**

| Línea | Propósito |
|---|---|
| permit ip 192.188.6.32/27 → 192.188.6.80/28 | Permite comunicación con VLAN Verde DER (misma VLAN, otro edificio) |
| permit icmp echo-reply → 192.188.6.96/29 | Permite respuesta a ping iniciado por VLAN ADMIN |
| deny ip → 192.188.6.0/27 | Bloquea comunicación hacia VLAN Naranja IZQ |
| deny ip → 192.188.6.64/28 | Bloquea comunicación hacia VLAN Naranja DER |
| deny ip → 192.188.6.96/29 | Bloquea inicio de comunicación hacia VLAN ADMIN |
| deny ip → 192.188.6.104/29 | Bloquea comunicación hacia subred de gestión DHCP |
| permit ip any any | Permite el resto del tráfico no contemplado explícitamente |

**Propósito de cada regla — ACL_NARANJA_DER (aplicada en VLAN 30, MS2):**

| Línea | Propósito |
|---|---|
| permit ip 192.188.6.64/28 → 192.188.6.0/27 | Permite comunicación con VLAN Naranja IZQ (misma VLAN, otro edificio) |
| permit icmp echo-reply → 192.188.6.96/29 | Permite respuesta a ping iniciado por VLAN ADMIN |
| deny ip → 192.188.6.32/27 | Bloquea comunicación hacia VLAN Verde IZQ |
| deny ip → 192.188.6.80/28 | Bloquea comunicación hacia VLAN Verde DER |
| deny ip → 192.188.6.96/29 | Bloquea inicio de comunicación hacia VLAN ADMIN |
| deny ip → 192.188.6.104/29 | Bloquea comunicación hacia subred de gestión DHCP |
| permit ip any any | Permite el resto del tráfico no contemplado explícitamente |

**Propósito de cada regla — ACL_VERDE_DER (aplicada en VLAN 40, MS2):**

| Línea | Propósito |
|---|---|
| permit ip 192.188.6.80/28 → 192.188.6.32/27 | Permite comunicación con VLAN Verde IZQ (misma VLAN, otro edificio) |
| permit icmp echo-reply → 192.188.6.96/29 | Permite respuesta a ping iniciado por VLAN ADMIN |
| deny ip → 192.188.6.0/27 | Bloquea comunicación hacia VLAN Naranja IZQ |
| deny ip → 192.188.6.64/28 | Bloquea comunicación hacia VLAN Naranja DER |
| deny ip → 192.188.6.96/29 | Bloquea inicio de comunicación hacia VLAN ADMIN |
| deny ip → 192.188.6.104/29 | Bloquea comunicación hacia subred de gestión DHCP |
| permit ip any any | Permite el resto del tráfico no contemplado explícitamente |


### Configuración — MS7

    en
    conf t
    !Creamos la lista de acceso
     ip access-list extended ACL_NARANJA_IZQ
      permit ip 192.188.6.0 0.0.0.31 192.188.6.64 0.0.0.15
      permit icmp 192.188.6.0 0.0.0.31 192.188.6.96 0.0.0.7 echo-reply
      deny ip 192.188.6.0 0.0.0.31 192.188.6.32 0.0.0.31
      deny ip 192.188.6.0 0.0.0.31 192.188.6.80 0.0.0.15
      deny ip 192.188.6.0 0.0.0.31 192.188.6.96 0.0.0.7
      deny ip 192.188.6.0 0.0.0.31 192.188.6.104 0.0.0.7
      permit ip any any
     exit
    !Asignamos la lista de acceso
     interface vlan 10
      ip access-group ACL_NARANJA_IZQ in
     exit

    !Creamos la lista de acceso
     ip access-list extended ACL_VERDE_IZQ
      permit ip 192.188.6.32 0.0.0.31 192.188.6.80 0.0.0.15
      permit icmp 192.188.6.32 0.0.0.31 192.188.6.96 0.0.0.7 echo-reply
      deny ip 192.188.6.32 0.0.0.31 192.188.6.0 0.0.0.31
      deny ip 192.188.6.32 0.0.0.31 192.188.6.64 0.0.0.15
      deny ip 192.188.6.32 0.0.0.31 192.188.6.96 0.0.0.7
      deny ip 192.188.6.32 0.0.0.31 192.188.6.104 0.0.0.7
      permit ip any any
     exit
    !Asignamos la lista de acceso
     interface vlan 20
      ip access-group ACL_VERDE_IZQ in
     exit
     end

### Configuración — MS2

    en
    conf t
    !Creamos la lista de acceso
     ip access-list extended ACL_NARANJA_DER
      permit ip 192.188.6.64 0.0.0.15 192.188.6.0 0.0.0.31
      permit icmp 192.188.6.64 0.0.0.15 192.188.6.96 0.0.0.7 echo-reply
      deny ip 192.188.6.64 0.0.0.15 192.188.6.32 0.0.0.31
      deny ip 192.188.6.64 0.0.0.15 192.188.6.80 0.0.0.15
      deny ip 192.188.6.64 0.0.0.15 192.188.6.96 0.0.0.7
      deny ip 192.188.6.64 0.0.0.15 192.188.6.104 0.0.0.7
      permit ip any any
     exit
    !Asignamos la lista de acceso
     interface vlan 30
      ip access-group ACL_NARANJA_DER in
     exit

    !Creamos la lista de acceso
     ip access-list extended ACL_VERDE_DER
      permit ip 192.188.6.80 0.0.0.15 192.188.6.32 0.0.0.31
      permit icmp 192.188.6.80 0.0.0.15 192.188.6.96 0.0.0.7 echo-reply
      deny ip 192.188.6.80 0.0.0.15 192.188.6.0 0.0.0.31
      deny ip 192.188.6.80 0.0.0.15 192.188.6.64 0.0.0.15
      deny ip 192.188.6.80 0.0.0.15 192.188.6.96 0.0.0.7
      deny ip 192.188.6.80 0.0.0.15 192.188.6.104 0.0.0.7
      permit ip any any
     exit
    !Asignamos la lista de acceso
     interface vlan 40
      ip access-group ACL_VERDE_DER in
     exit
     end



### Comandos de verificación

Para ver la lista de listas de acceso 

    show access-lists

Para ver la listas de acceso asignadas a dicha vlan

    show ip interface vlan [número]

Para ver la lista de lo que capturo la lista de accesso

    show access-lists nombreListaAcceso

### Pruebas de conectividad realizadas

| Origen | Destino | Resultado esperado | Resultado obtenido |
|---|---|---|---|
| Naranja IZQ | Naranja DER | Permitido | Successful|
| Verde IZQ | Verde DER | Permitido | Successful|
| Naranja IZQ | Verde IZQ/DER | Bloqueado | Failed|
| Verde IZQ | Naranja IZQ/DER | Bloqueado | Failed|
| ADMIN | Naranja/Verde (cualquiera) | Permitido | Successful|
| Naranja/Verde (cualquiera) | ADMIN | Bloqueado | Failed|

---