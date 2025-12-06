---
author: Alex
category:
  - sdwan
  - policies
draft: false
date: "2025-12-04T00:00:00+00:00"
title: "Cisco SD-WAN Policy Guide (2025): UX 2.0 Update"
description: "Guía actualizada de políticas SD-WAN: aprende Policy Groups, Topology y Application Catalog en Catalyst SD-WAN UX 2.0."
summary: "Guía actualizada de políticas SD-WAN: aprende Policy Groups, Topology y Application Catalog en Catalyst SD-WAN UX 2.0."
url: /policies-update-es/
tag:
  - policies
---

## Introducción

En el mundo de Cisco SD-WAN, las políticas juegan un papel central en cómo se anuncian las rutas dentro de la red overlay y cómo se maneja el tráfico. Antes, las políticas se dividían principalmente en **políticas de control** (para routing/topología) y **políticas de datos** (para la manipulación del tráfico).  

Con la llegada de la interfaz **SD-WAN User Interface 2.0 (UX 2.0)**, Cisco introdujo nuevos conceptos de políticas: **Policy Groups, definiciones de Topology** y un **Application Catalog**. Esta actualización tiene como objetivo explicar esos nuevos conceptos.  

Comparemos los tipos de políticas clásicas con el nuevo enfoque de UX 2.0.

## Tipos de Políticas Clásicas: Control vs Datos

### Políticas Centralizadas vs Localizadas

**Políticas Centralizadas** – configuradas en el controlador (vSmart) y aplicadas a routers SD-WAN seleccionados. Incluyen políticas de control, políticas de datos y app-aware routing policies.

**Políticas Localizadas** – configuradas directamente en routers individuales; afectan únicamente a ese dispositivo. Incluyen ACLs locales, route policies, configuraciones de QoS, etc.

### Políticas de control

Operan en el **plano de control**. Definen cómo se intercambia la información de routing, qué rutas se permiten o filtran, cómo funciona la publicación de TLOC (tunnel-locator), etc.  

Usos comunes: definir la topología del overlay (hub-and-spoke vs mesh), controlar qué rutas se comparten entre sitios/VPNs, filtrar o etiquetar rutas en OMP.

### Políticas de datos

Operan en el **plano de datos**. Definen cómo se reenvía el tráfico real; hacen match por IP origen/destino, puertos, aplicaciones, QoS, bloqueo de tráfico, direccionamiento de tráfico por aplicación, etc.  

Usos comunes: direccionamiento basado en aplicaciones, priorización, marcado/policing de QoS, control de caminos personalizados.

## Novedades en UX 2.0: Policy Groups, Topology y Application Catalog

Con UX 2.0, Cisco cambió el modelo de políticas. En lugar de múltiples conceptos repartido en diferentes páginas, ahora se utilizan **Policy Groups**, que agrupan diferentes sub-tipos de políticas. Además, la opción de **Topology** modifica cómo se manejan las políticas de control. Finalmente, el **Catálogo de Aplicaciones** facilita la gestión de aplicaciones predeterminadas y personalizadas dentro de la red.

### Policy Groups

Un **Group de Politicas** es una colección reutilizable de distintos perfiles de políticas: application priority/SLA, Next-Gen FW (URL filtering, IPS, etc.), Secure Internet Gateway (SIG) y DNS Security.

![](/wp-content/uploads/2025/12/ux-pol-1.png)

Una vez creado, un Policy Group puede asociarse a uno o varios sitios o dispositivos (gestionados dentro de un **Configuration Group**) y luego desplegarse.

![](/wp-content/uploads/2025/12/ux-pol-2.png)

Puedes elegir entre layouts **básico** y **avanzado**. El layout básico facilita una configuración rápida, mientras que el avanzado ofrece más control y opciones adicionales.

### Topology

UX 2.0 incluye una política de **Topología** donde defines la estructura del overlay. Cuando creas una política de Topología, eliges el tipo de topología (mesh, hub-spoke, custom), seleccionas los VPN relevantes y asignas los sitios (hub, spokes) o listas de sitios.

![](/wp-content/uploads/2025/12/ux-pol-3.png)

En versiones recientes, también puedes usar asignación basada en **tags**, permitiendo incluir dispositivos en topologías mediante etiquetas. Esto da mayor flexibilidad, especialmente en despliegues dinámicos o de gran escala.

Solo puede existir una definición de topología activa por grupo (Topology reemplaza el antiguo constructo de “control policy”).

### Application Catalog

UX 2.0 introduce un **Catálogo de Aplicaciones**, un repositorio donde puedes gestionar todo lo relacionado con aplicaciones: aplicaciones personalizadas, listas, nbar protocol packs y más.

![](/wp-content/uploads/2025/12/ux-pol-4.png)

Al configurar políticas, puedes referenciar listas de aplicaciones desde el Catálogo de aplicaciones, facilitando la reutilización de esas definiciones a través de distintos tipos de políticas.

![](/wp-content/uploads/2025/12/ux-pol-5.png)

El catálogo soporta aplicaciones predeterminadas (Cisco-provided), aplicaciones personalizadas e incluso aplicaciones obtenidas desde la nube.

## Beneficios del Enfoque UX 2.0

- **Workflows unificados**: En lugar de gestionar por separado data, AAR y security policies, construyes un solo Policy Group con lo relevante para un sitio o grupo de dispositivos. Esto reduce complejidad y errores.

- **Despliegue y gestión de cambios más simple**: Después de configurarlo, puedes desplegar Policy Groups a uno o varios dispositivos/sitios, facilitando pruebas antes de una implementación global.

- **Migración semi-automatizada y coexistencia**: UX 2.0 soporta migración desde modelos legacy mediante una herramienta de conversión, permitiendo coexistencia durante la transición.

- **Mejor visibilidad y gestión**: La nueva UI ofrece dashboards mejorados, visualizaciones basadas en topología, troubleshooting más sencillo y un catálogo centralizado de aplicaciones/políticas.

## Mejores Prácticas

- Usa nombres descriptivos para tus Policy Groups y perfiles asociados de Topology o aplicaciones.
- Aprovecha el **Catalog de Aplicaciones**: utiliza listas predefinidas cuando sea posible; crea aplicaciones personalizadas cuando sea necesario.
- Utiliza **topologías basadas en tags** para simplificar despliegues a gran escala y expansiones futuras.
- Para migraciones desde modelos legacy, considera un despliegue por fases; utiliza el despliegue selectivo disponible en UX 2.0.
- Siempre prueba nuevos Policy Groups en un lab o piloto limitado antes de extenderlo a toda la red.

## Conclusión

El modelo de políticas en Cisco SD-WAN ha recibido mejoras significativas en la UX 2.0. Estos cambios responden a retroalimentación real proveniente de la industria y buscan simplificar cómo se diseñan, prueban y gestionan las políticas. Aunque toda migración introduce cierta fricción, la herramienta de conversión realiza gran parte del trabajo, dejando sólo los ajustes finales al usuario. En conjunto, los beneficios superan ampliamente el esfuerzo necesario para adoptar UX 2.0.

Si eres nuevo en SD-WAN, recomiendo comenzar directamente con UX 2.0, no te vas a arrepentir 😉
