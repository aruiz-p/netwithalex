---
author: Alex
category:
  - devnet
  - estudio
date: "2024-04-21T12:49:34+00:00"
title: Docker - Prácticas de seguridad
url: /docker-Practices-de-seguridad/
draft: true

---
## Introducción

Docker ayuda con el embalaje, la distribución y la ejecución de aplicaciones. Sin embargo, debemos ser responsables para que podamos garantizar la seguridad del entorno Docker. Hay algunas mejores prácticas y estrategias que podemos seguir para lograr este objetivo.

## Las mejores prácticas y estrategias de seguridad

### Sin información confidencial en archivos

DockerFiles se utilizan para definir los pasos necesarios para construir imágenes de Docker. Es crucial evitar incluir información confidencial como credenciales, claves API u otros secretos directamente en Dockerfiles. Tenga en cuenta que estos archivos a menudo están controlados por versión y pueden ser accesibles para usuarios no autorizados, lo que resulta en un riesgo de seguridad significativo.

Considere usar variables de entorno o archivos de configuración externos para inyectar información confidencial en sus contenedores Docker en tiempo de ejecución.

### Implementar el control de acceso basado en roles (RBAC)

El control de acceso basado en roles (RBAC) es un principio de seguridad que restringe el acceso al sistema en función de los roles y privilegios asignados a usuarios o grupos. En el contexto de Docker, RBAC puede limitar las capacidades de los contenedores y evitar el acceso no autorizado a recursos sensibles.

Considere implementar mecanismos RBAC como:

- Utilizando las características de control de acceso de roles nativo de Docker (RBAC) para definir los permisos granulares para usuarios y servicios.
- Restringir las capacidades de contenedores utilizando perfiles de seguridad de Docker (SECCOMP, APPARMOR o SELINUX) para minimizar el impacto potencial de las violaciones de seguridad.
- Revisar y actualizar regularmente los controles de acceso para garantizar que se alineen con el principio de menor privilegio y reflejen los cambios en su entorno.

### Use variables y archivos de entorno

Las variables de entorno son mecanismo para pasar la información de configuración a los contenedores Docker. Nos permiten desacoplar la configuración de la aplicación desde la imagen del contenedor, por lo que podemos personalizar el comportamiento en diferentes entornos sin modificar la imagen subyacente.

Al usar variables de entorno, necesitamos seguir las mejores prácticas de seguridad:

- Evite la información confidencial de codificación dura directamente en DockerFiles o el código fuente.
- Use herramientas de gestión de secretos cifrados para almacenar y recuperar de forma segura datos confidenciales.
- Limite el acceso a las variables de entorno que contienen información confidencial a usuarios y servicios autorizados.

Podemos usar el soporte incorporado de Docker para archivos de entorno (`--env-file`) para cargar variables de entorno de archivos externos.

`` `` ``
###
Contenido de my_env_vars.env
###

Db_host = localhost
Db_user = admin
Db_password = SecretPassword
`` `` ``

Dockerfile consumiendo el archivo

`` `` ``
# Dockerfile

De Ubuntu: Último
COPIAR . /aplicación

# Cargar variables de entorno desde el archivo
Env_file = my_env_vars.env
Ejecutar set -o allexport; fuente $ env_file; establecer +o Allexport

CMD ["./App"]
`` `` ``

Para ejecutar construir y ejecutar

`` `` ``
# Construye la imagen de Docker
Docker Build -t MyApp.

# Ejecute el contenedor Docker con variables de entorno desde el archivo
Docker Run --env-File my_env_vars.env myapp
`` `` ``

## [Vuelve a BluePrint] (/Study-Devops-Benerint)