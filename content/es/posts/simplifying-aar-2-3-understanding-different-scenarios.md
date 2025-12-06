---
author: Alex
category:
  - AAR
  - Políticas
  - SD-WAN
date: "2024-02-07T20:27:37+00:00"
title: 'Application Aware Routing (AAR): 2/3 Analizando diferentes escenarios'
description: Aprende cómo AAR puede ayudarte a mejorar la experiencia del usuario y aplicación con Cisco SD-WAN  
summary: Aprende cómo AAR puede ayudarte a mejorar la experiencia del usuario y aplicación con Cisco SD-WAN  
url: /simplificando-aar-analizando-diferentes-escenarios/

---
### Introducción

Bienvenido de nuevo a la segunda entrega de mi serie en Application Aware Routing (AAR). En mi [publicación anterior](/simplificando-aar-1-3-las-bases/), discutimos los conceptos básicos de BFD y SLAs, estableciendo las bases para comprender cómo AAR optimiza el rendimiento de la red en función de los requisitos de las aplicaciones. También tocamos brevemente la configuración de AAR con un ejemplo simple.

Ahora, vamos a ver en profundidad diferentes configuraciones de AAR cuando no hay túneles que cumplan con los SLAs, más específicamente nos concentraremos en el comportamiento de _strict/drop_ y _Preferred Backup SLA_.

### Topología y configuración inicial

Comencemos con una topología simple

![](/wp-content/uploads/2024/01/aar-topology.png)

No estoy limitando los túneles entre los colores, por lo que tenemos un total de 4 túneles en cada borde WAN.

- mpls - mpls
- mpls - biz-inet
- biz-inet- mpls
- biz-inet - biz-inet

Este es el Hello interval de BFD y la configuración de app-route para todos los dispositivos; esto es lo más relevante para AAR. Si no estás familiarizado con ello, te recomiendo que revises mi [última publicación](/simplificando-aar-1-3-las-bases/) antes de continuar.

```
hello-interval 1000
bfd app-route multiplier 3
bfd app-route poll-interval 120000
```

### Escenario 1: SLA not met - Strict/Drop

Nuestra política de AAR está usando el SLA Business-Critical. Vou a introducir **pérdida de paquetes** para demostrar cómo cambia el tráfico.

```
BR10#show sdwan policy from-vsmart
from-vsmart sla-class Business-Critical
 loss    1
 latency 250
 jitter  100
from-vsmart app-route-policy _VPN10_AAR
 vpn-list VPN10
  sequence 1
   match
    source-ip      172.16.10.0/24
    destination-ip 172.16.20.0/24
   action
    sla-class       Business-Critical
    sla-class strict
    sla-class preferred-color mpls
  sequence 11
   match
    source-ip      172.16.20.0/24
    destination-ip 172.16.10.0/24
   action
    sla-class       Business-Critical
    sla-class strict
    sla-class preferred-color mpls
from-vsmart lists vpn-list VPN10
 vpn 10
```

Leamos lo que dice la documentación sobre nuestra configuración:

`sla-class preferred-color mpls`

> sla-class _sla-class-name_ preferred-color _color_ - Para configurar un túnel específico para el tráfico de datos que cumpla con una clase de SLA, incluye la opción preferred-color, especificando el color del túnel preferido. Si más de un túnel cumple con el SLA, el tráfico se enviará a través del túnel preferido. Si no hay un túnel del color preferido disponible, el tráfico se enviará a través de cualquier túnel que cumpla con la clase de SLA. Si ningún túnel cumple con el SLA, el tráfico de datos se enviará a través de cualquier túnel disponible.
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-aware-routing.html

Aquí hay un diagrama para visualizarlo mejor:

![](/wp-content/uploads/2024/01/arr100.png)

Ten en cuenta que incluso si tanto Biz-Internet **como** MPLS cumplen con el SLA, solo se utilizará MPLS.

Sí, Biz-Internet se utilizará incluso si no está especificado como un color preferido **si** el color preferido no cumple con el SLA. Desde mi experiencia, esta es una fuente frecuente de confusión, ya que la expectativa suele ser que, si MPLS no cumple con el SLA, se ejecutará la acción de _SLA not met_ sin considerar el resto de los colores.

`sla-class strict`

> Haz clic en **Strict/Drop** para hacer un match estricto de la clase SLA. Si no hay un túnel de plano de datos disponible que satisfaga los criterios de SLA, se tira el tráfico.
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-ware-routing.html

Ahora el diagrama se ve así:

![](/wp-content/uploads/2024/01/aar101.png)

#### Prueba 1 - MPLS/Biz-internet cumplen SLA

Inicio el tráfico de Branch 10 -> Branch 20 y lo capturo con NWPI. Tanto MPLS como Biz-Internet están teniendo métricas de KPI perfectas (0 pérdida, 0 latencia, 0 jitter)

```
BR10-PC1#ssh -l admin 172.16.20.2
Password:
```

![](/wp-content/uploads/2024/02/aar-1.png)

Podemos ver que el color _Actual Color_ `mpls` y _Tunnel Match Reason_ is `Matched sla and pref encap color`. Todo funcionando de acuerdo con nuestra definición de política.

#### Prueba 2- Biz-internet cumple, MPLS no cumple con SLA

Introduzco el 3% de pérdida de paquetes en el enlace MPLS. Veamos qué pasa.

