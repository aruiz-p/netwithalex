---
author: Alex
category:
  - AAR
  - Políticas
  - SD-WAN
date: "2024-02-01T16:25:27+00:00"
title: 'Application Aware Routing (AAR): 1/3 Las bases'
description: Aprende cómo AAR puede ayudarte a mejorar la experiencia del usuario y aplicación con Cisco SD-WAN  
summary: Aprende cómo AAR puede ayudarte a mejorar la experiencia del usuario y aplicación con Cisco SD-WAN  
URL: /simplificando-aar-1-3-las-bases/

---
### Introducción

Piensa en las tecnologías de enrutamiento que existen; la mayoría de ellas han mejorado mucho en reaccionar ante fallos de enlaces y cortes de energía mediante protocolos como OSPF LFA/FRR, EIGRP Feasible Successor, BGP PIC, etc.

Sin embargo, estos protocolos no son tan eficaces cuando se trata de abordar problemas como la degradación del rendimiento de la red durante brownouts, que pueden ser causados por fluctuaciones de energía o congestión en los enlaces. Estos escenarios presentan nuevos desafíos para los cuales los protocolos de enrutamiento tradicionales no tienen herramientas adecuadas.

Aquí es donde Application Aware Routing llega al rescate.

A pesar de generar cierta confusión entre los clientes y quienes están aprendiendo sobre SD-WAN, Application Aware Routing (AAR) representa una ventaja fundamental de esta tecnología.

En esta serie, exploraremos los conceptos básicos para comprender los principios y comportamientos clave que nos permitirán analizar diferentes escenarios.

Usaremos NWPI para entender mejor cómo funciona todo. Si no estás familiarizado con NWPI, te recomiendo leer este [post](/network-wide-path-insights-an-introduction/) que escribí sobre el tema. 

### AAR explicado en 5 líneas

Using IPSec tunnels formed between WAN Edges, **BFD** packets will be sent across them to measure the **loss, latency Utilizando túneles IPSec formados entre los WAN Edges, se enviarán paquetes BFD a través de ellos para medir la pérdida, latencia y jitter (KPIs).

Los usuarios pueden definir SLAs para distintos tipos de tráfico (voz, web, video, etc.) y configurar políticas de AAR para garantizar que el tráfico se envíe a través de rutas que cumplan con el SLA.

Si en algún momento la ruta deja de cumplir con el SLA, el tráfico será redirigido automáticamente a una ruta que sí lo haga.

Sencillo, ¿verdad? 😃

### SLAs

El Service Level Agreement (SLA) se refiere a la cantidad de pérdida, latencia y jitter que una aplicación puede tolerar mientras sigue funcionando correctamente. Su definición debe ser precisa y realista según el tipo de tráfico y el entorno de la red.

Por ejemplo, si estamos definiendo un SLA para voz, debemos conocer los valores aceptables de pérdida, latencia y jitter.

Si nuestro SLA de voz tuviera estos valores:

    Pérdida: 5%
    Latencia: 350 ms
    Jitter: 200 ms

Es muy probable que las llamadas tengan una mala calidad.

En cambio, estos valores serían mucho mejores:

    Pérdida: 1%
    Latencia: 150 ms
    Jitter: 50 ms

También es importante considerar la naturaleza del entorno al definir estos valores. Hay lugares donde los proveedores pueden ser menos confiables, el tráfico puede recorrer grandes distancias y los tipos de transporte pueden variar (satélite vs. fibra, por ejemplo).

Puedes utilizar los datos históricos de KPIs en SD-WAN Manager para construir una línea base y definir un SLA adecuado.

La definición de SLA se verá así:

![](/wp-content/uploads/2024/02/s1-aar100.png)

* * *

```
sla-class Custom-SLA
 loss 1
 latency 250
 jitter 100
```

### BFD

El protocolo Bidirectional Forwarding Detection (BFD) se usa para detectar rápidamente fallas entre dos dispositivos de red. En SD-WAN, también se usa para medir la pérdida, la latencia y el jitter. Los parámetros que usamos para configurar BFD dictarán qué tan rápido SD-WAN detectará y reaccionará a los problemas de red.

#### Hello Interval

Este intervalo representa la frecuencia con la que se enviará un paquete de BFD. Está configurado por color en milisegundos; El valor predeterminado **por defecto** es 1000 ms. Este paquete viajará hasta el WAN Edge del otro extremo y regresará. De esta manera, se medirán los KPIs para cada paquete.

![](/wp-content/uploads/2024/02/bfd-1.png)

![](/wp-content/uploads/2024/02/s1-aar101.png)

* * *

```
sdwan
 interface GigabitEthernetX
   tunnel-interface
   color mpls
   hello-interval 1000 <<<
```

