---
author: Alex
category:
  - sdwan
  - appqoe
draft: true
date: "2025-04-22T14:20:31+00:00"
title: "Serie AppQoE: Forward Error Correction (FEC)"
description: Descubre cómo SD-WAN mejora el rendimiento de aplicaciones expuestas a transportes irregulares y con pérdidas de paquetes.  
summary: Descubre cómo SD-WAN mejora el rendimiento de aplicaciones expuestas a transportes irregulares y con pérdidas de paquetes.  
url: /appqoe-opt-tcp/
tag:
  - TCP
  - optimizacion
---
## Introduction

¿Alguna vez has usado una aplicación que se siente demasiado lenta? Tal vez las videollamadas se congelan o un portal web tarda una eternidad en cargar. Estos (y otros) son signos de que tu red WAN podría estar teniendo problemas.

¿La buena noticia? Cisco SD-WAN incluye un conjunto de tecnologías diseñadas para mejorar el rendimiento en enlaces poco confiables o con alta latencia. En esta serie, desglosaremos tres funciones clave que pueden mejorar significativamente la experiencia de las aplicaciones en tu red: **_Optimización TCP, Forward Error Correction (FEC) y Duplicación de Paquetes**_.

En este primer post, exploraremos la Optimización TCP: cómo funciona, cuándo utilizarla y por qué puede marcar una gran diferencia para tus usuarios, especialmente en conexiones con alta latencia.

## Descripción de la Solución

El objetivo de la Optimización TCP es ajustar finamente las conexiones TCP para mejorar su rendimiento. Esto es especialmente útil cuando hay enlaces con alta latencia involucrados.

Los routers SD-WAN actuarán como proxies, lo que significa que interceptarán las conexiones TCP y las ajustarán para obtener un mejor desempeño. Veamos un ejemplo visual.

![](/wp-content/uploads/2025/04/tcp-opt-topo.png)

Sin la optimización TCP, el cliente y el servidor establecerán una sesión TCP directamente entre ellos.

Cuando se utiliza la Optimización TCP, el Router 1 interceptará y terminará la conexión TCP proveniente del cliente y establecerá una sesión TCP optimizada con el Router 2. De igual manera, el Router 2 creará una sesión TCP con el servidor.

**Nota** Todo este proceso es transparente para el cliente y el servidor, y los datos serán almacenados en caché en los routers para mantener activas las sesiones.

