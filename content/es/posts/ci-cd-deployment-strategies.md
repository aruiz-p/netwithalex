---
author: Alex
category:
  - devnet
  - material de estudio
date: "2024-04-21T12:03:50+00:00"
title: Estrategias de implementación de CI/CD
url: /ci-ci-estrategias-despliegue/
draft: true
---
## Introducción

En el mundo de CI/CD, la implementación de actualizaciones de manera eficiente y con una interrupción mínima es crucial. Existen diferentes estrategias de implementación de liberación que la organización puede adoptar, incluidos ** _ Big-bang, Rolling, Blue-Green y Canary ._ **

## opción de implementación

### Despliegue de Big Bang

Todos los cambios se agrupan y se despliegan de inmediato; esta es la esencia de la implementación de Big Bang. Si bien simplifica el proceso de implementación, ya que todo se expulsa simultáneamente, también plantea _ ** riesgos significativos ** _. Si surgen problemas durante el despliegue, afectan todo el sistema de inmediato, lo que puede conducir a un tiempo de inactividad generalizado o un mal funcionamiento. El más adecuado para proyectos más pequeños o para aquellos con baja complejidad donde el impacto de las fallas es mínimo.

- Se requiere tiempo de inactividad, todo o nada de enfoque
- Todos los sistemas son iguales, no hay variación entre sí.
- No se puede probar el tráfico de producción hasta que la aplicación esté en producción. (Prueba limitada)

![](/wp-content/uploads/2024/04/bigbang-1.png)

### despliegue rodante

Toma un enfoque más cauteloso implementando gradualmente actualizaciones en diferentes segmentos de la infraestructura. Esta estrategia implica actualizar un subconjunto de servidores o componentes a la vez mientras mantiene el resto del sistema operativo. La implementación incremental de los cambios reduce el riesgo de fallas en todo el sistema y permite una reversión más rápida si surgen problemas. Es particularmente efectivo para aplicaciones a gran escala con arquitecturas distribuidas, ya que garantiza la disponibilidad continua mientras se aplican actualizaciones.

- Actualiza los componentes individuales a la vez

![](/wp-content/uploads/2024/04/rolling.png)

### despliegue azul-verde

Dos entornos de producción idénticos, denominados azul y verde, se mantienen simultáneamente. Mientras que un entorno (digamos Blue) sirve tráfico en vivo, el otro (verde) permanece inactivo. Cuando las actualizaciones están listas para la implementación, se aplican al entorno inactivo (verde). Una vez que las actualizaciones se implementan y proban con éxito, el tráfico se cambia del azul al entorno verde, cambiando efectivamente sus roles. Este enfoque ofrece implementaciones de tiempo cero hacia abajo y proporciona un mecanismo de reversión directa al volver al entorno anterior si surgen problemas.

- Es muy costoso tener dos implementaciones idénticas

![](/wp-content/uploads/2024/04/bluegreen.png)

### Canary

La idea es lanzar actualizaciones a un pequeño subconjunto de usuarios (** grupo canario **) o servidores antes de lanzarlas a todo el sistema. Al monitorear el comportamiento y el rendimiento del grupo canario, los desarrolladores pueden evaluar el impacto de los cambios y detectar cualquier problema potencial desde el principio. Si el grupo Canary se desempeña como se esperaba, las actualizaciones se amplían gradualmente a la audiencia más amplia. Si surgen problemas, la implementación se puede detener o retroceder antes de afectar todo el sistema. La implementación canaria es ideal para proyectos donde las pruebas exhaustivas y la mitigación de riesgos son cruciales, lo que permite la experimentación controlada sin afectar toda la base de usuarios.

![](/wp-content/uploads/2024/04/canary.png)

### tabla de comparación

** Estrategia de implementación **** Ventajas **** Consideraciones **** Cero tiempo de inactividad **** Complejidad **** Big Bang ** Simplifica el proceso de implementación  
\- Requiere un riesgo mínimo de coordinación de fallas en todo el sistema  
\- Opciones de reversión limitada No  
\- Permite una reversión más rápida  
\- Garantiza la disponibilidad continua durante el despliegue prevé un monitoreo y coordinación cuidadosos para garantizar una transición sin problemas  
\- puede prolongar el proceso de implementación para aplicaciones a gran escala 2 ** Blue-verde ** \- proporciona mecanismo de reversión directa.  
\- La configuración y configuración inicial pueden ser complejas  
\- Muy costoso mantener Yes3 ** Canary ** Permite la experimentación y las pruebas controladas  
\- Detección temprana de problemas potenciales  
\- Minimiza el riesgo al limitar el impacto a un pequeño subconjunto de usuarios o servidores requiere una cuidadosa selección y monitoreo del grupo Canary  
\- puede no ser adecuado para todas las aplicaciones, especialmente aquellas con pequeñas bases de usuarios o donde los cambios afectan todo el sistema igualmente 3

## [Regrese a BluePrint] (/Study-Devops-Blueprint)