![](/wp-content/uploads/2024/02/aar-25.png)
![](/wp-content/uploads/2024/02/aar-4-e1706772628492.png)

Ahora, nuestro túnel MPLS está por encima del 1% de pérdida, por lo que ya no es elegible. Confirmamos que el túnel biz-internet se usa porque cumple con el sla - _Tunnel match reason_ es ` matched sla and color any`

#### Prueba 3-MPLS/Biz-internet no cumplen SLA

Nuestra última prueba para este escenario será estropear los SLA para ambos transportes. Nuevamente, introduzco el 3% de pérdida de paquetes en **ambos** MPLS y Biz-internet.

![](/wp-content/uploads/2024/02/aar3-e1706772589421.png)

Como se esperaba, ahora el tráfico se está tirando en BR10, ya que no hay transportes que coincidan con el SLA. Nota como _DROP\_REPORT_ indica `SdwanDataPolicyDrop`

### Escenario 2: Backup SLA Preferred Color

Exploremos un nuevo escenario, agregaremos un transporte adicional.

![](/wp-content/uploads/2024/02/aar-topo2.png)

Nuestra configuración de políticas tendrá ligeros cambios. Es importante destacar que, al usar la opción _Backup SLA preferred color_, la única acción disponible cuando el SLA no se cumple será _Load Balance_.

```
BR10#show sdwan policy from-vsmart
from-vsmart sla-class Business-Critical
 loss    1
 latency 250
 jitter  100
from-vsmart app-route-policy _VPN10_AAR
 vpn-list VPN10
  sequence 1
   match
    source-ip      172.16.10.0/24
    destination-ip 172.16.20.0/24
   action
    backup-sla-preferred-color private1
    sla-class       Business-Critical
    no sla-class strict
    sla-class preferred-color mpls
  sequence 11
   match
    source-ip      172.16.20.0/24
    destination-ip 172.16.10.0/24
   action
    backup-sla-preferred-color private1
    sla-class       Business-Critical
    no sla-class strict
    sla-class preferred-color mpls
from-vsmart lists vpn-list VPN10
 vpn 10
```

Nuevamente, veamos qué dice la documentación:

`backup-sla-preferred-color private1`

> **Cuando ningún túnel cumple con el SLA**, se puede elegir cómo manejar el tráfico:
>
> backup-sla-preferred-color _colors_: Dirige el tráfico de datos a un túnel específico. El tráfico de datos se envía a través del túnel configurado si la interfaz de dicho túnel está disponible; si no lo está, el tráfico se enviará por otro túnel disponible. Se pueden especificar uno o más colores.
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-aware-routing.html

Para ponerlo visualmente

![](/wp-content/uploads/2024/02/aar200.png)

#### Prueba 1- MPLS/Private1 complen con el SLA, Biz-Internet no cumple

Comenzaremos con la suposición de que Biz-Internet no cumple con el SLA, por lo que tenemos dos transportes disponibles: MPLS y Private1. Inicio el tráfico.

```
BR10-PC1#ssh -l admin 172.16.20.2
Password:
```

![](/wp-content/uploads/2024/02/s2-aar1-e17067776034732.png)

Nota como _SLA Strict_ tiene un varlo de `No` y el MPLS cumple con el SLA y se usa.

#### Prueba 2 - Private1 cumple con el SLA, MPLS/Biz-internet no cumplen

Para la segunda prueba, estamos introduciendo pérdida en el transporte MPLS. Con esto, el único transporte que cumple con el SLA es Private1, que también es el color preferido de respaldo para el SLA. Veamos cómo se ve.

![](/wp-content/uploads/2024/02/s2-aar101.png)

En el _Tunnel Match Reason_ podemos ver claramente que el SLA se cumple a través de un color que no es el preferido (Private1). ¿Qué crees que sucederá si Private1 no cumple con el SLA?

#### Prueba 3 - MPLS/Private1/Biz-Internet No confunde

Private1 es el único transporte que cumple con el SLA, me encargaré de eso al introducir la pérdida de paquetes.

![](/wp-content/uploads/2024/02/s2-102.png)

De nuevo (!) Private1 se usa, pero ahora el tráfico coincide con el SLA por defecto, en otras palabras, no hay túneles que coincidan con el SLA, por lo que se utilizará el túnel marcado _backup preferred_.

#### Bono

**Private1 no disponible**  
El resultado después de apagar la interfaz Private1, aún sin ningún túnel que cumpla con el SLA. Podemos ver que se realiza una coincidencia flexible en los túneles (se puede elegir cualquier túnel disponible).

**Biz-internet cumple, MPLS/Private1 no cumplen**  
El resultado después de que biz-internet vuelve a tener valores de pérdida cero, mientras que private1 no cumple con el SLA. Aunque no sea el color preferido, aún cumple con el SLA, por lo que es seleccionado.

![](/wp-Content/uploads/2024/02/S2-103.png)

![](/wp-content/uploads/2024/02/s2-104.png)

Como nota final, ten en cuenta que AAR siempre intentará usar los túneles que cumplan con el SLA especificado, **no te confundas solo porque los nombres de los colores no están explícitamente mencionados en la configuración.**

En la [Siguiente publicación](/simplificando-aar-3-3-fallback-to-best-path), exploraremos la opción restante _Fallback to Best Path_. ¡Nos vemos allí!