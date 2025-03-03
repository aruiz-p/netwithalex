---
author: Alex
category:
  - devnet
  - estudio
date: "2024-04-06T08:47:03+00:00"
title: Dockerfile
URL: /dockerfile-es/
draft: true

---
### Introducción

Docker puede construir imágenes automáticamente leyendo las instrucciones de un DockerFile. Un DockerFile es un documento de texto que contiene todos los comandos que un usuario podría llamar a la línea de comandos para ensamblar una imagen. Esta página describe los comandos que puede usar en un DockerFile.

Docker ejecuta instrucciones en un Dockerfile en orden. Un Dockerfile ** debe comenzar con una instrucción **.

### Ejemplo

`` `` ``
# Use la imagen oficial de Python como imagen base
De Python: 3.9-Slim

# Establezca el directorio de trabajo dentro del contenedor
WorkDir /App

# Copie el script de Python de la máquina host en el contenedor
Copiar my_script.py.

# Ejecute el script de Python cuando comience el contenedor
CMD ["Python", "my_script.py"]
`` `` ``

- `De Python: 3.9-Slim`: Especifica que nuestra imagen se basará en la imagen oficial de Python con la versión 3.9, utilizando la variante delgada.
- `WorkDir /App`: Establece el directorio de trabajo dentro del contenedor en ` /App`. Las instrucciones posteriores se ejecutarán en este directorio.
- `Copiar my_script.py. `: Copia un archivo llamado` my_script.py` de la máquina host en el directorio `/App` del contenedor. El '.' Se refiere al directorio actual en el contexto de Docker, que generalmente será el directorio que contiene DockerFile.
- `cmd [" python "," my_script.py "]`: Especifica el comando predeterminado para ejecutarse cuando se inicia un contenedor basado en esta imagen. En este caso, ejecuta el intérprete de Python (`Python`) con el script 'my_script.py` como argumento.

### ** Tabla de instrucciones **

DE
Envidia
COPIAR
EXPONER
AGREGAR
Huella de trabajo
CORRER
ETIQUETA
CMD

### Instrucciones en detalle

**DE**

Especifica la imagen base para construir el contenedor. Cada dockerfile debe comenzar con una instrucción desde la instrucción.

`` `` ``
Desde [--platform = <platforma>] <Image> [: <tag>] [como <name>]
`` `` ``

**AGREGAR**

Especifica el origen de un archivo o URL local y transfiere los archivos asociados al directorio de destino designado. Si la fuente está comprimida, como un archivo .tar, la instrucción Agregar extraerá automáticamente el contenido después de que finalice la operación de copia.

`` `` ``
Agregar [Opciones] <Src> ... <est>
`` `` ``

**CORRER**

Ejecuta comandos de shell dentro de la imagen durante el proceso de compilación. Esto generalmente se usa para tareas como la instalación de paquetes, la configuración del entorno, etc. Los comandos comunes incluyen 'instalación adecuada' y 'Instalación de PIP' para los paquetes de Python.

`` `` ``
Ejecutar [Opciones] <Command> ...
`` `` ``

** Workdir **

Establece el directorio de trabajo para cualquier ejecución posterior, CMD, EntryPoint, copia y agregue instrucciones. Es mejor usar rutas absolutas en lugar de relativas (../)

`` `` ``
WorkDir/Path/To/Workdir
`` `` ``

**EXPONER**

Funciona como un tipo de instrucción de documentación que puede usar para indicar los puertos que se utilizarán para el contenedor. El protocolo predeterminado es TCP, pero también puede definir UDP.

`` `` ``
Exponer <port> [<port>/<protocol> ...]
`` `` ``

** CMD **

Especifica el comando para ejecutar cuando se inicia un contenedor desde la imagen. Se puede anular en tiempo de ejecución. Solo puedes usarlo una vez.

`` `` ``
Comando cmd param1 param2
`` `` ``

** Entrypoint **

Establece un comando principal que siempre se ejecutará cuando se inicie el contenedor.

`` `` ``
Comando EntryPoint Param1 Param2
`` `` ``

** Env **

Establece variables de entorno en la imagen.

`` `` ``
Env <Key> = <valor> ...
`` `` ``

**VOLUMEN**

Crea un punto de montaje y lo marca como montado externamente. Utilizado para persistir datos fuera del contenedor.

`` `` ``
Volumen ["/Data"]
`` `` ``

**USUARIO**

Establece el nombre de usuario o UID para usar al ejecutar el contenedor.

`` `` ``
Usuario <serve> [: <grupo>]
`` `` ``

### Referencias adicionales

[Documentación oficial de Dockerfile] (https://docs.docker.com/reference/dockerfile/#user) \- Consulte esta página para obtener más detalles de los comandos.

### [Regrese a BluePrint] (/Study-Devops-Benerint)