---
author: Alex
draft: false
category:
  - sdwan
date: "2025-07-15T00:00:00+00:00"
title: Simplificando la gestión de túneles con Tunnel Groups en Cisco SD-WAN 
summary: Descubre cómo los Tunnel Groups en Cisco SD-WAN simplifican la gestión de túneles a gran escala. Aprende cómo funcionan, en qué se diferencian de las políticas de control y cuándo usarlos con ejemplos del mundo real.
description: Descubre cómo los Tunnel Groups en Cisco SD-WAN simplifican la gestión de túneles a gran escala. Aprende cómo funcionan, en qué se diferencian de las políticas de control y cuándo usarlos con ejemplos del mundo real.
url: /tunnel-groups-es/
tag:
  - bfd
  - topology
---
## Introducción

SD-WAN proporciona una arquitectura de overlay altamente flexible que permite construir túneles sobre múltiples tipos de transporte. Un aspecto clave de esta flexibilidad es cómo se reenvía el tráfico entre dispositivos edge a través de túneles basados en colores de transporte como **privados** (mpls, private1, etc) y **públicos** (public-internet, biz-internet, etc).

Existen un par de formas de controlar qué túneles se crean, siendo las más destacadas las [Políticas de Control](/policies-intro-es/) y los Tunnel Groups.

Los Tunnel Groups permiten a los administradores de red agrupar lógicamente túneles (TLOCs) bajo una etiqueta común, simplificando cómo se establecen los túneles entre dispositivos. Este enfoque hace que la gestión de túneles sea más escalable y fácil de mantener. En este post exploraremos qué son los Tunnel Groups, cómo funcionan y cómo pueden servir como alternativa a las Control Policies en ciertos escenarios.

## ¿Qué son los Tunnel Groups?

Un Tunnel Group es una etiqueta numérica que representa un conjunto de TLOCs (túneles) agrupados lógicamente según una intención definida.

{{< figure src="/wp-content/uploads/2025/07/tg1.png" title="Figura 1" caption="Tunnel Group configurado en Feature Template" >}}

Una vez que se configura un Tunnel Group ID, el router:

- Establecerá túneles con TLOCs que tengan el mismo Tunnel Group ID.
- Establecerá túneles con TLOCs que no tengan Tunnel Group ID (valor 0).

El Tunnel Group ID se propaga por OMP, igual que otros atributos de los TLOCs. Una diferencia clave entre Tunnel Groups y Políticas de Control es dónde se aplican: los Tunnel Groups controlan el establecimiento de túneles localmente en el dispositivo, mientras que las Control Policies determinan qué TLOCs son anunciados por el Controller (vSmart) a nivel central. Ambos mecanismos pueden usarse de forma independiente o combinada, dependiendo del diseño.

## Comportamiento del Tunnel Group: Ejemplo Real

Tomemos esta topología como referencia para el escenario:

{{< figure src="/wp-content/uploads/2025/07/tg3.png" alt="Topology" title="Figura 2" caption="Topología de Referencia" >}}

Vamos a validar cómo los Tunnel Group IDs influyen en qué túneles se forman realmente observando los TLOCs y sesiones BFD desde un dispositivo en el sitio 20.

