---
author: Alex
category:
  - devnet
  - estudio
date: "2024-04-06T10:45:40+00:00"
title: Docker Golden Images
URL: /Docker-Golden-Images/
draft: true

---
### Introducción

El concepto de imagen dorada se refiere a una imagen que ha sido ampliamente probada y verificada su completamente operativa. Siguiendo la misma lógica, una imagen que no ha sido probada o verificada no debe considerarse una imagen dorada. Las organizaciones deben definir el proceso para seguir para marcar una imagen como ** Golden **.

El proceso debe incluir metodologías de prueba y verificación. Además, desde un punto de vista de seguridad, los parches deben estar presentes y si hay dependencias externas, es crucial actualizarlos a sus respectivas versiones recomendadas.

### Etiquetas de imagen

Etiquetar imágenes de Docker juega un papel crucial en la gestión e identificación de diferentes versiones de imágenes, incluidas las imágenes doradas.

#### Beneficios

- ** Versiones de identificación ** **- ** El etiquetado le permite asignar nombres significativos a las imágenes de Docker, lo que facilita identificar diferentes versiones o variaciones de una imagen.
- ** Control de versión ** **- ** Al etiquetar imágenes doradas con números de versión o nombres de lanzamiento, puede mantener un historial de cambios y rastrear qué versiones han sido probadas y aprobadas. Puedes revertir si es necesario.
- ** Gestión del ciclo de vida ** **- ** El etiquetado ayuda a administrar el ciclo de vida de las imágenes doradas proporcionando una indicación de su estado y propósito ("Dev", "Qa", "Prod")
- ** Tubería de promoción ** **- ** Las etiquetas se pueden usar para significar la progresión de las imágenes a través de diferentes etapas de prueba y aprobación.

### Acceso a imágenes doradas

Las imágenes deben estar disponibles y accesibles a través de registros de confianza. Un ejemplo es el Docker Hub, que se monitorea y actualiza regularmente en función de las vulnerabilidades, las correcciones, etc. También es posible tener registros privados e imágenes base.

#### Ubicaciones comunes

-** Docker Hub -** Servicio basado en la nube proporcionado por Docker que sirve como un repositorio centralizado para imágenes de Docker. Puede tener registros públicos y privados.
-** Registro de confianza de Docker (DTR) -** Solución de almacenamiento de imágenes de grado empresarial proporcionada por Docker. Extiende las capacidades de Docker Hub para proporcionar características mejoradas de seguridad, cumplimiento y colaboración para organizaciones con estrictos requisitos reglamentarios y de seguridad.
-** Registros privados de Docker -** Los registros de Docker privados son repositorios autohospedados para almacenar imágenes de Docker dentro de la infraestructura de una organización. Los registros privados de Docker generalmente se implementan en las instalaciones o dentro de entornos de nube privados.

#### ciclo de vida de imagen

Por lo general, los pasos para cargar nuevas imágenes son los siguientes:

- Crea nuestro Dockerfile
- construir la imagen
- etiqueta la imagen
- Empujar la imagen

#### Proceso de empuje

Estos comandos se utilizan para cargar imágenes de Docker en un registro:

** Docker Iniciar sesión <URL> ** **: ** Este comando autentica el cliente Docker con la URL de registro Docker especificada. Le solicita que ingrese su nombre de usuario y contraseña para la autenticación. Una vez autenticado, Docker almacena un token de autenticación localmente, lo que le permite interactuar con el registro sin necesidad de iniciar sesión nuevamente hasta que expire el token.

`` `` ``
Registro de inicio de sesión de Docker.example.com
`` `` ``

** Docker Build -t <Image \ _name> -f <Dockerfile>. ** **: ** Este comando construye una imagen Docker usando DockerFile especificado (`-f`) y etiquetas (` -t`) con el nombre de la imagen dado. El '.' Al final del comando indica que el contexto de compilación es el directorio actual, donde se encuentra el DockerFile. Por ejemplo:

`` `` ``
Docker Build -t DataStore -f DockerFile_Datastore.
`` `` ``

** Docker Push <Registry \ _url>/<Image \ _name>: <tag> ** **: ** Este comando carga la imagen de Docker especificada en el registro con la URL dada. Incluye la ruta del repositorio (`<registry_url>/<Image_name>: <tag>`) donde la imagen se almacenará en el registro. Antes de presionar la imagen, asegúrese de haberlo etiquetado correctamente con la ruta del repositorio deseada. Por ejemplo:

`` `` ``
Docker Push Registry.example.com/datastore: Latest
`` `` ``

Siguiendo estos comandos, puede autenticarse con el Registro de Docker, construir su imagen Docker y luego empujarla al Registro de almacenamiento y distribución. Este proceso permite compartir las aplicaciones contenedoras con otros o implementarlas en entornos de producción.

### Referencias adicionales

[Docker Hub] (https://docs.docker.com/docker-hub/)

[Docker Trusted Registry (DTR)] (https://dockerlabs.collabnix.com/beginners/dockertrustedregistry.html)

[Registro de Docker privado] (https://docs.docker.com/registry/)

[Docker Image Build and Push] (https://docs.docker.com/reference/cli/docker/image/build/)

### [Regrese a BluePrint] (/Study-Devops-BanePrint/)