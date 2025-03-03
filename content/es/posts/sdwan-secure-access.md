---
author: Alex
category:
  - sdwan
  - seguridad
  - secure access
date: "2025-02-05T14:20:31+00:00"
title: Asegurando el borde con Cisco SD-WAN y Secure Access
summary: Descubre cómo Cisco SD-WAN y Cisco Secure Access trabajan juntos para mejorar el rendimiento y la seguridad de los usuarios en internet.
url: /sdwan-sse-integración /
tag:
  - sse
  - sdwan
  - seguridad
---
## Introducción

La seguridad siempre ha sido una prioridad para las organizaciones, pero proteger cada ángulo de la red sigue siendo un desafío. Al mismo tiempo, garantizar una experiencia óptima para aplicaciones y usuarios es igualmente importante. Las organizaciones a menudo han tenido que elegir entre soluciones centradas en la seguridad o en el rendimiento, lo que aumenta la complejidad de gestión y operación.

Cisco Secure Access es una solución robusta que aborda estos desafíos. Ofrece una seguridad de primer nivel al integrar tecnologías avanzadas y controles de acceso. Esto significa que los usuarios pueden obtener una conectividad **_segura y directa_** desde los sitios SD-WAN hacia internet y aplicaciones SaaS.

Veamos cómo funciona.

## SD-WAN se encuentra con Secure Access

A partir de la versión 20.13/17.13 de SD-WAN, la integración con Secure Access está disponible de forma nativa.

Con esta integración, se pueden establecer túneles IPSec automáticos hacia los Data Centers primarios y secundarios de Secure Access que estén mas cercanos a la ubicación de tu router, garantizando un rendimiento óptimo. Estos túneles enrutan el tráfico mientras aplican las políticas de seguridad de tu organización, proporcionando una forma sencilla y potente de mejorar tanto la seguridad como la conectividad a internet.