```
Barcelona_BR20-1#show sdwan omp tlocs table
...

                                                                                                                                                                                   PUBLIC           PRIVATE                      AFFINITY    
ADDRESS                                           TENANT                                                        PSEUDO                   PUBLIC                   PRIVATE  PUBLIC  IPV6    PRIVATE  IPV6     BFD                 GROUP       
FAMILY   TLOC IP          COLOR            ENCAP  ID        FROM PEER        STATUS    TENANT NAME              KEY     PUBLIC IP        PORT    PRIVATE IP       PORT     IPV6    PORT    IPV6     PORT     STATUS  REGION ID   NUMBER      
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ipv4     1.1.10.1         mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.11.0.2        12426   21.11.0.2        12426    ::      0       ::       0        down    None        None        
         1.1.10.1         biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.11.0.2        12346   31.11.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.10.2         mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.12.0.2        12346   21.12.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.10.2         biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.12.0.2        12346   31.12.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.20.1         mpls             ipsec  0         0.0.0.0          C,Red,R   [Default]                1       21.21.0.2        12346   21.21.0.2        12346    ::      0       ::       0        up      None        None        
         1.1.20.1         biz-internet     ipsec  0         0.0.0.0          C,Red,R   [Default]                1       31.21.0.2        12346   31.21.0.2        12346    ::      0       ::       0        up      None        None        
         1.1.20.2         mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.22.0.2        12346   21.22.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.20.2         biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.22.0.2        12346   31.22.0.2        12346    ::      0       ::       0        down    None        None        
         1.1.100.1        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.101.0.2       12406   21.101.0.2       12406    ::      0       ::       0        up      None        None        
         1.1.100.1        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.101.0.2       12346   31.101.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.100.2        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.102.0.2       12346   21.102.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.100.2        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.102.0.2       12346   31.102.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.1        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.201.0.2       12346   21.201.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.1        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.201.0.2       12346   31.201.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.2        mpls             ipsec  0         1.0.0.3          C,I,R     [Default]                1       21.202.0.2       12346   21.202.0.2       12346    ::      0       ::       0        up      None        None        
         1.1.200.2        biz-internet     ipsec  0         1.0.0.3          C,I,R     [Default]                1       31.202.0.2       12386   31.202.0.2       12386    ::      0       ::       0        down    None        None        
```
Se reciben 14 TLOCs desde vSmart, pero solo algunos túneles están activos:

```
Barcelona_BR20-1#sh sdwan bfd sessions 
                                      SOURCE TLOC      REMOTE TLOC                                      DST PUBLIC                      DST PUBLIC         DETECT      TX                              
SYSTEM IP        SITE ID     STATE       COLOR            COLOR            SOURCE IP                                       IP                                              PORT        ENCAP  MULTIPLIER  INTERVAL(msec  UPTIME          TRANSITIONS   
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1.1.100.1        100         up          mpls             mpls             21.21.0.2                                       21.101.0.2                                      12406       ipsec  7           1000           0:02:23:34      2             
1.1.100.2        100         up          mpls             mpls             21.21.0.2                                       21.102.0.2                                      12346       ipsec  7           1000           0:02:23:33      1             
1.1.200.1        200         up          mpls             mpls             21.21.0.2                                       21.201.0.2                                      12346       ipsec  7           1000           6:03:57:35      0             
1.1.200.2        200         up          mpls             mpls             21.21.0.2                                       21.202.0.2                                      12346       ipsec  7           1000           6:03:57:36      0             
1.1.100.1        100         up          biz-internet     biz-internet     31.21.0.2                                       31.101.0.2                                      12346       ipsec  7           1000           0:02:23:34      1             
1.1.100.2        100         up          biz-internet     biz-internet     31.21.0.2                                       31.102.0.2                                      12346       ipsec  7           1000           0:02:23:33      1             
1.1.200.1        200         up          biz-internet     biz-internet     31.21.0.2                                       31.201.0.2                                      12346       ipsec  7           1000           6:03:57:35      0             
1.1.200.2        200         up          biz-internet     biz-internet     31.21.0.2                                       31.202.0.2                                      12426       ipsec  7           1000           0:00:00:08      0             
```
Si examinamos los detalles de los TLOC, vemos que hay 4 TLOCs con un group-id no coincidente de 10. El resto o bien coinciden o están sin configurar ([0]). Recuerda que los dispositivos dentro del mismo sitio no forman túneles BFD.

