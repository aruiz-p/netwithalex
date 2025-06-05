---
author: Alex
category:
  - sdwan
  - appqoe
draft: false
date: "2025-06-03T00:00:00+00:00"
title: "Serie AppQoe: Packet Duplication"
description: Descubre cómo packet duplication utiliza múltiples transportes para minimizar el riesgo de pérdida de paquetes sobre transportes inestables, manteniendo la calidad de las aplicaciones más importantes para cumplir con los objetivos del negocio.
summary: Descubre cómo packet duplication utiliza múltiples transportes para minimizar el riesgo de pérdida de paquetes sobre transportes inestables, manteniendo la calidad de las aplicaciones más importantes para cumplir con los objetivos del negocio.
url: /appqoe-pkt-dup-es/
tag:
  - Packet Duplication
---
## Introducción

En mi [post anterior](/appqoe-fec-es/) hablé sobre FEC, una técnica para combatir la pérdida de paquetes enviando un paquete de paridad que permite reconstruir los datos perdidos. En esta publicación, exploraremos otra potente funcionalidad de SD-WAN diseñada para enfrentar transportes con pérdida: Packet Duplication. Aunque ambas funciones buscan mejorar la confiabilidad, lo hacen de formas muy diferentes.

Vamos a verlo:

## ¿Qué es Packet Duplication?

Como su nombre indica, _Packet Duplication_ consiste en enviar paquetes idénticos a través de múltiples rutas de transporte para aumentar la probabilidad de entrega exitosa. Si una ruta experimenta pérdida transitoria o es inherentemente inestable, el paquete duplicado enviado por un segundo transporte más confiable puede compensarlo, manteniendo la experiencia del usuario sin interrupciones.

Visualmente:

![](/wp-content/uploads/2025/05/pkt-dup1.png)

El _emisor_ utiliza **ambos** transportes, bronze y gold, para enviar la información duplicada. La ruta bronze sufre pérdida de paquetes, mientras que la ruta gold entrega todos los paquetes con éxito. El receptor descarta los duplicados y reenvía solo la copia que llega primero al destino, eliminando efectivamente la pérdida desde la perspectiva del usuario, sin importar qué ruta tomó el paquete.

Antes de duplicar un paquete, el _WAN Edge_ compara su tamaño con el _Path Maximum Transmission Unit (PMUT)_, si la longitud del paquete es menor que el PMUT, este se duplica.

