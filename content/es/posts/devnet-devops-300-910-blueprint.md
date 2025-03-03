---
author: Alex
category:
  - devnet
  - estudio
date: "2024-01-10T18:46:45+00:00"
tags:
  - automatización
  - devOps
title: Devnet DevOps (300-910) - Blueprint
url: /estudio-devops-blueprint/
draft: true

---
** [20% 1.0 CI/CD Pipeline] (/DevOps-CI-CD-Pipeline/) **

- [1.1 Describa las características y conceptos de herramientas de compilación/implementación como Jenkins, Drone o Travis CI] (/CI-Tools/)
- 1.2 Identificar la secuencia, componentes e integraciones para implementar una tubería de CI/CD para un escenario determinado
- 1.3 Problemas de solución de problemas con una tubería de CI/CD, como fallas basadas en código, problemas de tubería e incompatibilidad de la herramienta
- 1.4 Identifique las pruebas para integrarse en una tubería CI/CD para un escenario determinado
-[1.5 Identifique la estrategia de implementación de lanzamiento (Canary, Rollbacks y Blue/Green) para un escenario determinado] (/CI-CD-Deployment-Strategies/)
- 1.6 Diagnóstico de problemas de gestión de dependencia del código que incluyen API, cadena de herramientas y bibliotecas

** 15% 2.0 Embalaje y entrega de aplicaciones **

- 2.1 Identifique los pasos para contener una aplicación
- 2.2 Identificar pasos para implementar múltiples aplicaciones de microservicio
- 2.3 Evaluar los diagramas de arquitectura de microservicios y contenedores basados ​​en requisitos técnicos y comerciales (seguridad, rendimiento, estabilidad y costo)
-[2.4 Identificar prácticas de manejo seguro para elementos de configuración, parámetros de aplicación y secretos] (/Docker-Safety-Prices)
- [2.5 Construya un archivo Docker para abordar las especificaciones de la aplicación] (/dockerfile/)
-[2.6 Describa el uso de imágenes doradas para implementar aplicaciones] (/Docker-Golden-Images/)

** 20% 3.0 Infraestructura de automatización **

- 3.1 Describa cómo integrar las prácticas de DevOps en una estructura de organización existente
- 3.2 Describa el uso de herramientas de gestión de configuración para automatizar servicios de infraestructura como Ansible, Puppet, Terraform y Chef
- [3.3 Construya un libro de jugadas Ansible para automatizar una implementación de la aplicación de servicios de infraestructura] (/ansible-notas/)
- 3.4 Construya una configuración de Terraform para automatizar una implementación de la aplicación de servicios de infraestructura
- 3.5 Describa la práctica y los beneficios de la infraestructura como código
- 3.6 Diseñe una validación previa a la verificación del estado de red en una tubería de CI/CD para un escenario determinado
- 3.7 Diseñe una validación previa a la verificación de la infraestructura de aplicación en una tubería de CI/CD para un escenario determinado
- 3.8 Describa los conceptos de extender las prácticas de DevOps a la red para NetDevops
- 3.9 Identifique los requisitos como la memoria, la E/S de disco, la red y la CPU necesaria para escalar la aplicación o servicio

** 15% 4.0 Cloud y Multicloud **

- 4.1 Describa los conceptos y objetos de Kubernetes
- 4.2 Implementar aplicaciones a un clúster de Kubernetes
- 4.3 Utilice objetos de Kubernetes para construir una implementación para cumplir con los requisitos
- 4.4 Interpreta la tubería para la entrega continua de un archivo de configuración de drones
- 4.5 Validar el éxito de una implementación de la aplicación en Kubernetes
- 4.6 Describa el método y las consideraciones para implementar una aplicación en múltiples entornos, como múltiples proveedores de nube, configuraciones de alta disponibilidad, configuraciones de recuperación de desastres y prueba de portabilidad en la nube
- 4.7 Describa el proceso de seguimiento y proyección de costos al consumir nube pública
- 4.8 Describa los beneficios de la infraestructura como código para el consumo de nubes públicas repetibles
- 4.9 Comparar estrategias de servicios en la nube (construir versus comprar)

** 20% 5.0 Registro, monitoreo y métricas **

- 5.1 Identifique los elementos de los sistemas log y métricos para facilitar la resolución de problemas de la aplicación, como problemas de rendimiento y transmisión de registros de telemetría
- 5.2 Implementar un sistema de recopilación e informes de registro para aplicaciones
- 5.2.a registros agregados de múltiples aplicaciones relacionadas
- 5.2.B Capacidades de búsqueda
- 5.2.C Capacidades de informes
- 5.3 Solucionar problemas de una aplicación distribuida utilizando AppDyanMics con monitoreo de rendimiento de la aplicación
- 5.4 Describa los principios de la ingeniería del caos
- 5.5 Construya scripts de Python que usan API para realizar estas tareas
- 5.5.Un construir un tablero de monitoreo
- 5.5.B notificar el espacio de los equipos de webex
- 5.5.c respondiendo a alertas y interrupciones
- 5.5.D Creación de notificaciones
- 5.5.e Monitoreo de verificación de salud
- 5.5.f incidentes de apertura y cierre
- 5.6 Identificar requisitos de aplicación adicionales para proporcionar visibilidad sobre la salud y el rendimiento de la aplicación
- 5.7 Describa las capacidades de Kubernetes relacionadas con el registro, el monitoreo y las métricas
- 5.8 Describa la integración del registro, el monitoreo y la alerta en un diseño de tuberías de CI/CD

** 10% 6.0 Seguridad **

- 6.1 Identificar métodos para asegurar una aplicación e infraestructura durante la producción y prueba en una tubería de CI/CD
- 6.2 Identificar métodos para implementar un ciclo de vida de desarrollo de software seguro