Cada modelo de router tiene una capacidad definida de túneles (cantidad de túneles que puede formar). Reducir el hello interval por debajo de 1 segundo disminuye la capacidad de túneles.

**Ten esto en cuenta para evitar posibles problemas por exceder la capacidad del hardware.**

#### App route poll interval

Digamos que tenemos un intervalo de votación de 4000 ms. Cada 4 segundos se construye un nuevo cubo, este cubo tendrá su propio índice. ** El intervalo de encuesta predeterminado ** es de 600,000 ms (10 minutos).

A medida que el WAN Edge sigue enviando paquetes, el poll interval es el temporizador que ayuda a organizarlos en buckets. Estos buckets se utilizan para calcular las estadísticas del túnel.

El poll interval se configura a nivel de dispositivo (per box basis), lo que significa que se aplica de la misma manera para todos los colores.

Por ejemplo, si el poll interval es de 4000 ms, cada 4 segundos se crea un nuevo bucket, y cada uno tendrá su propio índice.

El **valor por defecto** del poll interval es 600,000 ms (10 minutos).

![](/wp-content/uploads/2024/02/bfd-4.png)

Para configurar el poll interval

![](/wp-content/uploads/2024/02/bfd-cofig.png)

* * *

```
bfd app-route poll-interval 60000
```

La pérdida promedio, la latencia y el jitter, se calcularán para cada poll interval. Podemos verificar esto directamente en _SD-WAN Manager -> Monitor -> Real Time -> App Route Statistics_

![](/wp-content/uploads/2024/02/bfd-5.png)

#### Multiplicador de ruta de la aplicación

![](/wp-Content/uploads/2024/02/captshot-2024-02-06-AT-12.16.05.png)

Para configurar el multiplicador

![](/wp-content/uploads/2024/02/bfd-cofig-1.png)

* * *

```
bfd app-route multiplier 3
```

El multiplicador indica la cantidad de buckets que se utilizarán para calcular las estadísticas del túnel, lo que influirá en la decisión de redirigir el tráfico cuando las condiciones de la red empeoren.

Para esta ilustración, se usa un multiplicador de 3.

Por defecto, el multiplicador está configurado en 6.

Observa que cada bucket contiene 4 paquetes.

`hello interval (s) x Poll interval (s)`

En el mundo real, 4 paquetes serían ** demasiado bajo **. Si tomamos todos los valores predeterminados, terminaríamos con un cubo de 600 paquetes. Consulte la [Guía de implementación de AAR] (https://www.cisco.com/c/en/us/td/docs/solutions/cvd/sdwan/cisco-sdwan-application-ware-routing-deploy-guide.html) para obtener más detalles y opciones.

Después de llenar todos los buckets, se calcularán las estadísticas del túnel.

![](/wp-content/uploads/2024/02/bfd-10.png)

Podemos visualizar los buckets y los cálculos de la media. Fíjate que los valores son los mismos para todos los buckets. No te confundas con las columnas de avg loss, latency y jitter que se mostraron antes.

![](/wp-content/uploads/2024/02/bfd-11.png)

Este mecanismo actúa como una ventana deslizante que descartará el bucket más antiguo para hacer espacio al nuevo, recalculando las estadísticas del túnel cada poll interval.

![](/wp-content/uploads/2024/02/bfd-9-1.png)

### Configuración de la política AAR

Ahora que tenemos una mejor comprensión sobre SLA y BFD, podemos crear las reglas que gobernarán el comportamiento de nuestra política AAR. Así es como se verá en SD-WAN Manager.

![](/wp-content/uploads/2024/02/s1-aar100-1.png)

Esta es probablemente la forma más simple que podemos configurar. El tráfico de Google Apps será _matcheado_ y se utilizarán los transportes que coincidan con nuestro Custom-SLA de manera indiferente.

En otras palabras, si tenemos 3 transportes diferentes y todos cumplen con el SLA, el tráfico se balanceará entre ellos.

¿Qué pasa si queremos que el tráfico prefiera uno de los transportes? Bueno, podemos especificar un _Prefered Color_ como se muestra a continuación:

![](/wp-content/uploads/2024/02/manger-aar.png)

En esta secuencia, las aplicaciones de voz se emparejan, y el SLA de los transportes debe coincidir con el de Bussiness-Critical. Se dará preferencia a MPLS si cumple con el SLA. La acción cuando no se cumple el SLA se configura en Load Balance entre los colores disponibles.

A primera vista, esto parece muy simple, pero hay algunos detalles que debemos conocer si queremos entender completamente cómo se comportará el tráfico bajo diferentes circunstancias.

Te invito a leer mi [próximo post](simplificando-aar-analizando-diferentes-escenarios) donde exploraremos algunos escenarios para tener un mejor entendimiento de esta tecnología. ¡Nos vemos allá!