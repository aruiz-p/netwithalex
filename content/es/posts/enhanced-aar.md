---
author: Alex
category:
  - sdwan
date: "2025-01-05T14:20:31+00:00"
title: Enhanced App Aware Routing
summary: Aprende los beneficios de EAAR y cómo implementarlo en tu red
url: /mejorado-aar/
tag:
  - eaar
  - sdwan
---
## Introducción

En mi anterior [serie de publicaciones](/simplificando-aar-1-3-las-bases), exploré **Application Aware Routing (AAR)** en profundidad, una tecnología clave en SD-WAN que dirige el tráfico sobre los transportes con mejor rendimiento. Si bien AAR ha sido una capacidad fundamental durante años, la evolución de las redes trajo nuevas ideas para mejorar su efectividad. Esto condujo a la introducción de **_Enhanced Application Aware Routing (EAAR)_**. 

## Limitaciones de AAR

Antes de sumergirnos en EAAR, entendamos por qué fue creado. 🤓

La implementación actual de AAR mide la calidad del transporte utilizando BFD, enviando probes a un intervalo definido (1s por defecto). La pérdida, la latencia y jitter se derivan de esos paquetes y estos valores se colocan en cubetas que van rotando para calcular una métrica promedio que representa la salud del túnel. Este proceso generalmente toma entre 10 y 60 minutos y con algunos ajustes de configuración es posible lograr tiempos de 2 a 10 minutos.

Para aquellas redes que requieren una detección más rápida, surgen algunos desafíos:
- Bajar el intervalo de _hello_ a menos de 1 segundo afecta la escala de túneles del dispositivo
- Bajar el multiplicador de BFD y el _poll interval_ podrían conducir a falsos positivos, moviendo el tráfico incluso con condiciones de red transitorias.
- Cambio constante del tráfico de un lado a otro, no hay un mecanismo para determinar si un transporte es estable nuevamente después de un evento de degradación de la red. 

## Enhanced AAR 

_Entonces, ¿qué es EAAR y cómo mejora a su predecesor?_ 🤔

En pocas palabras, estas son las ventajas:
- Utiliza **inline data** en lugar de BFD. En otras palabras, los paquetes de plano de datos se utilizan para medir la pérdida, la latencia y el jitter.
- Capacidad de mover el tráfico en **segundos** en lugar de minutos
- Amortiguación (_Dampening_) implementada para propósitos de **estabilidad**, asegurando que los transportes sean estables antes de reenviar el tráfico a través de ellos. 
- **Medición más precisa** de pérdida, latencia y jitter 

### Vamos por partes

Cuando EAAR está habilitado, los paquetes de datos se utilizarán para medir la pérdida, la latencia y la jitter. Entendamos las diferencias clave: 

#### Medición de pérdida 

Los routeres SD-WAN utilizarán _inline data_ junto con los números de secuencia IPSEC para medir la pérdida. 

Hay un mecanismo incorporado que permite a los routers determinar si la pérdida es local para el router 

- **Pérdida local** - Por lo general, debido a caídas de QoS 

O externo al router 
- **Pérdida de WAN** - Cualquier pérdida de paquetes fuera del router 

Para calcular la pérdida local, el router determinará la cantidad de paquetes que generó en comparación con la cantidad de paquetes que realmente fueron enviados. Para obtener la pérdida en el WAN, los routers SD-WAN vecinos informarán la cantidad de paquetes recibidos y utilizarán BFD (TLVs de Monitoreo de Ruta) para enviar esta información de vuelta al router original.

