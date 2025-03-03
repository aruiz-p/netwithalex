---
author: Alex
draft: true
category:
  - devnet
  - estudio
date: "2024-01-17T12:12:31+00:00"
title: 1.0 CI/CD Pipeline
url: /DevOps-CI-CD-Pipeline /

---
### Conceptos de CI/CD

La integración continua/entrega continua/implementación es una metodología para desarrollar soluciones. La idea es automatizar tanto como sea posible para moverse rápidamente y optimizar el proceso de desarrollo. La tubería NetDevops incluye una fase de construcción en la que nos centraremos para comprender mejor el concepto de integración continua.

! [] (/WP-Content/uploads/2024/01/captura de captación-2024-01-18-AT-18.45.53.png) Netdevops Pipeline

En ** Integración continua (CI) **, una idea crucial es la carga frecuente de código en la base de código, a menudo varias veces al día. Esto implica la fusión constante del trabajo de desarrolladores con la base de código y la identificación temprana de problemas mediante el uso de pruebas.

! [] (/WP-Content/uploads/2024/01/captura de captación-2024-01-17-AT-5.23.10.png) Diagrama de CI

##### Beneficios

- Reducir el tiempo de mercado
- lograr ingresos y resultados a un ritmo más rápido
- Backlog más pequeño
- Cambios de código más pequeños
- Menos errores

##### Proceso de construcción

Implica actividades como:

1. ** Integración: ** Código de fusión. Si varias personas están trabajando en un proyecto, los cambios de cada uno de ellos deben consolidarse en un solo lugar.
1. ** Limpieza: ** Analice el código para identificar errores de programación, defectos de software, estilo y cantidades incorrectas de espacios. Algunas herramientas de pelusa para saber:
   - [Pylint] (https://pypi.org/project/pylint/)
   - [Pyflakes] (https://pypi.org/project/pyflakes/)
   - [Bandit] (https://bandit.readthedocs.io/en/latest/)
   - [negro] (https://black.readthedocs.io/en/stable/)
1. ** Prueba unitaria: ** Probar componentes individuales del código, no toda la base del código. Escriba las pruebas primero, luego el desarrollo de Código - Prueba de desarrollo (TDD). Algunas herramientas para saber:
   - [Pytest] (https://docs.pytest.org/en/7.4.x/)
   - [Unittest] (https://docs.python.org/3/library/unittest.html)
1. ** Build: ** Esta etapa implica que todas las pruebas de pelusa y unidad se han completado y el código está listo para implementarse en el entorno de puesta en escena. Aquí se construye y almacena un "artefacto" de forma independiente.

** Sugerencia: ** Explore las diferentes herramientas que se muestran arriba y conozca los conceptos básicos de ellas

### Herramientas CI

Estas son algunas de las herramientas CI más comunes que existen. En la próxima publicación los exploraremos con más detalles.

CI ToolsUrl ** _ Travis Ci _ ** [https://travis-ci.org] (https://travis-ci.org) ** _ jenkins _ ** [https: // jenkins .io] (https://jenkins.io) ** _ drone _ ** [https://drone.io] (https://drone.io) ** _ gitlab CI _ ** [https://gitlab.com] (https://gitlab.com)

¡Haga clic en [aquí] (/CI-Tools/) para aprender sobre ellos!