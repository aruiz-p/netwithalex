---
author: Alex
category:
  - sdwan
  - appqoe
draft: false
date: "2025-05-19T00:00:00+00:00"
title: "Serie AppQoe: Forward Error Correction (FEC)"
description: Aprende cómo funciona Forward Error Correction (FEC) en Cisco SD-WAN para mejorar el rendimiento de las aplicaciones en enlaces con pérdida de paquetes. Explora casos de uso, configuración y resultados de pruebas.
summary: Aprende cómo funciona Forward Error Correction (FEC) en Cisco SD-WAN para mejorar el rendimiento de las aplicaciones en enlaces con pérdida de paquetes. Explora casos de uso, configuración y resultados de pruebas.
url: /appqoe-fec/
tag:
  - Loss Correction
---
## Introducción

Ofrecer un rendimiento de aplicación consistente sobre enlaces congestionados o poco confiables es un desafío constante para la mayoría de las redes. Incluso con funciones avanzadas como [Enhanced Application-Aware Routing](/aar-mejorado/) u [Optimización TCP](/appqoe-opt-tcp/), hay condiciones en los enlaces que van más allá de lo que el failover, el balanceo de carga o la optimización pueden resolver.

Al agregar un mecanismo de recuperación a nivel de paquete, FEC permite que Cisco SD-WAN enmascare la pérdida de paquetes y mantenga el rendimiento de las aplicaciones sin depender de retransmisiones.

En este post, exploraremos qué tan efectivo puede ser FEC en un entorno SD-WAN, simulando condiciones con pérdida de paquetes y midiendo las tasas de recuperación.

Si estás evaluando FEC para tu despliegue o simplemente tienes curiosidad sobre cómo funciona, este artículo te llevará por la teoría y la práctica.

Vamos allá!

## Qué es Forward Error Correction (FEC)?

Forward Error Correction (FEC) es una técnica que mejora la confiabilidad de la transmisión de datos añadiendo información redundante a los paquetes antes de enviarlos por la red. En lugar de esperar retransmisiones cuando se pierden paquetes, el receptor utiliza esa redundancia para reconstruir los datos faltantes en tiempo real.

En la implementación de Cisco SD-WAN, FEC agrupa 4 paquetes de datos y agrega 1 _paquete de paridad_. Si **uno** de esos 4 paquetes se pierde en el camino, el receptor puede reconstruirlo utilizando el paquete de paridad mediante una operación XOR.

Veámoslo en un diagrama:

![](/wp-content/uploads/2025/05/fec1.png)

El remitente transmite la información al receptor, pero el paquete 3 se pierde en tránsito. El receptor puede usar el paquete de paridad para reconstruir el paquete 3 y así evitar retransmisiones y retrasos que afectarían la experiencia de las aplicaciones.

Es importante notar que si se pierden más de 1 paquete, incluyendo el de paridad, no es posible reconstruir la información. El tamaño del bloque es siempre de 4 paquetes de datos + 1 de paridad y no puede modificarse. Un bloque puede contener paquetes de diferentes flujos.

> **Nota** El hecho de que FEC agregue 1 paquete de paridad por cada bloque de 4 incrementa el consumo de ancho de banda.