En el centro de Secure Access están los Network Tunnel Groups [(NTGs)](https://docs.sse.cisco.com/sse-user-guide/docs/manage-network-tunnel-groups), que gestionan las conexiones IPSec. Cada NTG incluye un Data Center de Secure Access primario y uno secundario. Aunque no es obligatorio configurar túneles hacia ambos, es altamente recomendable para garantizar alta disponibilidad en caso de que uno tenga algún problema

![](/wp-content/uploads/2025/02/sse-topo.png)

Es posible configurar hasta 16 túneles, 8 activos y 8 backups, lo que permite hacer _load balance_ entre los túneles activos para aumentar el ancho de banda disponible. 

Para establecer los túneles _automáticamente_, el Manager y el router necesitan conectividad a Internet y DNS habilitado. Esto permite que el router determine su propia dirección IP pública, se la comunique al Manager y se le asignen los Data Centers SSE primario y secundario más cercanos.

![](/wp-content/uploads/2025/02/get-ip.png)

Una vez que se establecen los túneles, el tráfico que los atraviesa está asegurado por las [funcionalidades de seguridad principales de SSE](https://www.cisco.com/c/en/us/products/collateral/security/secure-access/hybrid-workforcecloud-agile-security-s.html#ciscurezaccessprechovers) FWAAS, CASB, ZTNA y SWG y más.  

## Pasos de configuración

### Crear clave API 

Para comenzar, en SSE [crea una clave API](https://docs.sse.cisco.com/sse-user-guide/docs/add-secure-access-api-keys) para que el Manager se conecte de forma segura. Asegúrate de que se otorguen los siguientes privilegios:

- Deployment / Network Tunnel Group - **Read/Write**
- Deployment / Tunnels - **Read/Write**
- Deployment / Regions - **Read**

Obtendrás el _API Key y Key Secret_

### Ingrese credenciales en el Manager
A continuación, ingresa la información en el Manager en **_Administration > Settings > Cloud Credentials > SSE_**

![](/wp-content/uploads/2025/02/sse-config.png)

### Crear política SSE en el Manager

Crea una nueva política de SSE. Aquí es donde ingresas la información sobre los túneles IPSec. 

Desde **_Configuration > Policy Groups > Secure Service Edge_**

Lo siguiente es lo mínimo que necesitas:
- Tracker IP - Se utiliza para confirmar que el túnel está arriba.
- Tunnel - Al menos 1 túnel. Ingresa el nombre _tunnel name, source interface_ y selecciona _primary/secondary DC_
- Interface Pair - Especifica los túneles _Active y Backup_. Selecciona _none_ como backup si solo hay 1 túnel. 

**Nota** Puedes seleccionar la región SSE de tu preferencia o usar _auto_ para seleccionarla automáticamente.

![](/wp-content/uploads/2025/02/tunnel-config.png)

Luego, crea un _Policy Group_, asocia un dispositivo y agrega la política de SSE

![](/wp-Content/uploads/2025/02/Policyg.png)

### Redirigir el tráfico del lado del servicio

Ahora, necesitamos redirigir el tráfico de los usuarios al túnel. Hay dos opciones:

#### Ruta de servicio de tipo SSE 

En la funcionalidad _service vpn_, agrega una ruta _service_ con el proveedor de **_SSE Cisco-Secure Access_**

![](/wp-content/uploads/2025/02/static-sse.png)

Esto se agregará de la siguiente manera:

```
ip sdwan route vrf 10 0.0.0.0/0 service sse Cisco-Secure-Access
```

Puede seleccionar qué tráfico se reenvía a SSE modificando la ruta, sin embargo, este enfoque no es tan flexible  comparado con la segunda opción.

#### Data Policy

Las Data Policies proporcionan más flexibilidad para redirigir el tráfico. No solo podemos hacer match al destino, sino también a otros elementos útiles, como _source, aplicaciones_ y más.

![](/wp-content/uploads/2025/02/match-sse.png)

Este ejemplo hace match a todo el tráfico que proviene de mis usuarios en la VPN 10 y establece una acción de |`Secure Service Edge`. Observa que podemos habilitar la opción de _Fallback to Routing_ en caso de que los túneles no estén disponibles. 

![](/wp-content/uploads/2025/02/data-sse.png)

### Validaciones en el Manager

Verifica el estado del túnel desde **_Monitor > Tunnels > SIG/SSE Tunnels_**

![](/wp-content/uploads/2025/02/verify1.png)

Los logs están disponibles y son útiles para determinar si hay algún problema al establecer los túneles

![](/wp-content/uploads/2025/02/verify2.png)

### Validaciones del usuario

Para verificar que los usuarios están utilizando SSE, tenemos un par de opciones

La primera es visitar policy.test.sse.com, si el tráfico se redirige correctamente, verás algo como esto:

![](/wp-Content/uploads/2025/02/Policy-sse.png)

En el lado de SSE, hay una política que niega el tráfico a las aplicaciones de redes sociales, veamos el resultado de tratar de acceder a x.com 

![](/wp-content/uploads/2025/02/x-sse.png)

Por último, acceder a welcome.umbrella.com nos hará saber si el usuario está protegido  

![](/wp-Content/uploads/2025/02/Umbrella-sse.png)

## Consideraciones adicionales

- ECMP está disponible cuando múltiples túneles están activos.
- _Unequal Load Balance_ se puede lograr mediante la asignación de peso a los túneles IPSEC. 
- El _fallback to routing_  se activa cuando los túneles no están disponibles
- Los trackers son personalizables - Se puede definir una URL y _thresholds_ personalizados para cumplir con los SLA deseados
- Mecanismo de _dampening_ incorporado en túneles para evitar flapping entre túneles.
- La redirección de la política de datos proporciona una mayor flexibilidad que las rutas estáticas
- Usa interfaces Loopback para crear múltiples túneles hacia SSE

¡Espero que hayas aprendido algo útil! Nos vemos en el siguiente 👋