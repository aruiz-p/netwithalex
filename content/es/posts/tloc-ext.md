---
author: Alex
draft: false
category:
  - sdwan
date: "2025-08-15T00:00:00+00:00"
title: "TLOC Extension en Cisco SD-WAN: Diseño y Configuración"
summary: Aprende cómo funciona TLOC Extension en Cisco SD-WAN, sus casos de uso, pasos de configuración y mejores prácticas para mejorar la resiliencia y optimizar los recursos de la WAN.
description: Aprende cómo funciona TLOC Extension en Cisco SD-WAN, sus casos de uso, pasos de configuración y mejores prácticas para mejorar la resiliencia y optimizar los recursos de la WAN.
url: /tloc-ext-es/
tag:
  - TLOC
  - design
---
## Introducción

En una red Cisco SD-WAN, la resiliencia del transporte y el uso eficiente de los mismos son fundamentales para ofrecer una conectividad confiable. Cada WAN Edge en la red se conecta a una red de transporte, como MPLS, Internet, 4G/5G, etc. La combinación lógica única de _System-ip + Color + Encapsulation_ da lugar a lo que se conoce como **Transport Locators (TLOCs)**.

Sin embargo, en escenarios con dos routers, no todas las sucursales tienen el privilegio de contar con múltiples circuitos WAN conectados a cada router. En algunos casos, un circuito WAN puede terminar en un router, mientras que otro circuito solo está disponible en un segundo router. Aquí es donde entra TLOC Extension, permitiendo que un router aproveche la conexión de transporte de otro router en el mismo sitio, aumentando la redundancia de túneles y la flexibilidad.  

Exploremos esto en más detalle.

## ¿Qué es TLOC Extension?

TLOC Extension en Cisco SD-WAN permite que un router utilice la interfaz de transporte de otro router como si fuera propia. Esto se logra extendiendo el TLOC (Transport Locator) de un router a otro a través de una conexión de capa 2 o capa 3. Su representación física se vería algo así:

{{< figure src="/wp-content/uploads/2025/08/tloc-ex-1.png" title="Figura 1" caption="TLOC Extension de capa 2" >}}

En la práctica, esto significa que el Router BR1-1 puede aprovechar los circuitos de transporte del Router BR1-2s (por ejemplo, MPLS o Internet) sin estar directamente conectado a ellos, y viceversa. Este mecanismo asegura que todos los routers del sitio puedan usar plenamente todos los transportes disponibles en la red SD-WAN, incluso si no cuentan con conexiones de transporte dedicadas. La representación lógica se vería así:

{{< figure src="/wp-content/uploads/2025/08/tloc-ex-2.png" title="Figura 2" caption="TLOC extension con dos transportes" >}}

Ahora, _BR1-1_ puede usar **INET** como transporte y _BR1-2_ puede usar **MPLS** como transporte disponible.  

Algunas ventajas de usar TLOC Extension:

- Reduce el número de circuitos directos necesarios por router.
- Aporta resiliencia a nivel de sitio.
- Simplifica las implementaciones en sucursales donde solo hay un circuito de transporte disponible físicamente.

## Casos de Uso

TLOC Extension resuelve desafíos reales en implementaciones SD-WAN. Algunos casos comunes incluyen:

### Sucursales pequeñas con conectividad limitada
Una sucursal puede tener solo un enlace de Internet o MPLS. Con TLOC Extension, dos routers aún pueden operar con redundancia compartiendo ese único transporte.

{{< figure src="/wp-content/uploads/2025/08/tloc-ex-3.png" title="Figura 3" caption="TLOC Extension transporte único" >}}

### Optimización de costos
En lugar de adquirir múltiples circuitos para cada router, los costos se reducen al tener un circuito por tipo de transporte y compartirlo entre routers.

{{< figure src="/wp-content/uploads/2025/08/tloc-ex-4.png" title="Figura 4" caption="Comparación costo TLOC Extension vs Full Mesh" >}}

### Redundancia de transporte
Cuando un sitio tiene diferentes transportes distribuidos en distintos routers (Ejemplo: Internet en Router BR1-1, MPLS en Router BR1-2), TLOC Extension asegura que ambos routers puedan acceder a ambos tipos de transporte.  

{{< figure src="/wp-content/uploads/2025/08/tloc-ex-2.png" title="Figura 5" caption="TLOC extension con dos transportes" >}}

## Arquitectura y Funcionamiento

En el núcleo de SD-WAN, un TLOC se define por tres elementos:

- System IP del router
- Color (tipo de transporte al que pertenece el enlace. Ejempli: public-internet, mpls, lte)
- Encapsulation (IPsec/GRE)

Con TLOC Extension, un router establece un túnel interno a través de la LAN hacia otro router, lo que le permite anunciar y usar ese TLOC.

Puntos clave de la arquitectura:

