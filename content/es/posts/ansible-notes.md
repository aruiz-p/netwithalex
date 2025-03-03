---
author: Alex
category:
  - devnet
  - material de estudio
draft: true
date: "2024-05-19T13:49:53+00:00"
title: Ansible Notes
url: /ansible-notas/

---
### Introducción

[_Tis Post es parte de Devnet DevOps (300-910)-BluePrint _] (/Study-DevOps-BluePrint/)

Ansible es una herramienta de automatización de código abierto utilizada para la administración de configuración, implementación de aplicaciones y automatización de tareas. Aunque se puede utilizar para muchos propósitos, estamos particularmente interesados ​​en las tareas relacionadas con la automatización de redes.

Hay muchos recursos para aprender Ansible en línea, estas notas deberían servir solo como orientación, pero se le recomienda que profundice a su propio ritmo.

### Características clave

- ** Sin agente: ** Ansible no requiere que los agentes se instalen en nodos administrados, lo que lo hace liviano y fácil de implementar.
-** Basado en YAML: ** Los libros de jugadas y los archivos de configuración se escriben en YAML, legible por humanos y fáciles de entender.
- ** Idempotent: ** Ansible asegura que se mantenga el estado deseado del sistema, lo que hace que sea seguro ejecutar varias veces.

### Componentes

- ** Nodo de control **: la máquina donde se instala Ansible y desde la cual se inicia la automatización.
- ** Nodos administrados **: Las máquinas administradas por Ansible. Ansible se conecta a nodos administrados a través de SSH (Linux) o WinRM (Windows) para ejecutar tareas.
- ** Inventario **: Una lista de nodos administrados que Ansible administrará.
- ** Playbooks **: archivos YAML que contienen un conjunto de tareas que se ejecutarán en nodos administrados.
- ** Módulos **: Unidades de código que Ansible se ejecuta en nodos administrados para realizar tareas.

### Requisitos para usar Ansible

- ** Requisitos de nodo de control **
  - Ansible requiere que se instale Python
  - se puede instalar con yum, apt o pip
  - Más comúnmente instalado en Linux o macOS

### archivo de inventario

- Archivo de texto que contiene una lista de nodos administrados al que Ansible puede conectarse y administrar en este caso nuestros dispositivos de red.
- puede estar en formato ini o yaml
- Puede agrupar nodos y definir variables. Ver múltiples ejemplos [aquí] (https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)

### Playbook

Los archivos YAML que definen un conjunto de tareas que se ejecutarán en los nodos administrados. Son el corazón de la automatización Ansible.

#### Estructura

- ** Hosts **: Especifica los hosts o grupos de destino (inventario)
- ** Tareas **: contiene una lista de tareas que se ejecutarán en nodos administrados.
- ** Variables **: Define las variables utilizadas dentro del libro de jugadas.

Los libros de jugadas usan módulos para interactuar con los modos administrados. Recomiendo familiarizarse con algunos [módulos Cisco] (https://docs.ansible.com/ansible/latest/collections/cisco/index.html)

### Ejemplos prácticos

Puede encontrar algunos ejemplos prácticos en el siguiente repositorio [https://github.com/aruiz-p/netwithalex-ansible/tree/main).

### [Volver a BluePrint] (/Study-Devops-BanePrint/)