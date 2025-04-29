---
author: Alex
category:
  - AAR
  - Políticas
  - SD-WAN
date: "2024-02-11T12:11:06+00:00"
title: 'Simplificando AAR: 3/3 Fallback to Best Path'
description: Aprende cómo AAR puede ayudarte a mejorar la experiencia del usuario y aplicación con Cisco SD-WAN 
summary: Aprende cómo AAR puede ayudarte a mejorar la experiencia del usuario y aplicación con Cisco SD-WAN 
url: /simplificando-aar-3-3-fallback-to-best-path/
---
## Introducción

Para la publicación final de esta serie, exploremos la opción restante para manejar el tráfico cuando no se cumple SLA: _Fallback a Best Path_. Se introdujo en 20.5/17.5 y proporciona más flexibilidad y selección de ruta mejorada en comparación con las otras opciones. Entendamos por qué fue creado.

## Motivación

Con los [métodos anteriores] (/desmitificación-AAR-Enderstanding-Diferent-SceneRios/), el tráfico lo haría:

- se eliminará, rara vez se usan casos de uso específicos que se aplican a una pequeña cantidad de entornos.
- Esté equilibrado en las rutas disponibles, ampliamente utilizada, sin embargo, el tráfico podría estar utilizando la ruta de peor desempeño.

Tome el siguiente ejemplo

![](/wp-content/uploads/2024/02/s3-aar.png)

Tunnel 2 claramente tiene el peor rendimiento, sin embargo, con el método _load balance_, el tráfico aún podría usarlo en función del algoritmo de hash. ¿Cómo superamos esta situación? Probablemente lo hayas adivinado: _ ** Falta a la mejor ruta ** _

## Fuerte a la mejor ruta

Veamos cómo lo describe la documentación:

> Cuando el tráfico de datos no cumple con ninguno de los requisitos de la clase SLA, esta característica le permite seleccionar la mejor secuencia de criterios de ruta del túnel utilizando el túnel mejor alternativo.
>
> El gerente de Cisco SD-WAN usa Best of Weor (Bow) para encontrar el mejor túnel cuando ningún túnel cumple con ninguno de los requisitos de la clase SLA.
>
> https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-ware-routing.html

### lo mejor de lo peor

Veamos cómo funciona Bow con el siguiente ejemplo:

![](/wp-content/uploads/2024/02/s3-aar5.png)

El requisito de latencia de SLA se establece en 8. Ninguno de los túneles lo satisface, pero el túnel 1 es el más cercano, lo que lo convierte en lo mejor de peor con una latencia de 10.

Los criterios para elegir el arco son extremadamente flexibles, en este ejemplo utilizamos latencia, otras opciones podrían ser:

- Latencia - Solo latencia
- Jitter - Solo jitter
- Pérdida - Solo pérdida
- Latencia/Jitter - Primera latencia, si son iguales, entonces Jitter
- Latencia/pérdida - Primera latencia, si son iguales, entonces pérdida
- Jitter/Latencia - Primer jitter, si son iguales, entonces latencia
-. . .
- pérdida/jitter/latencia - Primera pérdida, luego fase, luego latencia

### Varianza

Vamos un paso más allá, el túnel 3 también está muy cerca de la latencia _8 ms_, no sería una mala idea enviar tráfico también en ese túnel. ¿Cómo logramos esto? Bueno, podemos implementar un _variance_ para acomodar pequeñas variaciones al elegir las mejores rutas.

Continuando con este ejemplo, echemos un vistazo a la selección de arco con una `varianza de 5 ms '.

`` `` ``
Bow Range = (mejor latencia, mejor latencia + varianza)
Rango de arco = (10, 15)
`` `` ``

La mejor latencia en los túneles es 10 (Túnel 1), observe que esto ** no es ** la latencia configurada en el SLA. Con esta varianza, si algún otro túnel tiene una latencia entre 10 y 15, también se eligirá para enviar tráfico. En nuestro ejemplo, el túnel 3 satisface la condición, por lo que ahora el túnel 1 y el túnel 3 se utilizarán como túneles _fallback_.

Como puede ver, el túnel 2 ya no se considera. ¡Excelente!

## Configuración

#### Clase SLA

Para usar este método, lo primero que debemos hacer es modificar nuestra clase SLA para indicar que debe buscar la ruta de mejor rendimiento cuando no se cumple SLA. Nuestra configuración se ve así:

![](/wp-content/uploads/2024/02/s3-aar7.png)

* * *

`` `` ``
SLA-CLASS Custom-SLA
 Pérdida 1
 Latencia 250
 Jitter 100
 alero-mejor túnel
  pérdida de criterios
  Varianza de pérdidas 2
`` `` ``

Observe que seleccionamos el _criteria_ para ser `pérdida 'y un _variance_ de` 2`. La varianza es un parámetro ** opcional **.