- Una extensión TLOC se construye sobre una interfaz LAN física entre dos routers; comúnmente se usan _sub-interfaces_ y encapsulación _dot1Q_. TLOC extension de capa 3 está disponible.
- El router “donante” proporciona acceso a su interfaz de transporte.  
- El router “receptor” puede entonces formar conexiones de plano de control y de datos usando ese transporte.

Desde la perspectiva de la red, ambos routers aparecen como si estuvieran **independientemente** conectados a cada transporte.

## Configuración de TLOC Extension

Aprovecharé _Workflows_ de la UX 2.0 para generar rápidamente una configuración para un sitio con dos routers y TLOC Extension, y luego la revisaremos.  

{{< figure src="/wp-content/uploads/2025/08/tloc-ex-5.png" title="Figura 6" caption="UX2.0 Workflow configuración TLOC Extension" >}}

La configuración resultante tiene las siguientes interfaces en el _Feature Profile de Transporte_.  

{{< figure src="/wp-content/uploads/2025/08/tloc-ex-6.png" title="Figura 7" caption="Interfaces Configurdas a través del Workflow" >}}

Se crearon seis interfaces diferentes, revisemos las tres primeras para entender cómo se configura TLOC Extension.

**Nota** Fue necesario eliminar configuración de NAT no requerido en algunas de esas interfaces, aparte de esto, la configuración estaba lista.

### Interfaz #1 - Internet TLOC en BR1-1

La primera interfaz se conecta directamente al transporte de Internet, configuración estándar de interfaz y TLOC. 

```
interface GigabitEthernet1
description WAN Interface - biz-internet
ip address 31.11.0.2 255.255.255.252
ip nat outside

sdwan
interface GigabitEthernet1
tunnel-interface
encapsulation ipsec weight 1
color biz-internet
```

### Interfaz #2 - BR1-1 Interfaz "donada" a BR1-2

La segunda interfaz es necesaria para extender el TLOC de Internet hacia BR1-2, requiere una dirección IP y BR1-2 la usará como gateway por defecto.  

```
interface GigabitEthernet4
description WAN Interface - biz-internet
ip address 172.17.1.6 255.255.255.252

sdwan
interface GigabitEthernet4
tloc-extension GigabitEthernet1
exit
```

### Interfaz #6 - INET TLOC en BR1-2

Esta es una configuración normal de TLOC que usará la dirección IP de _BR1-2_ como gateway por defecto para que funcione el transporte MPLS.  

```
interface GigabitEthernet5
description WAN VPN 0 Interface - MPLS
ip address 172.17.1.5 255.255.255.252
!
ip route 0.0.0.0 0.0.0.0 172.17.1.6

sdwan
interface GigabitEthernet5
tunnel-interface
encapsulation ipsec weight 1
color mpls
```

La subred 172.17.1.4/30 necesita ser anunciada en el underlay de MPLS. Normalmente se usa BGP para este anuncio.  

La configuración de las interfaces 3, 4 y 5 es un espejo de lo que acabamos de ver.  

## Validación

Una vez que esta configuración está en su lugar, podemos validar nuestro plano de control y de datos:  

En BR1-1, nota que hay dos interfaces de SD-WAN, Gig1 and Gig5

```
Lisbon_10-1#show sdwan control local-properties 
personality                       vedge
sp-organization-name              MYSDWAN-LAB123
organization-name                 MYSDWAN-LAB123
root-ca-chain-status              Installed
root-ca-crl-status                Not-Installed
...
                         PUBLIC          PUBLIC PRIVATE         PRIVATE         PRIVATE                         MAX   RESTRICT/           LAST         SPI TIME    NAT  VM          BIND
INTERFACE                IPv4            PORT   IPv4            IPv6            PORT    VS/VM COLOR       STATE CNTRL CONTROL/     LR/LB  CONNECTION   REMAINING   TYPE CON REG     INTERFACE
                                                                                                                      STUN                                              PRF IDs
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
GigabitEthernet1       31.11.0.2       12386  31.11.0.2       ::                12386    1/1  biz-internet     up     2      no/yes/no   No/No  0:00:00:10   0:03:06:00  N    5  Default N/A                           
GigabitEthernet5       172.17.1.5      12426  172.17.1.5      ::                12426    1/0  mpls             up     2      no/yes/no   No/No  0:00:00:15   0:04:19:53  N    5  Default N/A                           
          
```

Las conexiones de control funcionan en ambos transportes como se esperaba

