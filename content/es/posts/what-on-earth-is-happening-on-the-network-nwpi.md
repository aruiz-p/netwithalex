---
author: Alex
category:
  - SD-WAN
  - visibilidad
date: "2024-01-25T20:34:14+00:00"
title: Cisco SD-WAN Network Wide Path Insights (NWPI)
summary: Aprende a usar la herramienta de troubleshooting más avanzada para tu red Cisco SD-WAN. 
URL: /introduccion-network-wide-path-insights/

---
### Motivación: El escenario clásico de crisis

Imagina comenzar el día en el equipo de redes, solo para ser bombardeado con quejas sobre la principal aplicación interna. Empiezas a investigar. ¿Dónde se encuentra el problema? ¿Está aislado o afectando múltiples lugares? ¿Cuándo comenzó el problema? ¿Cuál es el impacto?

A continuación, intentas conectar con alguien que te ayude a verificar los conceptos básicos. DHCP y DNS funcionan. Gateway es accesible. ¡La conectividad a otros objetivos en el mismo DC es intermitente! Está empezando a ser raro ...

En este punto, cada minuto cuenta y un proceso de solución de problemas adecuado debe implementarse para verificar todos los dispositivos involucrados y aislar la raíz del problema. Comienza a tomar capturas de paquetes y trazas en diferentes puntos de la red, verificas los contadores, las sesiones BFD e IPSEC, buscas inconsistencias en la tabla OMP y enrutamiento, verificas largas configuraciones de políticas y parámetros de puerto, una cosa a la vez. Dos horas más tarde (¡con suerte!) tú y tu equipo finalmente llegan a la causa raíz...

### ¿Qué hizo Cisco al respecto?

Teniendo en cuenta el nivel de complejidad y los esfuerzos necesarios para resolver los problemas de la red, Cisco creó - _**Network Wide Path Insights**_ (NWPI) - para facilitar la solución de problemas de nuestra WAN. NWPI se introdujo por primera vez la versión 20.4 y con cada lanzamiento siguió mejorando. En la versión 20.9 hay varias mejoras importantes que lo ayudarán a determinar rápidamente lo que está sucediendo. ¿Quieres verlo en acción? ¡Vamos!

### ** Topología **

Usaremos este escenario para ejecutar nuestro traza NWPI. No existe una política centralizada, como resultado, tenemos un full mesh y el tráfico podría fluir en cualquiera de los túneles disponibles.

![](/WP-Content/uploads/2024/01/captshot-2024-01-25-AT-07.24.14.png)

### ** Comprensión de NWPI **

En VManage, navegue a la sección _Tools_ para encontrar esta característica. Para comenzar a usarlo, la información mínima que necesita es:

- Donde se genera el tráfico (`Site`` ID`)
- segmento de la red (`VPN`)

_Device_ se determina automáticamente. Puedes refinar aún más los filtros para capturar exactamente lo que está buscando.

![](/wp-content/uploads/2024/01/captshot-2024-01-24-AT-08.41.17.png) 20.12.2 NWPI

Una vez que los filtros estén en su lugar, comienza la traza y observa la magia.

#### Insight - Vista básica

Veamos los resultados de una pequeña transferencia **SCP** entre sitios 10 y 20. Esta es la primera información que veremos.

![](/wp-content/uploads/2024/01/capshot-2024-01-25-AT-07.36.03.png)

De lo anterior, podemos determinar:

- ** Dirección de flujo ** \- Tráfico que fluye del sitio 10 a 20
- ** Equipos edges tocando el tráfico ** \- BR10-1 y BR20-2
- ** Información de flujo ** \- src/dst ips, puertos, protocolo, aplicación, etc.
- ** Colores involucrados ** \- MPLS -> Biz- Internet
- ** KPIs SD-WAN específicos del flujo ** \- Pérdida de paquetes, latencia y jitter
- ** Porcentaje de pérdida de paquetes ** \- en los dispositivos WAN y en los dispositivos

Hagamos clic en _readout_ para obtener más información

#### Leer

![](/wp-content/uploads/2024/01/capshot-2024-01-25-AT-07.57.10.png)

Una nueva información está disponible para nosotros:

- ** Entrada coincidente en la tabla de enrutamiento ** \- 172.10.20.0/24 Provenía de OMP y sus respectivas métricas.
- ** Candidato y ruta elegida ** \- Rutas disponibles que se muestran y elegidas una resaltada en verde.
- ** Interfaces físicas involucradas ** \- tanto para el servicio como para el transporte
- ** Razón para elegir esta ruta ** \- Enrutamiento, sin embargo, tenga en cuenta que las políticas pueden anular la tabla de enrutamiento.

Si nos detenemos aquí, ya tenemos mucha información muy útil para comprender el flujo del tráfico, pero ¿qué pasa si necesitamos profundizar? Bueno, ahora exploremos las vistas _Advanced_

#### Vistas avanzadas

![](/wp-content/uploads/2024/01/capshot-2024-01-25-AT-08.16.33.png)

Si conoce [DataPath Packet Trace] (https://www.cisco.com/c/en/us/support/docs/content-networking/adaptive-session-redundancy-asr/117858-technote-asr-00.html#toc-hid-180344474), esta información será familiar. Esencialmente, nos dirá todas las funcionalidades que ejecuta el dispositivo a medida que se procesa el paquete. Algunos ejemplos podrían ser ACL, políticas, reglas FW, DPI, Netflow y mucho más. Hay casos en los que necesitamos determinar si o por qué el dispositivo está dejando caer paquetes, ¡este es el lugar para verificar!

En resumen aquí está la información adicional que podemos obtener:
- **Funcionalidades** \- Algunas de ellas dependerán de la configuración, otras siempre estarán allí en un entorno SD-WAN.
- **Drops** \- Si una característica está dejando caer el tráfico, tiene la información para saber exactamente por qué.
- **Detalles de bajo nivel sobre las funcionalidades** \- La mayoría de las veces no tendrá que lidiar con esto, pero puede ser útil cuando se comunica con el soporte técnico.

Hay mucho más que hacer con esta herramienta, pero creo que esto es suficiente como introducción. Sin embargo, antes de llegar a la conclusión, me gustaría mencionar algunos de los casos de uso en los que esta herramiento es extremadamente útil.

- Mal rendimiento de la aplicación
- Validación de políticas
- Aislamiento del problema
- Validación/solución de problemas DIA y SaaS (sí, también puede brindar información sobre el tráfico destinado a Internet)

¿En qué otros escenarios podrías pensar?

### Conclusión

NWPI es un gran ejemplo del esfuerzo de Cisco para crear una herramienta que puede ayudar a solucionar problemas más rápido y de manera simple y eficiente. Consulte esta [guía](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/network-wide-path-insight/network-wide-path-insight-user-guide/m-network-wide-path-insight.html) para saber más sobre las características introducidas y más. 

En mi experiencia, NWPI no se usa lo suficiente principalmente porque todavía es desconocido para muchos. Te animo a que lo pruebes y eventualmente lo incorpores a su conjunto de herramientas de solución de problemas, estoy seguro de que encontrará algún beneficio.