Nota: A partir de la versión 17.15.1a, los paquetes pueden duplicarse incluso si su tamaño es mayor al PMUT, utilizando [VRF and Underlay Fragmentation](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/ios-xe-17/systems-interfaces-book-xe-sdwan/configure-interfaces.html#g-fht-vfr-underlay-fragmentation).

Desde la versión 16.12.1b esta funcionalidad aplica para:
- Tráfico IPv4 sobre túneles IPv4.

Y a partir de la 17.15.1a se amplía el soporte a:
- Tráfico IPv4 sobre túneles IPv6
- Tráfico IPv6 sobre túneles IPv4
- Tráfico IPv6 sobre túneles IPv6

## Consideraciones

Aquí algunos puntos clave a tener en cuenta al habilitar Packet Duplication:

- Para que funcione, debe haber al menos dos transportes disponibles; no es una solución viable para sitios con un solo transporte.
- La cantidad de tráfico se duplica, tenlo en cuenta según la capacidad de tus transportes.
- El receptor necesita almacenar en búfer y descartar paquetes, lo que añade carga de procesamiento.
- Debe habilitarse solo para tráfico crítico, considerando los puntos anteriores.

## Configuración

Utilizando Policy Groups, podemos configurar Packet Duplication mediante una data policy que haga match con el tráfico deseado y aplique la acción _Loss Correction_ seleccionando _Packet Duplication_.

![](/wp-content/uploads/2025/05/pkt-dup2.png)

Esta configuración debe aplicarse en ambas direcciones, por lo que la política completa se ve así:

```
 data-policy data_service_Packet-Duplication
  vpn-list vpn_Corporate_Users
   sequence 1
    match
     source-ip      172.16.100.0/24
     destination-ip 172.16.10.0/24
    !
    action accept
     loss-protect pkt-dup
     loss-protection packet-duplication
    !
   !
   sequence 11
    match
     source-ip      172.16.10.0/24
     destination-ip 172.16.100.0/24
    !
    action accept
     loss-protect pkt-dup
     loss-protection packet-duplication
    !
   !
   default-action accept
  !
 !
!
apply-policy
 site-list site_10_100
  data-policy data_service_Packet-Duplication from-service
 !
!
```

**Nota**: Packet Duplication no es compatible con las siguientes acciones dentro de una data policy:
- local tloc
- remote_tloc

Puedes consultar la [guía de configuración](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/packet-duplication.html) para más información sobre restricciones y casos de uso.

## Verificación

Para asegurar que Packet Duplication está funcionando, hay algunas validaciones que podemos hacer.

Desde el dispositivo se puede ejecutar:

```
Lisbon_10-1#show sdwan tunnel statistics pkt-dup 
Generating output, this might take time, please wait ...
tunnel stats gre 21.11.0.2 21.101.0.2 0 0
 pktdup-rx       4844
 pktdup-rx-other 1
 pktdup-rx-this  4844
 pktdup-tx       8710
 pktdup-tx-other 4693
 pktdup-capable  true
tunnel stats gre 31.11.0.2 31.101.0.2 0 0
 pktdup-rx       1
 pktdup-rx-other 4844
 pktdup-rx-this  1
 pktdup-tx       4693
 pktdup-tx-other 8710
 pktdup-capable  true
 ```

- pktdup-rx: paquetes originales recibidos por el túnel primario
- pktdup-rx-other: paquetes duplicados recibidos por el segundo túnel
- pktdup-tx: paquetes duplicados enviados por el túnel primario
- pktdup-tx-other: paquetes duplicados enviados por el túnel secundario
- pktdup-capable: indica si se realizó intercambio de capacidades con el otro dispositivo

Esta misma información está disponible en la sección de _Real Time_ del Manager:

![](/wp-content/uploads/2025/05/pkt-dup3.png)

Además, se puede añadir un contador a la política para confirmar que la secuencia está siendo aplicada.

## Probando Packet Duplication

En mi laboratorio tengo dos túneles activos entre los dispositivos SD-WAN: uno sobre MPLS y otro sobre Biz-Internet.

Para probar Packet Duplication, introduciré pérdida de paquetes en el transporte MPLS y ejecutaré una prueba básica de iperf entre dos clientes para observar cómo se comporta.

El comando de prueba utilizado fue:

>iperf3 -c 172.16.100.11 -u -b 1M -t 20 --dscp ef

Este comando inicia una prueba UDP de 20 segundos, con un ancho de banda de 1 Mbps y marcado DSCP EF.

**Nota**: No me enfoco en el porcentaje exacto de pérdida en MPLS. La idea no es medir cuánto puede recuperar Packet Duplication, sino observar cómo actúa la funcionalidad. En teoría, incluso con una pérdida del 100% en un transporte, el tráfico debería llegar por el otro, siempre y cuando Packet Duplication esté correctamente configurado.

El _emisor_ utiliza biz-internet como transporte primario:

```
Lisbon_10-1#show sdwan policy service-path vpn 10 interface gigabitEthernet 2 source-ip 172.16.10.11 dest-ip 172.16.100.11 protocol 17 dscp 46 all            
Number of possible next hops: 1
Next Hop: GRE
  Source: 31.11.0.2 Destination: 31.101.0.2 Local Color: biz-internet Remote Color: biz-internet Remote System IP: 1.1.100.1
```

MPLS tiene un 80% de pérdida y los resultados de iperf fueron los siguientes:

```
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-20.00  sec  2.38 MBytes  1.00 Mbits/sec  0.000 ms  0/4188 (0%)  emisor
[  5]   0.00-20.04  sec  2.38 MBytes   998 Kbits/sec  0.255 ms  0/4188 (0.0%)  receiver
```

Vamos a revisar las estadísticas de Packet Duplication tanto en el emisor como en el receptor:

Emisor
```
Lisbon_10-1#show sdwan tunnel statistics pkt-dup
Generating output, this might take time, please wait ...
tunnel stats gre 21.11.0.2 21.101.0.2 0 0
 pktdup-rx       14
 pktdup-rx-other 0
 pktdup-rx-this  14
 pktdup-tx       0
 pktdup-tx-other 4206
 pktdup-capable  true
tunnel stats gre 31.11.0.2 31.101.0.2 0 0
 pktdup-rx       0
 pktdup-rx-other 14
 pktdup-rx-this  0
 pktdup-tx       4206
 pktdup-tx-other 0
 pktdup-capable  true
 ```
Observa lo siguiente:

|Rol   | Paquetes enviados primario| Paquetes duplicados secundario|
|-------|-------------------------|--------------------------|
|Emisor | 4206                    | 4206                     |

Se transmitió exactamente la misma cantidad de paquetes por los transportes MPLS y Biz-Internet.

Receptor
```
Munich_DC100-1#show sdwan tunnel statistics pkt-dup
Generating output, this might take time, please wait ...
tunnel stats gre 21.101.0.2 21.11.0.2 0 0
 pktdup-rx       0
 pktdup-rx-other 758
 pktdup-rx-this  0
 pktdup-tx       14
 pktdup-tx-other 0
 pktdup-capable  true
tunnel stats gre 31.101.0.2 31.11.0.2 0 0
 pktdup-rx       4206
 pktdup-rx-other 0
 pktdup-rx-this  758
 pktdup-tx       0
 pktdup-tx-other 14
 pktdup-capable  true
```

|Rol      |Paquetes recibidos primario | Paquetes recibidos secundario|
|---------|----------------------|------------------------|
|Receptor | 4206                 | 758                    |

Como el transporte secundario experimentaba una pérdida severa, solo el 18% de los paquetes fueron recibidos por él. Sin embargo, esto no representó un problema ya que el transporte primario entregó el 100% de los paquetes.

Si el escenario se invirtiera, es decir, si la ruta primaria sufriera pérdida y la secundaria permaneciera estable, los roles simplemente se intercambiarían. El primario entregaría el 18% y el secundario el 100%, y nuevamente, el usuario final no experimentaría ninguna pérdida de paquetes.

## Conclusión

Packet Duplication es una funcionalidad simple pero poderosa en Cisco SD-WAN que mejora significativamente la confiabilidad del tráfico. Al enviar paquetes idénticos por múltiples rutas de transporte, garantiza que, incluso si una ruta falla, el rendimiento de la aplicación no se vea afectado.

Aunque conlleva un mayor uso de ancho de banda y procesamiento adicional, puede marcar una gran diferencia para tráfico crítico como voz, video o sesiones remotas, especialmente en entornos con circuitos poco confiables.

Como con cualquier funcionalidad de SD-WAN, la clave es usarla de manera **estratégica**: activarla para el tráfico correcto, monitorear los resultados y ajustar las políticas según el comportamiento de tu red. ¿Has utilizado Packet Duplication en producción? Cuéntame tu experiencia en los comentarios o conéctate conmigo en [LinkedIn](https://www.linkedin.com/in/alexruizs/) para conversar sobre temas avanzados de SD-WAN.