#### Política AAR

A continuación, en la política de AAR especificamos la acción cuando no se cumple SLA: _Fallback a Best Path_

![](/wp-content/uploads/2024/02/s3-aar22.png)

* * *

`` `` ``
secuencia 1
 fósforo
  Fuente-IP 172.16.10.0/24
  Destino-IP 172.16.20.0/24
 acción
  SLA-CLASS Custom-SLA
  No hay clase SLA estricta
  MPLS de color de clase SLA
  SLA-Class Fallback-to-Best-Path
`` `` ``

Mantengamos la misma dinámica y construamos un diagrama:

![](/wp-content/uploads/2024/02/s3-aar8.png)

## Escenarios

Usando la misma topología exploremos algunas situaciones

![](/wp-content/uploads/2024/02/aar-topo2.png)

#### MPLS compatible, biz-internet/private1 no conforme

Iniciaré el tráfico de Branch10 -> Branch 20 y lo capturaré con NWPI. MPLS tiene métricas KPI perfectas (0, 0, 0)

![](/wp-content/uploads/2024/02/s3-aar1-1-1.png)

Observe cómo _fallback to Best Path_ se establece en `no 'y el tráfico coincide con el SLA y el color preferido. Veamos también el siguiente comando de verificación de BR10.`` `` ``
BR10#show sdwan tunnel sla
<. . .>
Túnel SLA-Class 1
 SLA-NAME Custom-SLA
 SLA-Loss 1
 SLA-Latencia 250
 sla-jitter 100
                                                                                                                                   RETROCEDER
                                         Remoto t sla sla
                             Sistema DST SRC Local t remota media media media clase de clase
PROTO SRC IP DST Puerto IP Puerto IP Color Pérdida de color Pérdida de latencia Jitter Índice SLA Nombre de clase Índice
-------------------------------------------------------------------------------------------------------------------------------------
Gre 21.1.10.2 21.1.20.2 0 0 1.1.0.20 MPLS MPLS 0 0 0 0,1 __all_tunnels__, Custom-SLA Ninguno
Gre 21.1.10.2 31.1.20.2 0 0 1.1.0.20 MPLS Biz-Internet 0 0 0 0,1 __all_tunnels__, Custom-SLA Ninguno
GRE 21.1.10.2 41.1.20.2 0 0 1.1.0.20 MPLS Private1 0 0 0 0,1 __all_tunnels__, Custom-SLA Ninguno
`` `` ``

Algunos comentarios sobre este resultado:

1. El hecho de que veamos túneles enumerados en Custom-SLA, nos dice que hay túneles que cumplen con la pérdida, la latencia y la fluctuación. Esto se espera ya que nuestro MPLS está teniendo métricas perfectas.
1. Vea que esta clase SLA tiene un identificador numérico de 1-`Túnel SLA-Class 1`. Verá referencia a este número en breve.
1. _Fallback SLA Class Index_ está configurado en `Ninguno`, esto significa que estos túneles no se están utilizando como túneles de respaldo, esto quedará claro en un segundo.

#### mpls/biz-inernet/private1 no conforma pero varianza de reunión

Ahora que no tenemos transportes que cumplan con el SLA, verifiquemos cómo lo mostrará NWPI:

![](/wp-content/uploads/2024/02/s3-aar10.png)

Observe que ahora NWPI está indicando que _fallback a la mejor ruta_ está en uso.

Private1 fue elegido para enviar este flujo en particular, pero ¿hay otros túneles que pudieran usarse? Vamos a ver BR10 nuevamente.

`` `` ``
BR10# show sdwan tunnel sla
Túnel SLA-Class 0
 sla-name __all_tunnels__
 SLA-Loss 0
 SLA-Latencia 0
 sla-jitter 0
                                                                                                                              RETROCEDER
                                         SLA SLA remoto
                             Sistema DST SRC T Local T Remote media media media clase
PROTO SRC IP DST Puerto IP Puerto IP Color Pérdida de color Pérdida de latencia Jitter Índice SLA Nombre de clase Índice
------------------------------------------------------------------------------------------------------------------------------------
Gre 21.1.10.2 21.1.20.2 0 0 1.1.0.20 MPLS MPLS 18 0 0 0 0 __all_tunnels__ Ninguno
GRE 21.1.10.2 31.1.20.2 0 0 1.1.0.20 MPLS Biz-Internet 9 0 0 0 __all_tunnels__ Ninguno
Gre 21.1.10.2 41.1.20.2 0 0 1.1.0.20 MPLS Private1 11 0 0 0 __all_tunnels__ Ninguno
Gre 31.1.10.2 21.1.20.2 0 0 1.1.0.20 Biz-Internet Mpls 4 0 0 0 __all_tunnels__ 1
Gre 31.1.10.2 31.1.20.2 0 0 1.1.0.20 Biz-Internet Biz-Internet 2 0 0 0 0 __all_tunnels__ 1
Gre 31.1.10.2 41.1.20.2 0 0 1.1.0.20 Biz-Internet Private1 4 0 0 0 __all_tunnels__ 1
GRE 41.1.10.2 21.1.20.2 0 0 1.1.0.20 Private1 MPLS 3 0 0 0 0 __all_tunnels__ 1
GRE 41.1.10.2 31.1.20.2 0 0 1.1.0.20 Private1 Biz-Internet 6 0 0 0 __all_tunnels__ Ninguno
GRE 41.1.10.2 41.1.20.2 0 0 1.1.0.20 Private1 Private1 3 0 0 0 0 __all_tunnels__ 1

Túnel SLA-Class 1
 SLA-NAME Custom-SLA
 SLA-Loss 1
 SLA-Latencia 250
 sla-jitter 100

BR10#
`` `` ``

Comentarios sobre este resultado:

1. Hay ** no ** túneles que cumplen con nuestro _custom-sla_
1. Incluso si MPLS es el color preferido, ** no es ** considerado porque no cumple con el rango de varianza de pérdida.
1. Hay 5 túneles que satisfacen la varianza. Observe cómo algunos túneles tienen una clase SLA de _Fallback SLA Index_ establecido en `1`, lo que significa que están sirviendo como alternativos para SLA-Class 1 (Custom-SLA). En este caso, el ** arco ** es _biz-instantet-biz-insternet_ tunnel con una pérdida media de `2`. El _variance_ se establece en 2, por lo que el rango de arco es 2-4. Los túneles que satisfacen la gama de arco también se utilizarán para reenviar el tráfico.

Dependiendo del hash de equilibrio de carga, se eligirán diferentes túneles`` `` ``
BR10#show Sdwan Policy Service-Path VPN 10 Interfaz GigabitEthernet 3 Fuente-IP 172.16.10.2 Dest-IP 172.16.20.2 Protocolo 6 Dest-Port 22
Siguiente salto: Gre
  Fuente: 31.1.10.2 Destino: 21.1.20.2 Color local: Biz-Internet Color remoto: Sistema remoto MPLS IP: 1.1.0.20

BR10#show SDWAN Policy Service-Path VPN 10 Interfaz GigabitEthernet 3 Fuente-IP 172.16.10.56 Dest-IP 172.16.20.2 Protocolo 6 Dest-Port 24
Siguiente salto: Gre
  Fuente: 31.1.10.2 Destino: 41.1.20.2 Color local: Biz-Internet Color remoto: Private1 Sistema remoto IP: 1.1.0.20
`` `` ``

Podemos ver dos túneles diferentes `biz -internet - mpls` y` biz -internet - privado`

#### Latencia fuera de cumplimiento

Hasta ahora hemos estado jugando solo con pérdida, porque los criterios de alojamiento estaban establecidos en la pérdida. Veamos qué sucede cuando un criterio diferente no se cumple. Estableceré la latencia del SLA a 15 ms. .

`` `` ``
BR10#show sdwan política de vsmart
From-vsmart SLA-Class Custom-SLA
 Pérdida 1
 latencia 15
 Jitter 100
 alero-mejor túnel
  pérdida de criterios
  Varianza de pérdidas 2

`` `` ``

Después de presentar algo de latencia, vemos algo interesante:

`` `` ``
BR10#show sdwan tunnel sla
Túnel SLA-Class 0
 sla-name __all_tunnels__
 SLA-Loss 0
 SLA-Latencia 0
 sla-jitter 0
                                                                                                                              RETROCEDER
                                         SLA SLA remoto
                             Sistema DST SRC T Local T Remote media media media clase
PROTO SRC IP DST Puerto IP Puerto IP Color Pérdida de color Pérdida de latencia Jitter Índice SLA Nombre de clase Índice
------------------------------------------------------------------------------------------------------------------------------------
Gre 21.1.10.2 21.1.20.2 0 0 1.1.0.20 MPLS MPLS 0 20 1 0 __all_tunnels__ 1
GRE 21.1.10.2 31.1.20.2 0 0 1.1.0.20 MPLS Biz-Internet 0 20 1 0 __all_tunnels__ 1
Gre 21.1.10.2 41.1.20.2 0 0 1.1.0.20 MPLS Private1 0 20 0 0 __all_tunnels__ 1
Gre 31.1.10.2 21.1.20.2 0 0 1.1.0.20 Biz-Internet Mpls 0 21 2 0 __all_tunnels__ 1
Gre 31.1.10.2 31.1.20.2 0 0 1.1.0.20 Biz-Internet Biz-Internet 0 21 2 0 __all_tunnels__ 1
Gre 31.1.10.2 41.1.20.2 0 0 1.1.0.20 Biz-Internet Private1 0 21 2 0 __all_tunnels__ 1
Gre 41.1.10.2 21.1.20.2 0 0 1.1.0.20 Private1 MPLS 0 16 1 0 __all_tunnels__ 1
GRE 41.1.10.2 31.1.20.2 0 0 1.1.0.20 Privado1 Biz-Internet 0 16 0 0 __all_tunnels__ 1
GRE 41.1.10.2 41.1.20.2 0 0 1.1.0.20 Private1 Private1 0 16 0 0 __all_tunnels__ 1

Túnel SLA-Class 1
 SLA-NAME Custom-SLA
 SLA-Loss 1
 SLA-Latencia 15
 sla-jitter 100

BR10#
`` `` ``

¡Todos los túneles se usan como túneles respaldados porque todos tienen un 0% de pérdida! ¿Es esta la situación ideal? Esto es discutible, tal vez para algunos tipos de tráfico está bien, pero para otros probablemente desee tener un segundo o tercer criterio para elegir los mejores túneles de respaldo.

#### Criterios de arco múltiple

Para la última prueba, veamos qué sucede cuando seleccionamos múltiples criterios para seleccionar el arco. Agregaré latencia a los criterios de SLA.

`` `` ``
BR10#show sdwan política de vsmart
From-vsmart SLA-Class Custom-SLA
 Pérdida 1
 latencia 15
 Jitter 100
 alero-mejor túnel
  latencia de pérdida de criterios
  Varianza de pérdidas 2
`` `` ``

Lo que esperamos es que si alguno de los KPI no cumple, el arco se decidirá en base a:

1. Pérdida media más baja. Si hay un empate, entonces
1. Latencia más baja

Verifiquemos`` `` ``
BR10#show sdwan tunnel sla
Túnel SLA-Class 0
 sla-name __all_tunnels__
 SLA-Loss 0
 SLA-Latencia 0
 sla-jitter 0
                                                                                                                              RETROCEDER
                                         SLA SLA remoto
                             Sistema DST SRC T Local T Remote media media media clase
PROTO SRC IP DST Puerto IP Puerto IP Color Pérdida de color Pérdida de latencia Jitter Índice SLA Nombre de clase Índice
------------------------------------------------------------------------------------------------------------------------------------
Gre 21.1.10.2 21.1.20.2 0 0 1.1.0.20 MPLS MPLS 0 20 1 0 __all_tunnels__ Ninguno
Gre 21.1.10.2 31.1.20.2 0 0 1.1.0.20 MPLS Biz-Internet 0 20 1 0 __all_tunnels__ Ninguno
Gre 21.1.10.2 41.1.20.2 0 0 1.1.0.20 MPLS Private1 0 20 1 0 __all_tunnels__ Ninguno
Gre 31.1.10.2 21.1.20.2 0 0 1.1.0.20 Biz-Internet Mpls 0 20 1 0 __all_tunnels__ Ninguno
Gre 31.1.10.2 31.1.20.2 0 0 1.1.0.20 Biz-Internet Biz-Internet 0 20 1 0 __all_tunnels__ Ninguno
Gre 31.1.10.2 41.1.20.2 0 0 1.1.0.20 Biz-Internet Private1 0 21 1 0 __all_tunnels__ Ninguno
GRE 41.1.10.2 21.1.20.2 0 0 1.1.0.20 Private1 Mpls 0 16 0 0 __all_tunnels__ 1
GRE 41.1.10.2 31.1.20.2 0 0 1.1.0.20 Privado1 Biz-Internet 0 16 0 0 __all_tunnels__ 1
GRE 41.1.10.2 41.1.20.2 0 0 1.1.0.20 Private1 Private1 0 16 0 0 __all_tunnels__ 1

Túnel SLA-Class 1
 SLA-NAME Custom-SLA
 SLA-Loss 1
 SLA-Latencia 15
 sla-jitter 100

BR10#
`` `` ``

Está claro que la pérdida no se usó para elaborar el arco, de lo contrario, veríamos todos los túneles actuando como respaldo para el índice de clase SLA 1. En cambio, se seleccionaron túneles con latencia más baja. Tenga en cuenta que podría haber agregado una varianza de latencia para incluir otros túneles con números similares.

## Conclusión

A través de las últimas tres publicaciones, hemos sido testigos de AAR como una funcionalidad crítica de SD-WAN para proteger el SLA de nuestras aplicaciones. Espero que después de explicar y verificar diferentes escenarios, ahora tenga una mejor comprensión y se sienta más seguro de probar AAR dentro de su infraestructura SD-WAN. Guía] (https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/policies/ios-xe-17/policies-book-xe/application-ware-routing.html#config-variance-best-tunnel-path) es muy completo, lo que está familiarizado con it y use lo necesita.

Déjame saber tus pensamientos en los comentarios.

¡Nos vemos pronto!