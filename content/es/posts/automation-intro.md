---
author: Alex
category:
  - sdwan
  - automation
draft: false
date: "2025-06-16T00:00:00+00:00"
title: "Primeros pasos con la automatización en SD-WAN"
description: Aprende los fundamentos de la automatización en SD-WAN y cómo comenzar a usar scripts y APIs para optimizar tus operaciones de red.
summary: Aprende los fundamentos de la automatización en SD-WAN y cómo comenzar a usar scripts y APIs para optimizar tus operaciones de red.
url: /automation-intro-es/
tag:
  - API
---
## Introducción

En esta publicación, exploraremos cómo dar los primeros pasos con la automatización en SD-WAN. Ya sea que estés comenzando con los scripts o apenas empieces a descubrir lo que es posible hacer con la API de SD-WAN, esta guía te ayudará a empezar. Cubriremos conceptos clave, ejemplos prácticos y las herramientas que necesitas para automatizar tareas y simplificar tus operaciones de red.

## ¿Por qué deberías automatizar?
Hay muchos casos donde la automatización puede aportar beneficios significativos. Ya sea para aplicar cambios más rápido, minimizar riesgos, ahorrar tiempo, construir herramientas personalizadas o integrar con sistemas de terceros — la automatización abre la puerta a una gestión de red más simple y eficiente.

Aquí algunos casos de uso comunes:

- **Despliegue de dispositivos** – Automatiza el proceso de despliegue de VMs y su configuración inicial usando plantillas predefinidas.  
- **Gestión de configuraciones** – Aplica configuraciones mediante la API en segundos, en lugar de iniciar sesión manualmente en cada dispositivo.  
- **Monitoreo y alertas** – Recibe alertas cuando ocurran eventos específicos en la red y crea scripts para monitorear exactamente lo que necesitas.  
- **Integración con otros sistemas** – Enriquece plataformas de Cisco o externas con información compartida directamente desde el SD-WAN Manager.  
- **Verificaciones de salud** – Verifica de forma continua que tu red esté funcionando correctamente y bajo el estado deseado.  
- **Ampliación de capacidades** – Experimenta con nuevas tecnologías y explora cómo integrarlas a tus tareas diarias.


## APIs REST de SD-WAN Manager
En el centro de la automatización están las APIs REST (Interfaz de Programación de Aplicaciones basada en Transferencia de Estado Representacional), que permiten que los sistemas se comuniquen entre sí a través de HTTP — el mismo protocolo que usa tu navegador para cargar páginas web.

La API REST del SD-WAN Manager te permite interactuar programáticamente con tu controlador SD-WAN. En lugar de hacer clic en la interfaz gráfica, puedes enviar solicitudes HTTP para:

- **GET** obtener información (por ejemplo, listar dispositivos, obtener estadísticas de salud)  
- **POST** crear nuevas configuraciones  
- **PUT** actualizar objetos existentes  
- **DELETE** eliminar configuraciones que ya no necesitas  

Cada solicitud se dirige a una URL específica e incluye encabezados, autenticación y datos.