Hay dos modos de operación: 
- **Always**: Se aplica FEC a todo el tráfico que haga match a la política, sin importar la cantidad de pérdida de paquetes en el transporte.
- **Adaptive**: Permite definir un _threshold_ de pérdida de paquetes antes de empezar a aplicar FEC. Por ejemplo, si hay 2% o más pérdida de paquetes, se debe aplicar FEC al tráfico. El porcentaje de pérdida de paquetes se saca con los [paquetes BFD](/simplificando-aar-1-3-las-bases/#bfd/). 

FEC es especialmente útil en aplicaciones en tiempo real como voz, video o sesiones interactivas, donde esperar retransmisiones provocaría retrasos severos.

Un punto importante es que FEC opera entre los dispositivos edge de SD-WAN, lo que lo hace completamente transparente para las aplicaciones: no es necesario modificar el comportamiento de clientes o servidores. Sin embargo, solo funciona cuando se utiliza encapsulación IPSec; **no está soportado sobre túneles GRE.**

Un detalle crítico de implementación es el tamaño de los paquetes: si los paquetes son demasiado grandes y terminan siendo fragmentados, la capacidad de FEC para reconstruirlos se reduce considerablemente. Para aprovechar al máximo FEC, asegúrate de que el _payload_ se mantenga por debajo del MTU del _path MTU_ para evitar la fragmentación.

## Configuración

Utilizando _Policy Groups_, podemos configurar FEC a través de política de datos que hagan match al tráfico y apliquen la acción _Loss Correction_

![](/wp-content/uploads/2025/05/fec2.png)

En mi caso, hice match a a todo el tráfico entre 172.16.10.0/24 y 172.16.100.0/24. Nota que se tienen los dos modos de operación: _Always y Adaptive_

Si se selecciona FEC Adaptive, el _threshold_ tiene que estar entre 1% y 5% pérdida de paquetes.

Está es la configuración completa de mi política:

```
vsmart_1# show running-config policy 
policy
 data-policy data_all_FEC
  vpn-list vpn_Corporate_Users
   sequence 1
    match
     source-ip      172.16.100.0/24
     destination-ip 172.16.10.0/24
    !
    action accept
     loss-protect fec-always
     loss-protection forward-error-correction always
    !
   !
   sequence 11
    match
     source-ip      172.16.10.0/24
     destination-ip 172.16.100.0/24
    !
    action accept
     loss-protect fec-always
     loss-protection forward-error-correction always
    !
   !
   default-action accept
  !
 !
 lists
  vpn-list vpn_Corporate_Users
   vpn 10
  !
  site-list site_10_100
   site-id 10
   site-id 100
  !
 !
 apply-policy
 site-list site_10_100
  data-policy data_all_FEC from-service
 !
!
!
```

## Verificación de FEC

No hay muchos comandos relacionados a FEC, pero podemos confirmar que FEC está operando con el siguiente comando:

```
Munich_DC100-1#show sdwan tunnel statistics fec 
tunnel stats ipsec 21.101.0.2 21.11.0.2 12346 12346
 fec-rx-data-pkts     16243
 fec-rx-parity-pkts   4075
 fec-tx-data-pkts     7
 fec-tx-parity-pkts   1
 fec-reconstruct-pkts 935
 fec-capable          true
 fec-dynamic          false
```
El _fec-reconstruct-pkts_ indica que se recuperaron 935 paquetes

Nota también que podemos fácilmente ver la cantidad de paquetes de paridad que se enviaron y recibieron siendo aproximadamente 1/4 del total de paquetes de datos enviados/recibidos. 

La misma información está también disponible a través de la interfaz gráfica del Manager, en la opción de _real time_

![](/wp-content/uploads/2025/05/fec3.png)

## Probando FEC

A bandwidth of 450k is **around** 5 VoIP calls and using a payload of 361 bytes. 

In this case, I am running unidirectional tests, but keep in mind FEC works in both directions.

Vamos a realizar algunas pruebas para ver FEC en acción y analizar la cantidad de pérdida de paquetes que puede recuperar. Mostraré distintos resultados para entender en qué condiciones FEC ofrece mejores beneficios.

**Nota** existe cierta pérdida de paquetes fuera de los routers SD-WAN que no puedo controlar. Por eso, para obtener resultados más precisos, primero tuve que encontrar una tasa de transmisión con la que obtuviera 0% de pérdida la **mayor parte del tiempo** en mis resultados con iperf3, y a partir de ahí comencé a introducir pérdida de manera controlada.

> iperf -c 172.16.100.11 -u -b 450k -t 30 -l 361 --dscp ef 

Un ancho de banda de 450 kbps equivale aproximadamente a 5 llamadas VoIP, utilizando un _payload_ de 361 bytes.

En este caso, estoy realizando pruebas unidireccionales, pero ten en cuenta que FEC funciona en ambos sentidos.

|% Pérdida|Total Paquetes enviados|Total Paquetes recibidos |Paquetes recuperados|% efectivo de pérdida|
|-----------------|------------------|----------------------|-----------------|---------------|
|1                |4693              |4639                  |54               |0              |
|2                |4694              |4588                  |96               |0,24           |
|3                |4693              |4558                  |111              |0,58           |
|4                |4694              |4524                  |147              |0,51           |
|5                |4693              |4448                  |195              |0,68           |
|6                |4693              |4401                  |231              |1,3            |
|7                |4693              |4374                  |238              |1,8            |
|8                |4693              |4331                  |283              |1,8            |
|9                |4693              |4292                  |297              |2,2            |
|10               |4693              |4215                  |304              |3,8            |
|12               |4693              |4122                  |348              |3,8            |
|15               |4695              |3941                  |382              |8              |
|18               |4696              |3815                  |356              |11             |
|20               |4696              |3731                  |368              |13             |

Veamos unas gráficas interesantes:

![](/wp-content/uploads/2025/05/fec4.png)
![](/wp-content/uploads/2025/05/fec5.png)
![](/wp-content/uploads/2025/05/fec6.png)

A medida que aumenta la pérdida de paquetes, también crece el número de paquetes recuperados, hasta cierto punto. Esto es esperable: FEC agrega redundancia, y cuanto más se pierde, más se necesita recuperar. Sin embargo, esta capacidad tiene un límite natural: si se pierden dos o más paquetes dentro del mismo bloque FEC —incluyendo el paquete de paridad— la recuperación ya no es posible y la pérdida efectiva comienza a aumentar.

También es importante destacar que FEC es una función **utiliza muchos recursos**, por lo que se recomienda activarla solo para tráfico crítico y, preferentemente, en combinación con un _threshold_ de pérdida de paquetes, en lugar de mantenerla activa permanentemente.

Aunque este laboratorio no replica a la perfección un entorno de producción, los resultados son bastante reveladores. FEC logró recuperar prácticamente todos los paquetes perdidos con hasta un 5% de pérdida introducida, y continuó recuperando cerca del 70% de los paquetes a aproximadamente 9% de pérdida. A partir de ahí, la eficiencia de recuperación empieza a disminuir. Dicho esto, no es común ver pérdidas constantes superiores al 10% en enlaces WAN de producción, y menos aún en ambas direcciones.

Por último, aunque las pruebas fueron unidireccionales, vale la pena mencionar que FEC puede aplicarse de forma independiente en cada dirección. Esto significa que, con una implementación bien ajustada, se podría tolerar cerca de un 5% de pérdida de paquetes por dirección sin afectar significativamente el rendimiento.

## Conclusion

Forward Error Correction (FEC) es una técnica proactiva que agrega redundancia antes de la transmisión de paquetes, permitiendo que el receptor recupere ciertas pérdidas sin necesidad de retransmisiones. Esto la hace especialmente valiosa para aplicaciones en tiempo real como voz o video, donde esperar retransmisiones generaría retrasos perjudiciales.

Recuerda que habilitar FEC viene con un costo: introduce carga adicional. El edge de SD-WAN que recibe los paquetes necesita usar capacidad de procesamiento adicional para reconstruir los que se perdieron. Por eso, es recomendable activarlo solo para tráfico crítico y, en lo posible, hacerlo condicionado a un _threshold_ de pérdida de paquetes.

FEC no reemplaza la necesidad de corregir enlaces de red defectuosos. Más bien, actúa como una capa de mitigación inteligente que ayuda a suavizar pérdidas moderadas o transitorias, manteniendo una experiencia de usuario consistente incluso cuando la red no es perfecta.

En resumen, cuando se implementa correctamente, FEC puede ser una herramienta muy poderosa en tu arquitectura SD-WAN, ayudando a garantizar un rendimiento de aplicaciones constante sobre redes imperfectas.

💭 ¿Qué opinas sobre el uso de Forward Error Correction en SD-WAN? ¿Lo has utilizado antes? ¿Tienes dudas sobre cómo funciona o cuándo activarlo?

Déjame tus comentarios, preguntas o experiencias. Me encantaría saber cómo están abordando esta función en sus despliegues reales.
¡Aprendamos entre todos!