Los equipos IOS-XE SD-WAN usan el algoritmo BBR el cual utiliza información sobre RTT (Round Trip Time) and ancho de banda disponible para optimizar la conexión. Si te gustaría profundizar en el tema te recomiendo ver [este vídeo](https://www.youtube.com/watch?v=VIX45zMMZG8&t=1607s) de Neal Cardwell. 

La implementación actual de Optimization TCP tiene definidos dos roles:

- **Controller Node:** Equipo que intercepta y distribuye el trafico al _Service Node_.
- **Service Node:** Motores de optimización para la aceleración del tráfico.

En un escenario de la vida real, la recomendación es tener servicios de optimización en las sedes y en los Data Centers. Hay requerimientos diferentes basados en el volumen de tráfico que los equipos van a procesar. Te sugiero leer la [Documentación de Cisco](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/appqoe/ios-xe-17/appqoe-book-xe/m-tcp-optimization.html) para informarte sobre los requerimientos de hardware y más.

En las sedes pequeñas, es común utilizar un _Integrated Service Node_, es decir, un solo equipo puede interceptar, distribuir y optimizar el tráfico. Por otro lado, en el Data Cetner, un cluster de [External Service Nodes](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/appqoe/ios-xe-17/appqoe-book-xe/m-support-for-multiple-appqoe-service-nodes.html) es necesario para para lograr un mayor rendimiento y distribuir volúmenes más altos de tráfico entre los miembros del cluster.

En general, la Optimización TCP es un proceso intensivo para los dispositivos, por lo que es crucial confirmar los requisitos de la plataforma. Por ejemplo, mi entorno de demostración cuenta con dos Catalyst 8000V con 8 CPUs y 16 GB de RAM, requisitos adecuados para una implementación pequeña.

Veamos en la práctica qué efecto tiene la optimización TCP en el tráfico. Para demostrarlo, voy a tomar una captura de paquetes en el lado WAN con y sin optimización. 

### Window Scaling Sin Optimización 

Veamos como se comporta el window scalind sin optimización 

![](/wp-content/uploads/2025/04/router1-tcp-opt-disabled.png)

Nota como el _window size_ se mantuvo estable alrededor de 1,000,000 Bytes después de aproximadamente 5 segundos

### Window Scaling Con Optimización

Veamos la misma información pero con la optimización activa. 

![](/wp-content/uploads/2025/04/router1-tcp-opt-enabled.png)

Nota como el _window size_ estuvo en constante cambio a lo largo de la sesión, recuperándose rápida y agresivamente después de caer. 

Por qué es tan importante este parámetro? Le pedí a ChatGPT que lo explicará de una manera simple y concisa. 

> _La ampliación de ventana (window scaling) es crucial en redes con alta latencia o gran ancho de banda, ya que permite que TCP utilice una ventana de recepción más grande, lo cual impacta directamente en la cantidad de bytes in flight (datos no confirmados) que el emisor puede enviar. Sin esta opción, el tamaño máximo de la ventana es de 65,535 bytes —demasiado pequeño para enlaces de alta velocidad— lo que lleva a una infrautilización del enlace. Con window scaling, la ventana puede crecer hasta varios gigabytes, permitiendo al emisor mantener más datos "en vuelo" y sostener un alto rendimiento incluso con demoras_.

En resumen, la sesión TCP se divide en tres segmentos, donde los routers que optimizan anuncian una mayor ampliación de ventana (window scaling) y gestionan las conexiones con el cliente y el servidor. El tráfico ahora se rige por el algoritmo BBR para maximizar el rendimiento.

## Configuración
Para configurar esta funcionalidad, se pueden utilizar _Feature Templates_ o _Configuration Groups_ (con la versión 20.15 o superior). En mi caso utilizaré _Configuration Groups_ y voy a tener _Internal Service Nodes_ en ambos lados

Para empezar, añado la funcionalidad "App QoE" en el _Service Profile_ con la siguiente configuración:

![](/wp-content/uploads/2025/04/tcp-opt-config.png)

- **Service Node** para hacer aceleración
- **Forwarder** para actuar como Controller Node

Esta es la configuración que se enviará al equipo:

```
interface VirtualPortGroup2
  no shutdown
  ip address 192.168.2.1 255.255.255.0
  service-insertion appqoe
!
service-insertion appnav-controller-group appqoe ACG-APPQOE
  appnav-controller 192.168.2.1
!
service-insertion service-node-group appqoe SNG-APPQOE
  service-node 192.168.2.2
!
service-insertion service-context appqoe/1
  appnav-controller-group ACG-APPQOE
  service-node-group      SNG-APPQOE
  cluster-type            integrated-service-node
  enable
  vrf global
!
```

El estatus debe ser **"Running"**


```
Lisbon_10-1#show sdwan appqoe tcpopt status 
==========================================================
                  TCP-OPT Status
==========================================================

Status
------
TCP OPT Operational State      : RUNNING
TCP Proxy Operational State    : RUNNING
```

A continuación, creo una política de datos muy sencilla para hacer match del tráfico entre el cliente y el servidor y selecciono la acción de _AppQoE Optimization_ y selecciono la casilla de _TCP Optimization_.

![](/wp-content/uploads/2025/04/policy-config.png)

```
vsmart_1# show running-config policy 
policy
 data-policy _VPN_10_AppQoE
  vpn-list VPN_10
   sequence 1
    match
     source-data-prefix-list      BR10_172_16_10_0
     destination-data-prefix-list DC_100_172_16_100_0
    !
    action accept
     tcp-optimization
     service-node-group SNG-APPQOE
    !
   !
   sequence 11
    match
     source-data-prefix-list      DC_100_172_16_100_0
     destination-data-prefix-list BR10_172_16_10_0
    !
    action accept
     tcp-optimization
     service-node-group SNG-APPQOE
    !
   !
   default-action accept
  !
 !
 ```
**Nota** la dirección en la cual se debe aplicar la política es _ALL_

```
vsmart_1# show running-config apply-policy 
apply-policy
 site-list BR_10
  data-policy _VPN_10_AppQoE all
 !
 site-list DC_100
  data-policy _VPN_10_AppQoE all
 !
!
 ```
## Verificando la Optimización TCP

Para verificar que el tráfico está siendo optimizado, podemos habilitar On-Demand Troubleshooting y seleccionar un periodo de tiempo. 

![](/wp-content/uploads/2025/04/odt-tcp.png)

También, con la información en tiempo real podemos sacar la lista de flows que están siendo optimizados

![](/wp-content/uploads/2025/04/rt-tcp.png)

La columna de _Services_ indica que la Optimización TCP se está aplicando a esos flujos

 ## Probando el rendimiento de la Optimization TCP

Para evaluar el impacto de la Optimización TCP, ejecuté pruebas con iperf utilizando diferentes valores de latencia para observar en qué condiciones la función ofrece mayores beneficios. Aunque no se trata de un entorno de laboratorio profesional, proporciona información valiosa sobre cómo se comporta la optimización en la práctica.

**Nota**  Mi tráfico de iperf no está encriptado. No es posible optimizar tráfico encriptado sin antes desencriptarlo a través de TLS/SSL Decryption

Algunos detalles adicionales:

- El ancho de banda está topado a **250 Mbps** en los routers.
- Utilizo **4 flujos en paralelo**, cada uno simulando una descarga de **100 MB**:

> iperf -c 172.16.100.11 -n 100MB -P 4 -i 15 -R

Para mantener consistencia, corro cada prueba 5 veces, descarto el resultado más alto y más bajo y al final saco un promedio de los tres restantes.

La siguiente tabla muestra los resultados obtenidos:

| Delay | TCP Opt | BW (Mbps) | Time (s) |
|-------|---------|-----------|----------|
| 0     |Disabled | 248       |   ~ 13   |
| 0     |Enabled  | 121,6     |   ~ 27   |
| 50    |Disabled | 99,7      |   ~ 33   |
| 50    |Enabled  | 124       |   ~ 26   |
| 100   |Disabled | 71        |   ~ 46   |
| 100   |Enabled  | 131       |   ~ 25   |
| 150   |Disabled | 66        |   ~ 49   |
| 150   |Enabled  | 125       |   ~ 26   |
| 200   |Disabled | 59        |   ~ 56   |
| 200   |Enabled  | 131       |   ~ 25   |
| 250   |Disabled | 63        |   ~ 52   |
| 250   |Enabled  | 126       |   ~ 26   |

Aquí hay una representación visual de la misma información

![](/wp-content/uploads/2025/04/charts.png)

Lo que puedo concluir de los resultados:

1. Con un delay de 0 ms, la optimización reduce el rendimiento (121 Mbps frente a 248 Mbps), debido al procesamiento que introduce esta funcionalidad.

2. A medida que aumenta el delay, la optimización mejora el rendimiento y reduce el tiempo de transferencia, lo cual ya es evidente a partir de un delay de 50 ms.

3. El rendimiento disminuye significativamente sin optimización TCP. El ancho de banda baja de 248 Mbps a 0 ms a ~59–63 Mbps con retrasos de 200–250 ms. El tiempo también aumenta proporcionalmente.

4. El rendimiento se mantiene estable a través de diferentes valores de delay con optimización TCP. El rendimiento se mantiene alrededor de 125–131 Mbps incluso con retrasos altos. El tiempo de transferencia también es consistente, alrededor de ~26s.

## Conclusión

La optimización TCP es altamente efectiva para mitigar el impacto de la latencia en el rendimiento de TCP. Si bien introduce algo de sobrecarga en condiciones de baja latencia, sus beneficios se vuelven más evidentes a medida que aumenta el retraso. En escenarios con retrasos de 100 ms o más, la optimización puede ayudar a duplicar el rendimiento y reducir el tiempo de transferencia. Si estás pensando en habilitarla, ten en cuenta que, dependiendo del modelo de router, obtendrás rendimientos diferentes.

Además, esta función no debe habilitarse para todo el tráfico, sino que debe activarse para una aplicación específica o un conjunto de aplicaciones que necesiten aceleración. Finalmente, esta función ofrece mayores beneficios en líneas intercontinentales, transportes satelitales o enlaces de alta latencia similares.

¡Espero que esta publicación haya sido útil y nos vemos en la próxima!