```
Barcelona_BR20-1#show sdwan omp tlocs | include tloc|mpls|internet|groups|site
tloc entries for 1.1.10.1      <<< Group ID Diferente
                 mpls
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.10.1       <<< Group ID Diferente
                 biz-internet
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.10.2       <<< Group ID Diferente
                 mpls
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.10.2       <<< Group ID Diferente
                 biz-internet
     site-id           10
     groups            [ 10 ]
     site-type        not set
tloc entries for 1.1.20.1    <<<< TLOC propio 
                 mpls
     site-id           20
     groups            [ 20 ]
     site-type        not set
     site-id           20
     groups            [ 20 ]
)     site-type        not set
tloc entries for 1.1.20.1    <<<< TLOC propio
                 biz-internet
     site-id           20
     groups            [ 20 ]
     site-type        not set
     site-id           20
     groups            [ 20 ]
)     site-type        not set
tloc entries for 1.1.20.2   <<<< mismo sitio
                 mpls
     site-id           20
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.20.2   <<<< mismo sitio
                 biz-internet
     site-id           20
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.100.1       #1
                 mpls
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.100.1       #2
                 biz-internet
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.100.2       #3
                 mpls
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.100.2        #4
                 biz-internet
     site-id           100
     groups            [ 0 ]
     site-type        not set
tloc entries for 1.1.200.1       #5
                 mpls
     site-id           200
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.200.1       #6
                 biz-internet
     site-id           200
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.200.2       #7
                 mpls
     site-id           200
     groups            [ 20 ]
     site-type        not set
tloc entries for 1.1.200.2       #8
                 biz-internet
     site-id           200
     groups            [ 20 ]
     site-type        not set
```

## Tunnel Group + opción restrict

Como se muestra arriba, solo se establecen túneles entre colores coincidentes, lo que significa que la opción restrict está habilitada en los dispositivos:

{{< figure src="/wp-content/uploads/2025/07/tg2.png" title="Figura 3" caption="Configuración de Tunnel Group con la opción 'restrict'" >}}

Si se desactiva la opción restrict para el túnel MPLS, veamos qué sucede con las sesiones BFD:

```
Barcelona_BR20-1#show sdwan bfd sessions 
                                      SOURCE TLOC      REMOTE TLOC                                      DST PUBLIC                      DST PUBLIC         DETECT      TX                              
SYSTEM IP        SITE ID     STATE       COLOR            COLOR            SOURCE IP                                       IP                                              PORT        ENCAP  MULTIPLIER  INTERVAL(msec  UPTIME          TRANSITIONS   
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
1.1.100.1        100         up          mpls             mpls             21.21.0.2                                       21.101.0.2                                      12406       ipsec  7           1000           0:03:14:45      2             
1.1.100.2        100         up          mpls             mpls             21.21.0.2                                       21.102.0.2                                      12346       ipsec  7           1000           0:03:14:44      1             
1.1.200.1        200         up          mpls             mpls             21.21.0.2                                       21.201.0.2                                      12346       ipsec  7           1000           6:04:48:46      0             
1.1.200.2        200         up          mpls             mpls             21.21.0.2                                       21.202.0.2                                      12346       ipsec  7           1000           6:04:48:47      0             
1.1.100.1        100         up          mpls             biz-internet     21.21.0.2                                       31.101.0.2                                      12346       ipsec  7           1000           0:00:03:37      0             
1.1.100.2        100         up          mpls             biz-internet     21.21.0.2                                       31.102.0.2                                      12346       ipsec  7           1000           0:00:03:37      0             
1.1.200.1        200         up          mpls             biz-internet     21.21.0.2                                       31.201.0.2                                      12346       ipsec  7           1000           0:00:03:37      0             
1.1.200.2        200         up          mpls             biz-internet     21.21.0.2                                       31.202.0.2                                      12426       ipsec  7           1000           0:00:03:37      0             
1.1.200.1        200         up          biz-internet     mpls             31.21.0.2                                       21.201.0.2                                      12346       ipsec  7           1000           0:00:03:33      0             
1.1.200.2        200         up          biz-internet     mpls             31.21.0.2                                       21.202.0.2                                      12346       ipsec  7           1000           0:00:03:35      0             
1.1.100.1        100         up          biz-internet     biz-internet     31.21.0.2                                       31.101.0.2                                      12346       ipsec  7           1000           0:03:14:45      1             
1.1.100.2        100         up          biz-internet     biz-internet     31.21.0.2                                       31.102.0.2                                      12346       ipsec  7           1000           0:03:14:44      1             
1.1.200.1        200         up          biz-internet     biz-internet     31.21.0.2                                       31.201.0.2                                      12346       ipsec  7           1000           6:04:48:46      0             
1.1.200.2        200         up          biz-internet     biz-internet     31.21.0.2                                       31.202.0.2                                      12426       ipsec  7           1000           0:00:51:19      0             
```
Ahora hay túneles entre diferentes colores, pero respetando el Tunnel Group ID.

