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



---