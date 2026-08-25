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


### Criterio de numeración
La numeración fue definida de manera que exista una separación entre los identificadores, permitiendo agregar nuevas VLAN posteriormente sin necesidad de modificar las VLAN existentes.

    VLAN 10: Naranja – Edificio Izquierdo.
    VLAN 20: Verde – Edificio Izquierdo.
    VLAN 30: Naranja – Edificio Derecho.
    VLAN 40: Verde – Edificio Derecho.
    VLAN 99: Administración.

La VLAN 99 se mantiene separada de las VLAN destinadas a los usuarios para facilitar la administración y gestión de los dispositivos de red.


### Configuración realizada en MS1
La creación de las VLAN se realizó en el switch multicapa MS1 mediante el siguiente procedimiento:

    enable
    configure terminal

    hostname MS1

    ! Creación de VLANs
    vlan 10
    name VLAN_Naranja_EdificioIZQ_202000806
    exit

    vlan 20
    name VLAN_Verde_EdificioIZQ_202000806
    exit

    vlan 30
    name VLAN_Naranja_EdificioDER_202000806
    exit

    vlan 40
    name VLAN_Verde_EdificioDER_202000806
    exit

    vlan 99
    name VLAN_ADMIN_EdificioADM_202000806
    exit

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

    !modo trunk
     interface range g1/0/1-2
      switchport trunk encapsulation dot1q
      switchport mode trunk
      switchport trunk allowed vlan 10,20,30,40,99
      no shutdown
      exit

    !modo access
    interface range g1/0/3-4
     switchport mode access
     switchport access vlan 99
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

    !modo trunk
    interface range g1/0/1-2
     switchport trunk encapsulation dot1q
     switchport mode trunk
     switchport trunk allowed vlan 10,20,30,40,99
     no shutdown
     exit

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
    interface range g1/0/1-9
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
| MS1         | Servidor VTP      |
| MS2–MS9     | Clientes VTP      |
| SW1–SW4     | Clientes VTP      |


### 4.2 Protocolo STP

El Spanning Tree Protocol (STP) se implementó para evitar bucles de capa 2 dentro de la red conmutada.

Debido a que la topología cuenta con enlaces redundantes entre switches, existe la posibilidad de que se formen caminos circulares. Para evitar estos bucles se utilizó Rapid-PVST+, que proporciona una convergencia más rápida que STP tradicional y permite manejar una instancia de spanning-tree por VLAN.

La configuración establece a MS1 como Root Bridge principal para las VLAN utilizadas en el proyecto.

VLAN configuradas para STP

| VLAN | Root Bridge principal |
| ---: | --------------------- |
|   10 | MS1                   |
|   20 | MS1                   |
|   30 | MS1                   |
|   40 | MS1                   |
|   99 | MS1                   |


### Configuración realizada

MS1

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
     spanning-tree vlan 30 root primary
     spanning-tree vlan 40 root primary
     spanning-tree vlan 99 root primary
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