---
author: Alex
category:
  - sdwan
  - policies
draft: false
date: "2025-07-01T00:00:00+00:00"
title: "Políticas de Control vs Política de Datos en Cisco SD-WAN: Diferencias Clave"
description: Aprende las diferencias clave entre las control y data policies en Cisco SD-WAN, cuándo usar cada una y cómo impactan en el enrutamiento y el reenvío del tráfico.
summary: Aprende las diferencias clave entre las control y data policies en Cisco SD-WAN, cuándo usar cada una y cómo impactan en el enrutamiento y el reenvío del tráfico.
url: /policies-intro-es/
tag:
  - policies
---

Aprende las diferencias clave entre las control y data policies en Cisco SD-WAN, cuándo usar cada una y cómo impactan en el enrutamiento y el reenvío del tráfico.


## Introducción

Cisco SD-WAN proporciona herramientas poderosas para definir tanto el comportamiento de enrutamiento como el flujo del tráfico a través del overlay. En el núcleo de esta flexibilidad existen dos tipos clave de políticas: **políticas de control** y **Data Policies**. Entender la diferencia entre ambas es crucial para diseñar implementaciones SD-WAN escalables y optimizadas.

En esta publicación, exploraremos qué hace cada tipo de política, en qué se diferencian y cuándo usar una sobre la otra.

## Políticas Centralizadas y Locales

Las políticas se categorizan según el lugar donde se configuran y aplican: pueden ser _Centralizadas_ o _Locales_. El Manager utiliza **netconf/yang** para enviar la configuración de políticas.  

**Políticas Centralizadas**

Se configuran en el Controller (vSmart) y se envían a un **grupo seleccionado** de routers SD-WAN a través del protocolo Overlay Management Protocol (**OMP**). Incluyen:
- políticas de control
- Data policies
- Políticas de App-Aware Routing

**Políticas Locales**

Se configuran directamente en los routers SD-WAN y **solo afectan a ese dispositivo específico**. Incluyen:
- ACLs locales
- Políticas de enrutamiento (Route policies)
- Configuración de QoS

![](/wp-content/uploads/2025/06/policies1.png)

## ¿Qué son las Políticas de Control?

Las políticas de control en Cisco SD-WAN se aplican **en el plano de control** – específicamente, definen cómo se intercambia la información de ruteo entre los Controllers y los routers SD-WAN.

**Características clave:**

* **Se aplican en los Controllers**
* Influyen en la publicación de rutas y propagación de TLOCs
* Definen qué rutas se aceptan, modifican o rechazan

**Casos de uso típicos:**

- Crear topologías (hub/spoke, partial-mesh, etc.)
- Controlar route-leaking entre VPNs
- Filtrado o etiquetado de rutas OMP

 Las políticas de control definen la topología SD-WAN, decidiendo qué rutas existen y hacia dónde pueden ir.

## ¿Qué son las Políticas de Datos?

Las políticas de datos, por otro lado, operan en el **plano de datos**. Definen cómo se maneja el tráfico real.

**Características clave:**

* También se configuran en los Controllers pero se **aplican en los routers SD-WAN**
* Operan a nivel de plano de datos, no afectan la publicación de rutas
* Pueden hacer match a varios campos: IP de origen/destino, puertos, aplicaciones, etc.

**Casos de uso típicos:**

* Redirección de tráfico según aplicación
* Aplicación de clases de QoS
* Bloqueo de ciertos tipos de tráfico (por ejemplo, mediante reglas tipo ACL)

Usando una analogía: las políticas de control son los urbanistas que deciden dónde se construyen las calles; las políticas de datos son las reglas de tránsito que dictan cómo se mueve el tráfico por ellas.

## Diferencias clave a simple vista

| Característica        | Control Policy               | políticas de datos                      |
|-----------------------|------------------------------|----------------------------------|
| **Plano**             | Plano de control             | Plano de datos                   |
| **Se aplica en**      | Controller                   | WAN Edge                         |
| **Afecta**            | Publicación de rutas         | Reenvío del tráfico real         |
| **Casos comunes**     | Filtrado de rutas, TLOCs     | Redirección de tráfico, QoS, ACLs|
| **Direccionalidad**   | Entre controllers            | Del controller a los routers     |

## Orden de ejecución de políticas

Los siguientes pasos ocurren cuando un paquete pasa por un router de SD-WAN:

![](/wp-content/uploads/2025/06/policies2.png)

Primero se aplican las **políticas locales de entrada**, como las ACLs a nivel de interfaz y configuraciones de QoS. Estas pueden filtrar paquetes y aplicar marcados o remarcar para QoS.

Después se evalúa la **política de Application-Aware Routing centralizada**. Este toma decisiones de reenvío solo entre rutas de costo igual que ya estén en la tabla de enrutamiento.

Luego se evalúa la **política de datos centralizada**, que tiene la capacidad de sobrescribir cualquier decisión de reenvío tomada por AAR.

Una vez aplicadas las políticas, el sistema realiza una búsqueda de **enrutamiento y envío** para identificar la interfaz de salida apropiada.

Se aplican los mecanismos de **queueing y scheduling** para moldear el tráfico según la configuración de QoS.

Finalmente, se aplica la **política local de salida** antes de colocar el paquete en el medio.

Entender este orden es clave para evitar errores de configuración y asegurar que tus políticas funcionen como esperas. Un error común es sobrescribir sin querer una decisión de AAR aplicando una políticas de datos centralizada que coincide con el mismo tráfico y lo redirige a otro lugar.

## ¿Cuándo usar cuál?

Elegir entre control y data policies depende de **qué problema estás tratando de resolver**:

* ¿Quieres **limitar qué rutas se comparten** entre sitios? → Usa una **políticas de control**.  
* ¿Necesitas **redirigir ciertas aplicaciones** por MPLS o DIA? → Eso es trabajo de una **políticas de datos**.  
* ¿Intentas **evitar loops de publicidad de rutas** o controlar fugas entre VPNs? → políticas de control.  
* ¿Quieres **priorizar tráfico VoIP** o bloquear tráfico P2P? → políticas de datos.

En muchos diseños, ambas se usan en conjunto. Por ejemplo, una control policy puede garantizar que solo ciertos prefijos sean alcanzables entre sitios, mientras que una políticas de datos se encarga de cómo se prioriza y reenvía ese tráfico coincidente.

## Conclusión

Las políticas de datos y control tienen propósitos distintos pero complementarios en Cisco SD-WAN. Una gobierna la **topología y decisiones de enrutamiento**, la otra define **cómo se mueve el tráfico** dentro de esa topología.

Entender estas diferencias te permite construir entornos SD-WAN más inteligentes y eficientes. ¿Quieres profundizar más? Consulta la documentación oficial de Cisco sobre [políticas de control y Data Policies](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/policy-overview.html).

También te recomiendo leer mi análisis detallado sobre [Application-Aware Routing](/simplificando-aar-1-3-las-bases/), y descubrir cómo SD-WAN puede mover automáticamente el tráfico hacia los caminos de mejor rendimiento y garantizar una buena experiencia para el usuario.
