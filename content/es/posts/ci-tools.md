---
author: Alex
category:
  - devnet
  - material de estudio
date: "2024-01-18T19:46:23+00:00"
title: Herramientas de integración continua (CI)
url: /herramientas-ci/
draft: true
---
## Describa las características y conceptos de herramientas de compilación /implementación como Jenkins, Drone o Travis CI

En la [Última publicación] (/DevOps-CI-CD-Pipeline/) hablamos sobre los conceptos de CI/CD, enfatizando la importancia de la fase ** _ construcción _ ** como parte del proceso de integración continua. Ahora es el momento de profundizar en las siguientes herramientas de CI: Jenkins, Drone, Travis y Gitlab.

### Jenkins

Jenkins es un servidor de automatización _ ** gratuito (código abierto) diseñado para automatizar partes del proceso de desarrollo de software, sobre todo el edificio, las pruebas y la implementación de cambios en el código. Está basado en Java, tiene una gran comunidad y ha estado allí por un tiempo.

Ofreciendo modelanguage/structureOtUrlvm/contenedor (autogestionado) groovyjenkinsfilehttps: //www.jenkins.io/


Su tubería se mantiene en formato maravilloso y se procesa como guión maravilloso. Este es el archivo que Jenkins buscará de forma predeterminada

Aquí hay un ejemplo para comprender la estructura:

`` `` ``
tubería {
  agente cualquiera

  etapas {
    Stage ('Checkout') {
      pasos {
        <...>
      }
    }

    etapa ('construir') {
      pasos {
        <...>
      }
    }

    etapa ('test') {
      pasos {
        <...>
      }
    }

    etapa ('implement') {
      pasos {
        <...>
      }
    }
  }

  correo{
    <...>
  }
}
`` `` ``

- `` `Etapas`` `**` --` ** 4 (pago, construir, probar e implementar)
- `Pasos` ** `--` ** Acciones para ejecutar en cada una de las etapas
- `Post` `` `` Secciones que contienen acciones para ejecutar en función del resultado de la tubería (éxito, falla, siempre)

Visite esta página para obtener más información sobre la [tubería] (https://www.jenkins.io/doc/book/pipeline/)

Jenkins también ofrece una interfaz de usuario para la visualización de tuberías llamado _ [Ocean Blue] (https://www.jenkins.io/doc/book/blueocean/) _

![](/wp-content/uploads/2024/01/screenshot-2024-01-31-at-07.26.51.png) https://www.jenkins.io/blog/2016/05/26/introducing-blue-cean/

La idea detrás de esto es mejorar la experiencia del desarrollador: facilitar la comprensión de la tubería, hacer cambios y monitorear el estado y los problemas.

* * *

### dron

Construido sobre una tecnología de contenedores. El conjunto de características se puede aumentar a través de ** complementos ** que pueden funcionar con las tecnologías de control de fuente (GitHub, Bitbucket, GitLab, etc.).

Dos versiones disponibles - ** GRATIS ** Para código abierto y servicio en la nube (no gratis).

Ofreciendo modelanguage/structureoturlsaas/contenedor (autogestionado) yaml.drone.ymlhttps: //www.drone.io/

Está escrito en GO y usa la siguiente estructura .Drone.yml:

`` `` ``
tipo: tubería
Nombre: Ejemplo

pasos:
  - Nombre: instalar
    Imagen: Nodo: 14
    Comandos:
      - instalación de NPM

  - Nombre: prueba
    Imagen: Nodo: 14
    Comandos:
      - Prueba de NPM

  - Nombre: implementar
    Imagen: Alpine
    Comandos:
      - Echo "Implementación de la aplicación"
      # Incluye comandos de implementación o scripts aquí

`` `` ``

- `amable` ** `--` ** Indica que esta es una configuración de tuberías (otros valores podrían ser Docker, Kubernetes, Secret)
- `Nombre --` Nombre de la tubería
- `Pasos` ** `--` ** Indique secciones individuales de la tubería. Cada paso tiene su propio nombre, imagen de Docker y comandos para ejecutar.

Drones GUI te permitirá

Mira este [video] (https://youtu.be/kzmoclcpvmk) por arnés para familiarizarse con el dron y la GUI.

Visite [Drone.io] (https://www.drone.io/) para obtener más información sobre esta herramienta, complementos y verificar la documentación.

* * *

### Travis CI

Se integra muy bien con la API de Github.com. Se ofrecen como proyectos de software como servicio y código abierto se pueden probar para ** _ gratis _ **. VM/opciones de contenedores también están disponibles.

Ofreciendo modelanguage/structureoturlsaas/contenedor (autogestionado) yaml.travis.ymlhttps: //travis-ci.org

`` `` ``
---
Idioma: node_js
node_js:
  - "14"

Env:
  - Ci_reg_img_app = "my_node_app"

antes_script:
  - Echo "Iniciar sesión en Dockerhub ..."
  - echo "$ docker_password" | Docker Login -U "$ Docker_username" - -Password -Stdin

guion:
  - Echo "Building Docker Image ..."
  - Docker Build -T $ CI_REG_IMG_APP.

desplegar:
  <...>

`` `` ``

- `lenguaje` y `nodo` ** `--` ** Define el lenguaje que se utilizará y las versiones
- `env` ** `--` ** La definición de variable de entorno ocurre aquí
- `antes_script` ** `--` ** Contiene comandos que se ejecutarán antes de ejecutar el script de compilación principal
- `script` ** `--` ** contiene el script de compilación principal
- `implement` ** `--` ** Especifica la configuración de la implementación

Recomiendo consultar este [video] (https://www.youtube.com/watch?v=_og2kydtlwk) que muestra lo fácil que es integrarse con Github y comenzar con Travis.

* * *

### gitlab

Enterprise y edición comunitaria disponible.

Gitlab tiene muchas características para DevOps:

- Repositorio Git
- Registro de imágenes de Docker
- Junta de proyectos
- tuberías CI/CD
- corredores

Ofreciendo modelanguage/structureoturlsaas/contenedor (autogestionado) yaml.gitlab-ci.ymlhttps: //gitlab.com

`` `` ``
---
etapas:
  - construir
  - prueba
  - desplegar

Variables:
  Node_version: "14"antes_script:
  - NPM instalación -g hilo
  - Instalación del hilo

construir:
  Etapa: construir
  guion:
    - Construcción del hilo

prueba:
  Etapa: prueba
  guion:
    - Prueba de hilo

desplegar:
  Etapa: desplegar
  guion:
    - Echo "Implementación de la aplicación ..."
    # Agregar comandos de implementación aquí
  solo:
    - principal
`` `` ``

- `Etapas` `` `define las etapas de la tubería
- `Variables -` Define las variables globales para la tubería.
- `antes de script -` Especifica los comandos que se ejecutarán antes de ejecutar cualquier trabajo en la tubería
- Etapa `construir -` para construir la aplicación. Ejecuta el comando `yarn build`.
- `Test` `` `` Etapa para ejecutar pruebas automatizadas antes de la implementación
- `implementar - 'responsable de implementar la aplicación

Aquí hay un ejemplo de cómo esto podría representarse gráficamente.

![](/wp-content/uploads/2024/02/gitlab1.png)

Sugeriría explorar algunos de los [tutoriales] (https://docs.gitlab.com/ee/tutorials/) para familiarizarse con Gitlab.