---
author: Alex
category:
  - sd-routing
date: "2024-02-24T12:06:47+00:00"
title: 'Un nuevo capítulo: SD-Routing revoluciona la gestión de red'
summary: Explora cómo SD-Ruting puede ayudarte de manera simple y efectiva a administrar y monitorear equipos autónomos (IOX-XE regular, sin SD-WAN). 
url: /sd-routing-revoluciona-la-gestion-de-red/

---
## Introducción

Las Redes Definidas por Software (SDN) llegaron con la promesa de simplificar la administración de la red, permitiendo a los equipos de redes automatizar y adoptar un enfoque programático. Cisco creó soluciones como Catalyst Center, ACI, Meraki y _Viptela SD-WAN_. Esta última introdujo nuevos controladores que cambiaron las reglas que rigen el funcionamiento de la red WAN.

Las organizaciones que no estaban listas para dar el salto completo a SD-WAN, pero que aún deseaban los beneficios de los principios de SDN, tenían opciones limitadas, como integrarse con sistemas de terceros o depender de Catalyst Center. Aunque estas soluciones resolvían algunos desafíos, quedó una brecha significativa sin cubrir hasta que llegó... ¡SD-Routing!

## ¿Qué es SD-Routing?

En términos simples, SD-Routing es el punto medio entre SDN y SD-WAN. Con SD-Routing, las organizaciones pueden adoptar de manera gradual el enfoque de redes definidas por software en su infraestructura existente.

La idea detrás de esto es administrar la red _no-SD-WAN_ (también conocida como "tradicional") desde un _Single Pane of Glass_ llamado SD-WAN Manager (anteriormente vManage). ¡Por supuesto, también es posible gestionar dispositivos SD-WAN al mismo tiempo!

A diferencia de SD-WAN, no es necesario conectarse al SD-WAN Controller (anteriormente vSmart), por lo que los protocolos de ruteo existentes se mantienen. Además, se obtiene una capa de seguridad adicional con el SD-WAN Validator (anteriormente vBond), que se encarga de permitir la conexión solo a los dispositivos autorizados.

## Beneficios

Algunos de los beneficios de SD-Routing incluyen:

- **Onboarding** – Los nuevos dispositivos pueden integrarse de forma fácil y rápida utilizando SD-WAN Manager.
- **Configuración** – Administra la configuración de tus dispositivos desde un único lugar mediante un método reutilizable llamado Configuration Groups, que permite escalar más rápido. Además, cuenta con _workflows_ optimizados para políticas de seguridad y conectividad a la nube.
- **Monitoreo** – Supervisa tus dispositivos, sitios y aplicaciones. Recibe alertas y eventos, y aprovecha las capacidades de notificación de SD-WAN Manager.
- **Gestión de software** – Distribuye, instala y activa imágenes de software de manera sencilla y rápida en uno o varios dispositivos.
- **Troubleshooting** – Ejecuta diversas operaciones desde vManage, como sesiones SSH, pruebas de velocidad, traceroutes y más.
- **Transición a SD-WAN** – Si planeas implementar SD-WAN, SD-Routing es un excelente punto de partida para familiarizarte con SD-WAN Manager y simplificar la migración. 🚀

## Descripción general del SD-WAN Manager

Déjame darte un recorrido rápido por el SD-WAN Manager y un vistazo a algunas de las características. Si ya lo conoces, probablemente aún te sorprenderá el nuevo aspecto del 20.13. ¡Te lo enseño!

### Descripción general de la red

Cuando iniciamos sesión por primera vez, se presenta un _overview_ de la red para que podamos determinar rápidamente cuántos dispositivos y controladores están conectados, información de las aplicaciones, salúd de los equipos y más.

Oye, ¿a dónde se fue el Controller (vSmart)?

![](/wp-content/uploads/2024/02/sdr-20.png) SD-WAN Manager 20.13

La apariencia magnética te recordará a otros productos de seguridad de Cisco o Meraki, esto es genial para mantener la experiencia consistente. Observe que arriba a la derecha hay un botón que te permite activar el modo SD-Routing, esto es muy útil para mostrar la información referente a SD-Routing y la razón por la que no vemos el Controller. 