Hasta este punto, existe una mejora importante en cómo se realiza la medición de pérdidas, sin embargo, esto podría mejorarse aún más al aprovechar mediciones _per queue_ de QoS. Para lograr esto, necesitamos asociar una clase de SLA con una [App Probe Class](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-xebook-xe/application-aware-routing.html#concept_gjc_5pm). Veamos un ejemplo. 

![](/wp-content/uploads/2025/02/app-probe.png)

Con esta App Probe Class, el router utilizará (y generará paquetes de BFD) con DSCP 18, imitando el tráfico menos importante que estará sujeto a diferentes reglas y rutas en los routers locales y externos. Esto proporcionará una medición más precisa de la pérdida de paquetes para cada tipo de tráfico en los transportes especificados. Si no hay _inline data_, BFD se usa para obtener mediciones.

**Nota** Si usas GRE, la medición _per queue_ **no**  está disponible. 

Aquí hay un visual para comprender mejor cómo se medirán la pérdida dependiendo de múltiples factores.

|Encapsulación  | App Probe Class |Tipo de medición | Túneles públicos                       | Túneles privados                           |
|---------------|-----------------|-----------------|---------------------------------------|-------------------------------------------|
| IPsec         | Sí             | Por SLA         | total pérdida WAN + pérdida local por queue | pérdida WAN per queue + pérdida local por queue |
| IPsec         | No              | Todos los SLAs        | total pérdida WAN + total pérdida local      | total pérdida WAN + total pérdida local          |
| GRE           | -               | Todos los SLAs        | total pérdida WAN + total pérdida local      | total pérdida WAN + total pérdida local          |

#### Latencia 

Para medir la latencia, el router simplemente calculará el tiempo necesario para enviar y recibir paquetes entre los dispositivos de origen y de destino. Se utiliza _inline data_ y puede llegar a la granularidad de App Probe Class.

#### Jitter

Vale la pena mencionar que el jitter se calcula por dirección (recibir o transmitir). El jitter se calcula en el receptor e informa al remitente usando TLV BFD. Se utiliza _inline data_ y, en caso de no haber tráfico de datos, se usa BFD. 

### SLA Dampening
Uno de los beneficios de EAAR es dirigir el tráfico en segundos en lugar de minutos, pero ¿qué pasaría si hay condiciones de red transitorias que hacen que los transportes no cumplan con el SLA cada pocos minutos? El tráfico cambiaría constantemente entre los transportes, lo que no es un escenario deseable y la razón por la cual se introdujo el _dampening_. 

La idea general es que cuando un enlace de transporte deja de cumplir con el SLA, el tráfico se redirige a una ruta alternativa. Una vez que el transporte vuelve a estar en cumplimiento, el dispositivo no mueve el tráfico de inmediato. En su lugar, inicia un temporizador para asegurarse de que el enlace permanezca estable durante un período determinado antes de reutilizarlo.

Al final, el dampening ayuda a evitar cambios innecesarios en el tráfico, lo que podría afectar negativamente el rendimiento debido a la inestabilidad del transporte.

## Configuración de EAAR

Para habilitar EAAR tenemos tres opciones predefinidas:

| Mode         | Poll Interval | Poll Multiplier         | Dampening Multiplier |
|--------------|---------------|-------------------------|----------------------|
| Aggressive   | 10s           | 6 (10s-60s)             | 120 (20 mins)        |
| Moderate     | 60s           | 5 (60s-300s)            | 40  (40 mins)        |
| Conservative | 300s          | 6 (300s-1800s)          | 12  (60 mins)        |

**Nota** Para usar tiempos personalizados, la configuración debe hacerse a través de plantillas CLI.

EAAR sigue el mismo principio fundamental que AAR, utilizando buckets que van rotando para calcular la pérdida promedio, la latencia y la jitter. Con el modo _aggressive_, el tráfico tomaría entre 10 y 60 segundos en cambiar, dependiendo de qué tan severo sea el deterioro. 

El _Dampening Multiplier_ (_poll interval_ **x** _Dampening Multiplier_) es de 1200 segundos, lo que significa que antes de cambiar el tráfico a un transporte, debe ser estable durante 20 minutos. 

En mi laboratorio, estoy usando _Configuration Groups_, sin embargo, esto está [disponible a través de Templates](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/m-enhanced-application-aware-routing.html#configure-enhanced-application-aware-routing-using-a-feature-template-in-cisco-sd-wan-manager) también. 

![](/wp-content/uploads/2025/02/eaar-config-groups.png)

Puedes usar una variable, en lugar de un valor global, para que sea aplicable también a los dispositivos que no correrán EAAR. En este caso, los dispositivos habilitados para EAAR harán _fallback_ a AAR. 

La siguiente configuración se agrega a los dispositivos:

```
  bfd enhanced-app-route enable
  bfd enhanced-app-route pfr-poll-interval 10000
  bfd enhanced-app-route pfr-multiplier 6
  bfd sla-dampening enable
  bfd sla-dampening multiplier 120
```
Veamos cómo funciona

## Demo

En mi laboratorio, uso el Manager con versión 20.16.1 y mis dispositivos con 17.15.1a

**Nota** La versión mínima requerida es 20.12/17.12 

Comencemos con algunas verificaciones después de aplicar la configuración. 

Para verificar los temporizadores y multiplicadores configurados

```
BR10#show sdwan app-route params
Enhanced Application-Aware routing
    Config:                  :Enabled   
    Poll interval:           :10000     
    Poll multiplier:         :6         

App route 
    Poll interval:           :120000    
    Poll multiplier:         :5         

SLA dampening  
    Config:                  :Enabled   
    Multiplier:              :120    
```

Para verificar qué sesiones de BFD están utilizando EAAR, busca la columna _Flags_
```
BR10#show sdwan bfd sessions alt

                                  SOURCE TLOC      REMOTE TLOC                     DST PUBLIC    DST PUBLIC                                  
SYSTEM IP   SITE ID   STATE       COLOR            COLOR            SOURCE IP      IP            PORT        ENCAP  BFD-LD    FLAGS   UPTIME    
-------------------------------------------------------------------------------------------------------------------------------------------------
1.1.1.20    200       up          biz-internet     biz-internet     30.1.10.2      30.1.20.2     12406       ipsec  20006     EAAR    0:00:20:31 
1.1.1.20    200       up          mpls             mpls             30.2.10.2      30.2.20.2     12366       ipsec  20002     EAAR    0:00:20:38 
1.1.1.20    200       up          private1         private1         30.3.10.2      30.3.20.2     12366       ipsec  20003     EAAR    0:00:20:37 
``` 

Para obtener más detalles sobre un túnel específico. 

```
BR10#show sdwan app-route stats summary 
Generating output, this might take time, please wait ...
app-route statistics 30.1.10.2 30.1.20.2 ipsec 12386 12406
 remote-system-ip         1.1.1.20
 local-color              biz-internet
 remote-color             biz-internet
 sla-class-index          0,1,2
 fallback-sla-class-index None
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      0             0             0        0        0             0             0             0             
1      64            0             0        0        0             0             0             0             
2      0             0             0        0        0             0             0             0             
3      0             0             0        0        0             0             0             0             
4      0             0             0        0        0             0             0             0             
5      64            0             0        0        0             0             0             0             
```

Nota que no hay tráfico entre mis dispositivos, por lo que el recuento total de paquetes en cada _bucket_ es bajo.

La misma información está disponible a través de la interfaz gráfica, utilizando la opción de _Real Time_ > _App Route Statistics_

![](/wp-content/uploads/2025/02/eaar-1.png)

### Escenario 1 - Deterioro ligero

Esta es la topología de mi laboratorio

![](/wp-content/uploads/2024/02/aar-topo2.png)

Para esta primera prueba, usaré los siguientes parámetros de SLA:

```
SLA_Real-Time 
Loss: 3%
Latency: 150ms
Jitter: 100ms
```
La política AAR indica que: 

- Use MPLS como transporte principal 
- Si ningún color cumple con el SLA y Private1 está disponible, usarlo. 
- Si Private1 no está disponible, se hace _load balance_ entre todos los colores restantes. 

Estoy haciendo match del tráfico entre 172.16.10.0/24 y 172.16.20.0/24. 
```
BR10#show sdwan policy from-vsmart
from-vsmart app-route-policy app_route_AAR
 vpn-list vpn_Corporate_Users
  sequence 1
   match
    source-data-prefix-list      BR10
    destination-data-prefix-list BR20
   action
    backup-sla-preferred-color private1
    sla-class       SLA_Real-Time
    no sla-class strict
    sla-class preferred-color mpls
  sequence 11
   match
    source-data-prefix-list      BR20
    destination-data-prefix-list BR10
   action
    backup-sla-preferred-color private1
    sla-class       SLA_Real-Time
    no sla-class strict
    sla-class preferred-color mpls
```

Estado inicial sin deterioro en la red 

```
BR10#show sdwan app-route stats summary | i color|damp|mean         
 local-color              biz-internet
 remote-color             biz-internet
 sla-dampening-index      None
  mean-loss    0.000
  mean-latency 1
  mean-jitter  0
 local-color              mpls
 remote-color             mpls
 sla-dampening-index      None
  mean-loss    1.212
  mean-latency 1
  mean-jitter  0
  mean-loss    1.212
  mean-latency 1
  mean-jitter  0
 local-color              private1
 remote-color             private1
 sla-dampening-index      None
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
```
Observa que el número de paquetes por bucket aumentó dramáticamente

```
BR10#show sdwan app-route stats remote-color mpls summary  
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0,1
 fallback-sla-class-index None
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    1.176
  mean-latency 0
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      131136        1501          0        0        100084        20846         0             0             
1      131072        1438          0        0        95287         19605         0             0             
2      131072        1400          0        0        100985        20937         0             0             
3      131072        1781          0        0        85553         18271         0             0             
4      64            0             0        0        72618         15942         0             0             
5      131072        1589          0        0        65198         14226         0             0             
          
```

El tráfico está utilizando MPLS como transporte primario

```
BR10# show sdwan policy service-path vpn 10 interface gigabitEthernet 4 source-ip 172.16.10.10 dest-ip 172.16.20.10 protocol 6 all       
Number of possible next hops: 1
Next Hop: IPsec
  Source: 30.2.10.2 12366 Destination: 30.2.20.2 12366 Local Color: mpls Remote Color: mpls Remote System IP: 1.1.1.20
```

Introduzco el 3% de pérdida de paquetes en el transporte MPLS y veré cuánto tiempo lleva cambiar el tráfico. Dado que ya hay alrededor del 1% de pérdida, el 3% debería ser suficiente para activar un cambio. 

El transporte MPLS tiene más del 3% de pérdida

```
BR10#show sdwan app-route stats remote-color mpls summary 
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    3.125          <<<<<<<<<<
  mean-latency 0
  mean-jitter  0
```

Después de 56 segundos, el tráfico cambió y cualquiera de los transportes que cumplen el SLA podría usarse 

```
BR10# show sdwan policy service-path vpn 10 interface gigabitEthernet 4 source-ip 172.16.10.10 dest-ip 172.16.20.10 protocol 6 all
Number of possible next hops: 2
Next Hop: IPsec
  Source: 30.3.10.2 12366 Destination: 30.3.20.2 12366 Local Color: private1 Remote Color: private1 Remote System IP: 1.1.1.20
Next Hop: IPsec
  Source: 30.1.10.2 12386 Destination: 30.1.20.2 12366 Local Color: biz-internet Remote Color: biz-internet Remote System IP: 1.1.1.20

```

Si quito la pérdida, podemos ver que el mecanismo de _dampening_ se activa. Entonces, si el transporte es estable durante 20 minutos, se volverá a utilizar como ruta preferida. 

```
BR10#show sdwan app-route stats remote-color mpls summary 
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      1        <<<<<<<<<<
 app-probe-class-list None
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
```

### Escenario 2 - Mayor deterioro

En este caso, introduciré una pérdida de paquetes del 10% para el transporte de biz-internet, lo que hace que private1 sea el único transporte cumpliendo el SLA. 

Después de alrededor de 45 segundos, la pérdida de Biz-Internet fue de 12%

```
BR10#show sdwan app-route stats local-color biz-internet summary         
Generating output, this might take time, please wait ...
app-route statistics 30.1.10.2 30.1.20.2 ipsec 12386 12366
 remote-system-ip         1.1.1.20
 local-color              biz-internet
 remote-color             biz-internet
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    12.500
  mean-latency 0
  mean-jitter  0
```

El tráfico cambió a private1 exclusivamente

```
BR10#show sdwan policy service-path vpn 10 interface gigabitEthernet 4 source-ip 172.16.10.10 dest-ip 172.16.20.10 protocol 6 all
Number of possible next hops: 1
Next Hop: IPsec
  Source: 30.3.10.2 12366 Destination: 30.3.20.2 12366 Local Color: private1 Remote Color: private1 Remote System IP: 1.1.1.20
```

Después de eliminar la pérdida de paquetes, Biz-Internet tiene el mecanismo de dampening activado

```
BR10#show sdwan app-route stats local-color biz-internet summary 
Generating output, this might take time, please wait ...
app-route statistics 30.1.10.2 30.1.20.2 ipsec 12386 12366
 remote-system-ip         1.1.1.20
 local-color              biz-internet
 remote-color             biz-internet
 sla-class-index          0
 fallback-sla-class-index 1
 enhanced-app-route       Enabled
 sla-dampening-index      1         <<<<<<<<<
 app-probe-class-list None
  mean-loss    1.562
  mean-latency 0
  mean-jitter  0
```
En este caso, el tiempo para cambiar el tráfico se redujo como consecuencia de un deterioro mayor. 

### Escenario 3 -  Múltiples App Probe Class

Para este escenario final, veamos cómo obtener el mayor beneficio de EAAR. 

La configuración es más compleja ya que involucra QoS, App Probe Class y política AAR.

- Se requiere QoS para clasificar y enviar tráfico en diferentes colas.
- App Probe Class para medir la pérdida, la latencia y el jitter en cada una de esas colas, de forma independiente.

Mi configuración de QoS tiene 3 colas y la cola 2 manejará el tráfico menos importante. 

- cola 0 para el tráfico de control
- cola 1 para tráfico en tiempo real (marcado DSCP 46)
- cola 2 para tráfico transaccional (marcado DSCP 18)

Utilizo una política de datos para hacer match del tráfico en el lado del servicio, marcarlo con el DSCP correcto y ponerla en la clase de QoS correcta. También creé un shaper en mi interfaz MPLS.

Para demostrar las cosas, tendré dos transferencias de datos:
- HTTP GET (puerto 8000)
- Copia SCP (puerto 22) 

Mis SLA tienen las siguientes configuraciones:

|SLA Class Name     | Loss |Latency|Jitter  |
|-------------------|-----|--------|--------|
| SLA_Real-Time     | 3 % | 150 ms | 100 ms |
| SLA_Transactional | 5 % | 45 ms  | 150 ms | 


Lo primero que hay que notar es que mis dos clases de sondeo de aplicaciones se miden de manera independiente. Observa cómo la **_pérdida media_** para _Transactional-Probe_Class_ es **1**, mientras que para _Real_Time_Probe_Class_ es **0**.

```
BR10#show sdwan app-route stats local-color mpls summary 
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12366 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0,1,2
 fallback-sla-class-index None
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    0.000
  mean-latency 1
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      16384         0             0        0        35827         111807        0             0             
1      49152         0             1        0        37788         118236        0             0             
2      49216         0             1        0        36859         115551        0             0             
3      16384         0             1        0        23894         77280         0             0             
4      32768         0             1        1        33179         103759        0             0             
5      32768         0             1        0        21485         71702         0             0             

 app-probe-class-list Real_Time_Probe_Class
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      0             0             0        0        -             -             -             -             
1      32768         0             1        0        -             -             -             -             
2      32768         0             0        0        -             -             -             -             
3      0             0             0        0        -             -             -             -             
4      32768         0             1        2        -             -             -             -             
5      0             0             0        0        -             -             -             -             

 app-probe-class-list Transactional-Probe_Class
  mean-loss    0.000
  mean-latency 1
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      16384         0             0        0        -             -             -             -             
1      49152         0             1        0        -             -             -             -             
2      49216         0             1        0        -             -             -             -             
3      16384         0             1        0        -             -             -             -             
4      32768         0             1        1        -             -             -             -             
5      32768         0             1        0        -             -             -             -             

```

Ahora, he bajado mi shaper. EAAR fue rápido en detectar un cambio en la latencia para el SLA _Transactional_, ahora son 53 ms. 

```
BR10# show sdwan app-route stats local mpls summary
Generating output, this might take time, please wait ...
app-route statistics 30.2.10.2 30.2.20.2 ipsec 12386 12366
 remote-system-ip         1.1.1.20
 local-color              mpls
 remote-color             mpls
 sla-class-index          0,1,2
 fallback-sla-class-index None
 enhanced-app-route       Enabled
 sla-dampening-index      None
 app-probe-class-list None
  mean-loss    0.000
  mean-latency 53
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      2048          0             54       0        4396          8513          0             0             
1      1024          0             55       0        4461          8534          0             0             
2      8256          0             49       0        4429          8549          0             0             
3      16384         0             54       0        4411          8549          0             0             
4      0             0             53       0        4440          8549          0             0             
5      0             0             54       0        4443          8549          0             0             

 app-probe-class-list Real_Time_Probe_Class
  mean-loss    0.000
  mean-latency 0
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      0             0             0        0        -             -             -             -             
1      0             0             0        0        -             -             -             -             
2      8192          0             0        0        -             -             -             -             
3      16384         0             0        0        -             -             -             -             
4      0             0             0        0        -             -             -             -             
5      0             0             0        0        -             -             -             -             

 app-probe-class-list Transactional-Probe_Class
  mean-loss    0.000
  mean-latency 53    <<<<<<<
  mean-jitter  0
       TOTAL                       AVERAGE  AVERAGE  TX DATA       RX DATA       IPV6 TX       IPV6 RX       
INDEX  PACKETS       LOSS          LATENCY  JITTER   PKTS          PKTS          DATA PKTS     DATA PKTS     
-------------------------------------------------------------------------------------------------------------
0      2048          0             54       0        -             -             -             -             
1      1024          0             55       0        -             -             -             -             
2      8256          0             49       0        -             -             -             -             
3      16384         0             54       0        -             -             -             -             
4      0             0             53       0        -             -             -             -             
5      0             0             54       0        -             -             -             -             
```

Ahora que no se cumple mi SLA _Transactional_ con una latencia máxima de 45 ms, usaré [NWPI](/Network-Wide-Path-Insights-An-Introduction/) para comprender cómo se envía el tráfico. Examinemos el tráfico HTTP con el puerto 80000

![](/wp-content/uploads/2025/02/eaar-nwpi-8000.png)

Observa que, en la dirección _upstream_, el color _local_ y _remote_ es **private1**, lo que indica que el tráfico se ha alejado de MPLS y su latencia de 53 ms. Justo lo que esperábamos ✅

Ahora, veamos cómo está fluyendo el tráfico en el puerto 22 

![](/wp-content/uploads/2025/02/eaar-nwpi-22.png)

Una vez más, observa el _local_ y _Remote Color_ en la dirección _upstream_, observa cómo **MPLS** todavía está en uso para este tráfico, ya que no hay problemas en los transportes para este tipo de tráfico. 

En resumen, el tráfico con DSCP 46 funciona perfectamente bien en el transporte MPLS, sin embargo, el tráfico con DSCP 18 estaba teniendo más latencia que el SLA configurado, por lo que se trasladó a Private1 ya que cumple con el SLA. 

Podemos confirmar que estamos midiendo y tomando decisiones de ruteo _per queue_, esta es una gran diferencia 🤯!

## Lecciones aprendidas 

- Utilizando _inline data_, el número de muestras aumenta dramáticamente en comparación con el tamaño de muestra de BFD. 📈
- EAAR puede redirigir el tráfico en segundos, en lugar de minutos. ⏩
- EAAR ofrece los mayores beneficios en los transportes con QoS, como MPLS. 🚀
- Incluso en los transportes sin QoS, las mediciones de _inline data_ aumentan el tamaño y la precisión de las muestras. ⏳
- El temporizador de dampening es útil para garantizar que los transportes sean estables antes de marcarlos como válidos. ✅
- La interoperabilidad entre dispositivos que ejecutan EAAR y dispositivos que ejecutan AAR es posible 🔄

¡Espero que hayas aprendido algo útil! Nos vemos en el siguiente 👋