En resumen, si la opción restrict está configurada junto con Tunnel Groups, los túneles se formarán si:

- El color del TLOC coincide.
- Los Tunnel Group IDs coinciden o uno de los extremos no tiene Tunnel Group configurado (Group 0).

## Casos de Uso

Veamos algunos casos reales para esta funcionalidad.

### Agrupar colores privados y públicos

Existen múltiples etiquetas para colores privados y públicos. Dependiendo del diseño de red, podrías terminar con diferentes nombres de colores por región. Ejemplo:

|Continente| Colores Públicos |  Colores Privados  |
|---------|-------------|-------------------|
|Europa   | biz-internet|   mpls            |
|América  | gold        | metro-ethernet    |

Usando Tunnel Groups, podemos separar fácilmente los túneles entre colores privados y públicos:

{{< figure src="/wp-content/uploads/2025/07/tg4.png" title="Figura 4" caption="Agrupación de Colores basada en Tunnel Group ID" >}}

Asignando Tunnel Group 10 a los colores privados, nos aseguramos que solo se establezcan túneles entre mpls y metro-ethernet. De igual forma, el Tunnel Group 20 permitirá túneles entre colores públicos, gold y biz-internet.

### Escalamiento Horizontal

Cuando los routers del data center deben construir túneles con muchas sucursales remotas, cada una usando múltiples transportes, pueden alcanzar el límite de escala de túneles de la plataforma. Tunnel Groups ayudan a distribuir la creación de túneles entre diferentes routers y sucursales, haciendo el diseño más escalable.

{{< figure src="/wp-content/uploads/2025/07/tg5.png" title="Figura 5" caption="Estrategia de Escalamiento Horizontal con Tunnel Groups" >}}

Así, cada sucursal estará conectada a ambos data centers, pero manteniendo bajo control la cantidad de sesiones en los routers del data center.

### Crear Mesh Parcial

Si necesitas crear una malla parcial, los Tunnel Groups pueden ser de gran ayuda.

{{< figure src="/wp-content/uploads/2025/07/tg6.png" title="Figura 5" caption="Topología de Mesh Parcial" >}}

En este caso, los dispositivos con Tunnel Group 10 y sin Tunnel Group (0) establecerán túneles entre ellos. De igual forma, los dispositivos con Tunnel Group 20 y sin Tunnel Group (0) formarán túneles.

## Conclusión

Los Tunnel Groups en Cisco SD-WAN ofrecen una forma escalable de controlar el establecimiento de túneles en despliegues pequeños o grandes. Al etiquetar TLOCs con IDs de grupo numéricos, puedes:

- Crear grupos lógicos de túneles sin depender únicamente del color.
- Reducir la complejidad de las políticas de control centralizadas.
- Diseñar topologías más predecibles y escalables como malla parcial, hub-and-spoke o escalamiento horizontal.

Ya sea para lidiar con nombres de colores inconsistentes entre regiones, escalar routers del data center o simplemente tener una forma más limpia de gestionar túneles, los Tunnel Groups ofrecen una herramienta potente para simplificar el diseño.

Cuando se combinan con la opción restrict, los Tunnel Groups te dan control granular sobre qué túneles se permiten,por color, por grupo o ambos. Con la estrategia adecuada, puedes reducir significativamente la cantidad de túneles innecesarios, manteniendo alta disponibilidad y buen rendimiento.