```
Lisbon_10-1#show sdwan control connections
                                                                                       PEER                                          PEER                                          CONTROLLER 
PEER    PEER PEER            SITE       DOMAIN PEER                                    PRIV  PEER                                    PUB                                           GROUP      
TYPE    PROT SYSTEM IP       ID         ID     PRIVATE IP                              PORT  PUBLIC IP                               PORT  ORGANIZATION            LOCAL COLOR     PROXY STATE UPTIME      ID         
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
vsmart  dtls 1.0.0.3         999        1      10.1.0.4                                12446 10.1.0.4                                12446 MYSDWAN-LAB123          biz-internet    No    up     0:20:59:37 0           
vsmart  dtls 1.0.0.3         999        1      10.1.0.4                                12446 10.1.0.4                                12446 MYSDWAN-LAB123          mpls            No    up     0:07:45:45 0           
vbond   dtls 0.0.0.0         0          0      10.1.0.3                                12346 10.1.0.3                                12346 MYSDWAN-LAB123          biz-internet    -     up     0:20:59:38 0           
vbond   dtls 0.0.0.0         0          0      10.1.0.3                                12346 10.1.0.3                                12346 MYSDWAN-LAB123          mpls            -     up     0:07:48:46 0           
vmanage dtls 1.0.0.1         999        0      10.1.0.1                                12646 10.1.0.1                                12646 MYSDWAN-LAB123          biz-internet    No    up     0:20:59:38 0           

```

El plano de datos también se estableció usando MPLS y Biz-internet

```
Lisbon_10-1#sh sdwan bfd sessions 
                                      SOURCE TLOC      REMOTE TLOC                                      DST PUBLIC                      DST PUBLIC         DETECT      TX                              
SYSTEM IP        SITE ID     STATE       COLOR            COLOR            SOURCE IP                                       IP                                              PORT        ENCAP  MULTIPLIER  INTERVAL(msec  UPTIME          TRANSITIONS   
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1.1.20.1         20          up          biz-internet     mpls             31.11.0.2                                       21.21.0.2                                       12346       ipsec  7           1000           0:07:48:41      0             
1.1.20.2         20          up          biz-internet     mpls             31.11.0.2                                       21.22.0.2                                       12346       ipsec  7           1000           0:07:48:41      0             
1.1.200.1        200         up          biz-internet     mpls             31.11.0.2                                       21.201.0.2                                      12346       ipsec  7           1000           0:21:01:36      1             
1.1.200.2        200         up          biz-internet     mpls             31.11.0.2                                       21.202.0.2                                      12346       ipsec  7           1000           0:21:01:36      1             
...                  
1.1.20.1         20          up          mpls             mpls             172.17.1.5                                      21.21.0.2                                       12346       ipsec  7           1000           0:07:47:43      0             
1.1.20.2         20          up          mpls             mpls             172.17.1.5                                      21.22.0.2                                       12346       ipsec  7           1000           0:07:47:43      0             
1.1.200.1        200         up          mpls             mpls             172.17.1.5                                      21.201.0.2                                      12346       ipsec  7           1000           0:07:47:43      0             
1.1.200.2        200         up          mpls             mpls             172.17.1.5                                      21.202.0.2                                      12346       ipsec  7           1000           0:07:47:43      0             
```

## Consideraciones

Al desplegar TLOC Extension, ten en cuenta lo siguiente:

### Usa conexiones directas siempre que sea posible
Un cable directo entre routers es preferible a un switch compartido para minimizar puntos de falla.  

### Usa sub-interfaces si no hay suficientes interfaces
En situaciones donde los dispositivos no tienen puertos disponibles, puedes usar TLOC extension de capa 2 con sub-interfaces; solo ten presente que el ancho de banda será compartido.  

### Planea para la redundancia
Los sitios críticos funcionan mejor con conexiones directas a los transportes disponibles. TLOC Extension ayuda a extender transportes, pero no protege en caso de fallas de dispositivo.  

### Anuncia las subredes de TLOC extension
Para que tus TLOCs privados extendidos tengan conectividad, la subred local debe ser alcanzable en el transporte privado. Puedes usar BGP, rutas estáticas u otro método de enrutamiento.  

### Evita sobrecargar el enlace LAN
El enlace entre routers debe dimensionarse adecuadamente, ya que todo el tráfico extendido pasará por él.  

### TLOC Extension en Capa 3
TLOC Extension puede configurarse en capa 3 con encapsulación GRE. Esto es especialmente útil cuando dos dispositivos no tienen adyacencia directa en capa 2.  

## Conclusión

TLOC Extension es una función útil en Cisco SD-WAN que mejora la flexibilidad y resiliencia en sucursales. Al permitir que los routers compartan conexiones de transporte, optimiza el uso de recursos, reduce costos y asegura que todos los routers puedan participar plenamente en la red SD-WAN, incluso cuando los enlaces WAN dedicados son limitados.  

Una planificación adecuada, redundancia y monitoreo son esenciales para obtener el máximo valor de esta función. Cuando se implementa correctamente, TLOC Extension puede mejorar significativamente la solidez y rentabilidad de una implementación SD-WAN.

👉 Nos vemos en el próximo blog, donde seguiremos explorando formas prácticas de fortalecer y optimizar tu diseño de Cisco SD-WAN.