### Monitoreo de red

Si queremos ver información más detallada del dispositivo, podemos visitar la pestaña _devices_. SD-WAN Manager está constantemente actualizando esta información, por lo que tenemos una vista precisa. Nota como los dispositivos se muestran como  _**SD-Ruting**_.

![](/wp-content/uploads/2024/02/SDR-12-1.png)

BR20 no tiene una buena salud debido a la alta utilización de la memoria, ¿podríamos saber cuándo comenzó esto? Hagamos doble clic en él.

![](/wp-content/uploads/2024/02/SDR-14.png)

Comenzó a ir más allá del 75% alrededor de las 12:30, esto se debió a que activé _Performance Monitor_ para obtener información del rendimiento de las aplicaciones.

Centrémonos en el menú izquierdo, mira las opciones de **monitoreo** disponibles que incluyen **aplicaciones, funciones de seguridad** e **información en tiempo real**. La sección **Troubleshooting** es el lugar para usar las herramientas que mencioné anteriormente.

### Configuración

En las versiones 20.13/17.13 se agregó soporte para _SD-Routing Configuration Groups_. Con ellos, puedes crear _Feature Profiles_ basados en _parcels_, que son elementos individuales que, en conjunto, conforman toda la configuración del router.

En la versión 20.13, los siguientes parcels están disponibles:

![](/wp-Content/uploads/2024/02/sdr-31.png)

![](/wp-content/uploads/2024/02/sdr-30.png)

Tenemos el CLI Configuration Profile para aplicar cualquier configuración que no esté disponible a través de parcels. Podemos definir variables para hacer nuestro Profile reutilizable en múltiples dispositivos. También es posible usar un CLI Profile completo en lugar de parcels.

![](/wp-content/uploads/2024/02/sdr-33.png)

![](/wp-content/uploads/2024/02/sdr-22.png)

Por cierto, si tienes una combinación de dispositivos SD-WAN y SD-Routing, verás ambos Configuration Groups en la lista. Actualmente, SD-WAN tiene un conjunto más amplio de parcels, pero con el tiempo, SD-Routing también recibirá más configuraciones sin depender de CLI.

### Workflows

La _workflows library_ nos ayudará a realizar fácilmente ciertas acciones como _onboarding_, configuración de seguridad, actualizaciones de software, entre otras.

![](/wp-content/uploads/2024/02/sdr-23.png)

![](/wp-content/uploads/2024/02/sdr-244.png)

![](/wp-content/uploads/2024/02/sdr-200)

En lugar de hacer clic en múltiples páginas, podemos seguir un flujo paso a paso con toda la información necesaria en un solo lugar. Personalmente, me gusta cómo los workflows simplifican las cosas.

### Conectividad con la nube

Es muy probable que tu organización utilice algún tipo de conectividad a la nube, ya sea para acceder a aplicaciones o ejecutar cargas de trabajo. SD-Routing cubre esta necesidad.

Puedes automatizar la conexión con el proveedor de nube, extendiendo tu red para acceder a esos recursos sin complicaciones.

![](/wp-Content/uploads/2024/02/sdr-6.png)

![](/wp-content/uploads/2024/02/sdr-26.png)

Hay múltiples opciones. También se proporciona algo de esto para SD-WAN, por lo que recomiendo verificar la [Guía de configuración](https://www.cisco.com/c/en/us/td/docs/routers/cloud_edge/c8300/software_config/cat8300swcfg-xe-17-pook/mcloud-onroramp-sd-crouting.html) de SD-Routing.

## Conclusión

El propósito de esta publicación es mostrar lo que es posible con SD-Routing y el vacío que busca llenar. Adoptar esta tecnología puede impactar positivamente los flujos de trabajo operativos, mejorar la agilidad de la red y optimizar la utilización de recursos.

Si tu organización aún no ha adoptado ninguna forma de SDN, te invito a reflexionar sobre tus procesos diarios, identificar los principales desafíos y pensar en cómo SD-Routing podría ayudarte a resolverlos.

¡Déjame saber qué opinas y nos vemos en el próximo post! 🚀