Puedes encontrar la documentación de los endpoints de Cisco [aquí](https://developer.cisco.com/docs/search/?products=Catalyst%20SD-WAN).

También puedes acceder a la **documentación de la API dentro del producto** en:

> _https://vmanage-ip:port/apidocs_

![](/wp-content/uploads/2025/06/api1.png)

## Códigos de estado HTTP

Los códigos de estado HTTP te indican el resultado de tu solicitud a la API. Aquí están los más comunes:

- **200 OK** – La solicitud fue exitosa.  
- **301 Moved Permanently** – El recurso tiene una nueva URL.  
- **400 Bad Request** – La solicitud está mal formada.  
- **401 Unauthorized** – Necesitas iniciar sesión.  
- **403 Forbidden** – No tienes permiso para acceder a este recurso.  
- **404 Not Found** – El recurso solicitado no existe.  
- **500 Internal Server Error** – Algo falló en el servidor.  
- **503 Service Unavailable** – El servidor está sobrecargado o fuera de servicio temporalmente.

Esperamos recibir un **200 OK** al realizar llamadas a la API.

## Autenticación

### Generar una ID de sesión (cookie)

Antes de interactuar con la API del SD-WAN Manager, necesitas autenticarte — tal como lo harías al iniciar sesión en la interfaz gráfica. Para ello, se debe usar el siguiente endpoint:

```
POST https://{vmanage-ip-address}/j_security_check
Content-Type: application/x-www-form-urlencoded
HTTP Body: "j_username={admin}&j_password={credential}"
```

Se trata de una operación **POST**, que requiere el encabezado _Content-Type: application/x-www-form-urlencoded_ y espera un cuerpo con el siguiente formato:

_j_username=**{admin}**&j_password=**{credential}**_.

Vamos a probarlo con [curl](https://curl.se/). Ten en cuenta que cuando usamos la opción _**--data**_, curl entiende que debe usar el método POST.

```
alex ~ % curl -v -k --location 'https://10.1.1.1:443/j_security_check' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'j_username=netwithalex' \
--data-urlencode 'j_password=netwithalex'
*   Trying 10.1.1.1:443...
* Connected to 10.1.1.1 (10.1.1.1) port 443
...
* using HTTP/1.x
> POST /j_security_check HTTP/1.1
> Host: 10.1.1.1
> User-Agent: curl/8.7.1
> Accept: */*
> Content-Type: application/x-www-form-urlencoded
... 
* upload completely sent off: 45 bytes
< HTTP/1.1 200 OK
< set-cookie: JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH.b387d75e92857dc1e41025f809f43b054a0d0aa8ebf50be8d2fc1aa2f87c90da; path=/; secure; HttpOnly
...
< 
* Connection #0 to host 10.1.1.1 left intact
```
Debemos guardar el valor de _set-cookie_
> JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH 

y utilizarlo a continuación para generar un **token**.

### Obtener un token

Un token XSRF (también llamado CSRF o Cross-Site Request Forgery token) es una medida de seguridad que ayuda a proteger una aplicación web contra acciones no autorizadas realizadas por atacantes.

Se utiliza el siguiente endpoint para obtener un token:

```
GET https://{vmanage-ip-address}/dataservice/client/token
Content-Type: application/json
HTTP Header: "Cookie: JESSIONID={session hash id}"
```
Este **token es requerido principalmente para operaciones POST**, si estás realizando operaciones GET probablemente no lo necesitarás. Ten esto en cuenta al construir scripts que requieran acciones POST.

```
alex ~ % curl -v -k --location 'https://10.1.1.1:443/dataservice/client/token' \
--header 'Content-Type: application/json' \
--header 'Cookie: JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH.b387d75e92857dc1e41025f809f43b054a0d0aa8ebf50be8d2fc1aa2f87c90da'
*   Trying 10.1.1.1:443...
* Connected to 10.1.1.1 (10.1.1.1) port 443
...
* using HTTP/1.x
> GET /dataservice/client/token HTTP/1.1
> Host: 10.1.1.1
> Accept: */*
> Content-Type: application/json
> Cookie: JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH.b387d75e92857dc1e41025f809f43b054a0d0aa8ebf50be8d2fc1aa2f87c90da
< HTTP/1.1 200 OK
...
< vary: Accept-Encoding
< content-type: application/json; charset=UTF-8
>
* Connection #0 to host 10.1.1.1 left intact
F1FE79CE079240E12A393B73F0EB5445F31BC456322F634EDEF8A9A7D5856A6FC7763D2C80D2253118917910502F20A9A1CA% 
```

El token se muestra al final:

> F1FE79CE079240E12A393B73F0EB5445F31BC456322F634EDEF8A9A7D5856A6FC7763D2C80D2253118917910502F20A9A1CA

Con esto, estás listo para interactuar con el SD-WAN Manager 

## Ejemplos
### GET - Obtener la configuración de un dispositivo

Vamos a obtener la configuración de un dispositivo con la siguiente llamada

```
GET https://{{vmanage-ip}}:{{port}}/dataservice/template/config/running/{deviceId}
Content-Type: application/json
```
En este caso,e el _deviceId_ es el chassis number, se puede encontrar en la UI:

![](/wp-content/uploads/2025/06/api2.png)

```
alex ~ % curl -k -v --location 'https://10.1.1.1:8443/dataservice/template/config/running/C8K-C0302D69-35A9-4E85-8909-2031A2165FE8' \
--header 'Content-Type: application/json' \                 
--header 'Cookie: JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH.b387d75e92857dc1e41025f809f43b054a0d0aa8ebf50be8d2fc1aa2f87c90da'
*   Trying 10.1.1.1:8443...
* Connected to 10.1.1.1 (10.1.1.1) port 8443
...
* using HTTP/1.x
> GET /dataservice/template/config/running/C8K-C0302D69-35A9-4E85-8909-2031A2165FE8 HTTP/1.1
> Host: 10.1.1.1:8443
> User-Agent: curl/8.7.1
> Accept: */*
> Content-Type: application/json
> Cookie: JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH.b387d75e92857dc1e41025f809f43b054a0d0aa8ebf50be8d2fc1aa2f87c90da
> 
...
< vary: Accept-Encoding
< content-type: application/json; charset=UTF-8
< server: svcproxy
< transfer-encoding: chunked
< 
{"config":" system\n system-ip             1.1.200.1\n overlay-id            1\n site-id               200\n no transport-gateway enable\n port-offset           0\n control-session-pps   300\n admin-tech-on-failure\n sp-organization-name  MYSDWAN-LAB123\n organization-name     MYSDWAN-LAB123\n port-hop\n track-transport\n track-default-gateway\n upgrade-confirm...}
```
La configuración se regresa dentro de las {} 

### POST - Crear un nuevo usuario

Vamos a hacer una operación POST sencilla para ver cómo debemos pasar el token.

Usaremos el siguiente endpoint

```
POST https://{{vmanage-ip}}:{{port}}/dataservice/admin/user
Headers: "Content-Type: application/json", "accept: */*"
Payload = {
   "group":[
      "operator"
   ],
   "description":"Demo User",
   "userName":"apiUser",
   "password":"apiuser",
}
```

Necesitamos especificar el grupo, la descripción, el nombre de usuario y la contraseña.

Hagámoslo con curl

```
alex ~ % curl -v -k --location 'https://10.1.1.1:8443/dataservice/admin/user' \
--header 'Content-Type: application/json' \
--header 'X-XSRF-TOKEN: F1FE79CE079240E12A393B73F0EB5445F31BC456322F634EDEF8A9A7D5856A6FC7763D2C80D2253118917910502F20A9A1CA' \
--header 'Cookie: JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH.b387d75e92857dc1e41025f809f43b054a0d0aa8ebf50be8d2fc1aa2f87c90da' \
--data '{
   "group":[
      "operator"
   ],
   "description":"API User",
   "userName":"apiuser",
   "password":"apiuser"
}'
*   Trying 10.1.1.1:8443...
* Connected to 10.1.1.1 (10.1.1.1) port 8443
...
> POST /dataservice/admin/user HTTP/1.1
> Host: 10.1.1.1:8443
> User-Agent: curl/8.7.1
> Accept: */*
> Content-Type: application/json
> X-XSRF-TOKEN: F1FE79CE079240E12A393B73F0EB5445F31BC456322F634EDEF8A9A7D5856A6FC7763D2C80D2253118917910502F20A9A1CA
> Cookie: JSESSIONID=dEzBwsDlx-eMKye1xbXBgT7NZVfXRe1tJQZvkMLH.b387d75e92857dc1e41025f809f43b054a0d0aa8ebf50be8d2fc1aa2f87c90da
> Content-Length: 117
> 
...
* upload completely sent off: 117 bytes
< HTTP/1.1 200 OK
...
< vary: Accept-Encoding
< vmanagerequestid: 45dc1c76-8a00-48e4-80f0-fe9b2f2a894b
< x-content-type-options: nosniff
< strict-transport-security: max-age=31536000; includeSubDomains
< permissions-policy: geolocation=(self), microphone=(), camera=(), fullscreen=(self)
< content-type: application/octet-stream; charset=UTF-8
< server: svcproxy
< transfer-encoding: chunked
< 
* Connection #0 to host 10.1.1.1 left intact
{}%
```
Recibimos un **HTTP 200** y una respuesta vacía `{}`, lo que indica que el usuario fue creado correctamente. Vamos a verificar en la interfaz gráfica (UI).

![](/wp-content/uploads/2025/06/api3.png)

El usuario _apiuser_ se creó correctamente!

## Conclusión

En este post, cubrimos los conceptos básicos que necesitas para comenzar a trabajar con las REST APIs del SD-WAN Manager — desde entender los códigos de estado HTTP hasta recuperar configuraciones y crear usuarios. Los ejemplos usaron *curl* intencionalmente para que puedas ver la estructura de las solicitudes, entender los encabezados y aprender cómo trabajar con cookies y el token XSRF.

En la siguiente parte, traduciremos estos conceptos a Python y comenzaremos a automatizar tareas del mundo real paso a paso.
