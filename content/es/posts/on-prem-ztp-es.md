---
author: Alex
category:
  - sdwan
draft: false
date: "2025-05-05T14:20:31+00:00"
title: "Cisco SD-WAN ZTP On-Prem: Automatizando el onboarding de los routers"
description: Plug and Play (PnP) te permite agregar dispositivos SD-WAN automáticamente a través de la nube de Cisco. Esta publicación explica cómo lograr Zero-Touch Provisioning (ZTP) en un entorno aislado y local (on-premises).  
summary: Plug and Play (PnP) te permite agregar dispositivos SD-WAN automáticamente a través de la nube de Cisco. Esta publicación explica cómo lograr Zero-Touch Provisioning (ZTP) en un entorno aislado y local (on-premises). 
url: /ztp-on-prem-es/
tag:
  - ZTP
  - onboarding
---
## Introducción

Cuando implementas un nuevo sitio remoto, quieres que el proceso sea lo más sencillo posible. No quieres enviar el dispositivo a una ubicación intermedia solo para precargarle una configuración, y luego volver a enviarlo a su destino final. Tampoco quieres tener que viajar al sitio, conectar un cable de consola y configurar manualmente cada dispositivo. Ahora imagina multiplicar ese esfuerzo por docenas o cientos de sitios 🤯.

Para resolver este desafío, Cisco creó un proceso de incorporación automatizado llamado Plug and Play (PnP) o Zero Touch Provisioning (ZTP). La idea es simple: un router se enciende, obtiene una dirección IP, localiza su overlay de SD-WAN, se conecta a los controladores y descarga su configuración—todo esto sin intervención humana más allá de conectarlo a la corriente y la red.

Un paso clave en este proceso es ayudar al router a descubrir el overlay de SD-WAN. Esto se puede lograr a través de la nube de Cisco o, en entornos aislados (air-gapped), alojando tu propio servidor ZTP On-Prem. Hoy vamos a explorar esta segunda opción.

## Funcionalidad PnP/ZTP 

El proceso en la imagen describe lo que hace un router SD-WAN al iniciar sin configuración:

![](/wp-content/uploads/2025/05/ztp5.png)

**Nota** este proceso está disponible únicamente en **plataformas físicas**. 

El dominio ZTP está definido en el servidor DHCP, por ejemplo, si el nombre de dominio es **_cisco.com_**, el router intentará resolver ztp.**_cisco.com_**. Para más información puedes consultar la siguiente [documentación de Cisco](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/sdwan-xe-gs-book/cisco-sd-wan-overlay-network-bringup.html#c_Start_the_Enterprise_ZTP_Server_7841.xml)

## Configuración

Necesitamos completar las siguientes tareas para utilizar ZTP on prem

1. Agregar y configurar el servidor ZTP 
2. Cargar la lista de equipo al servidor ZTP 
3. Preparar la configuración del equipo en el Manager
4. Configurar un servidor DHCP y DNS
5. Lanzar el proceso de PnP

###  Agregar y configurar el servidor ZTP 

El servidor ZTP utiliza la misma imagen que un Validator regular. Sigue el proceso habitual para levantar una nueva máquina virtual con conectividad hacia el Manager.

La siguiente es la configuración mínima necesaria:

```
vbond# show run system
system
 host-name               ztp-server
 system-ip               10.10.10.194
 site-id                 5
 sp-organization-name    SDWAN-LAB123
 organization-name       SDWAN-LAB123
 vbond 192.168.200.2 local ztp-server 

vpn 0
 interface eth0
  ip dhcp-client
  ipv6 dhcp-client
  no shutdown
 !
 interface ge0/0
  ip address 192.168.200.3/24
  no shutdown
 !
 ip route 0.0.0.0/0 192.168.200.253
!
 ```

Nota el sufijo **_ztp-server_**, esto indicará al dispositivo que actuará como servidor ZTP.

Desde el Manager, agrega el servidor ZTP a la lista de Controladores:

![](/wp-content/uploads/2025/05/ztp1.png)

Dependiendo del método de autenticación del Controlador, genera y firma el CSR.

Si estás utilizando certificados Enterprise, necesitarás instalar el certificado raíz y el certificado firmado.

Por ejemplo:

```
ztp-server# request root-cert-chain install home/admin/root-ca.crt
ztp-server# request certificate install home/admin/ztp.crt
```
**Nota**: El servidor ZTP no tiene ninguna conexión de control con el Manager ni con ningún otro controlador.

### Cargar la lista de equipo al servidor ZTP 

Ahora que el servidor ZTP está instalado, es necesario cargar la lista de equipo que se conectarán a él.

La manera más sencilla es obtener el archivo _serialFile.viptela_ desde el portal PnP y copiarlo localmente en el equipo.

```
ztp-server:~$ ls -l | grep serial
-rw-r--r-- 1 admin admin  2364 Apr 24 21:41 serialFile.viptela
```
Luego es necesario ejecutar el siguiente comando

```
ztp-server# request device-upload chassis-file home/admin/serialFile.viptela 
Uploading chassis numbers via VPN 0
Copying ... /home/admin/serialFile.viptela via VPN 0
file:  /tmp/tmp.CbUWf8GnSN/viptela_serial_file
PnP
Verifying public key received from PnP against production root cert
is_public_key_ok against production root ca:  OK
Signature verified for viptela_serial_file
final file:  /tmp/tmp.CbUWf8GnSN/viptela_serial_file
Signature verification Suceeded.
Success: Serial file is /tmp/tmp.CbUWf8GnSN/viptela_serial_file
INFO: Input File specified was '/usr/share/viptela/chassis_numbers.tmp'
INFO: # of complete chassis entries written: 12
Json to CSV conversion succeeded!
Successfully loaded the chassis numbers file to the database.
```
Para verificar que la lista se cargó correctamente

```
ztp-server# show ztp entries 
                                                                                                                                                  ROOT     
                                                                                                                           VBOND  ORGANIZATION    CERT     
INDEX  CHASSIS NUMBER                                   SERIAL NUMBER                             VALIDITY  VBOND IP       PORT   NAME            PATH     
-----------------------------------------------------------------------------------------------------------------------------------------------------------
...
23     ASR1001-HX-XXXXXXXXXXX                           XXXXXXXX                                  valid     192.168.200.1  12346  SDWAN-LAB123  default   
```
### Preparar la configuración del equipo en el Manager

Para logar esto, utilizaré un _Configuration Group_, pero se puede logar con _Templates_ igualmente

![](/wp-content/uploads/2025/05/ztp2.png)

Creo la configuración y la envío al equipo. Como está offline, la terea queda como "Scheduled"

![](/wp-content/uploads/2025/05/ztp3.png)

### Configurar un servidor DHCP y DNS

Para hacer sencillo, configuro un switch intermedio como servidor DHCP y DNS con la siguiente configuración:

```
ip dhcp pool ASR
 vrf MPLS
 network 192.168.11.4 255.255.255.252
 default-router 192.168.11.6 
 dns-server 192.168.11.6 
 domain-name cisco.com

ip host vrf MPLS ztp.cisco.com 192.168.200.3
ip host vrf MPLS devicehelper.cisco.com 192.168.200.3
```
Ok, estamos listos para lanzar el proceso PnP

### Lanzar el proceso de PnP

Para activar el proceso de PnP, el dispositivo debe tener una configuración en blanco. Voy a usar el siguiente comando para restablecer la configuración y activar el proceso.

```
Router#request platform software sdwan config reset 

%WARNING: Bootstrap file doesn't exist and absence 
of it can cause loss of connectivity to the controller.
For saving bootstrap config, use:
request platform software sdwan bootstrap-config save
Proceed to reset anyway? [confirm]

Backup of running config is saved under /bootflash/sdwan/backup.cfg
Config reset requested from a console session.
Waiting for up to 60 seconds for IOS to initiate reload or report failure.
IOS return status: "cfgreset_proceed"
Config reset is raised successfully, device will reload shortly.
```

Para ver los siguientes logs es necesario acceso por consola: 

El equipo arranca y comienza el proceso de PnP

```
*May  4 04:18:42.659: %PNP-6-PNP_DISCOVERY_STARTED: PnP Discovery started
```
El equipo obtiene una dirección ip, router por defecto y un nombre de dominio

```
Autoinstall trying DHCPv4 on GigabitEthernet0/0/0,GigabitEthernet0/0/1,GigabitEthernet0/0/2,GigabitEthernet0
...
*May  4 04:19:44.999: %PKI-2-NON_AUTHORITATIVE_CLOCK: PKI functions can not be initialized until an authoritative time source, like NTP, can be obtained.
Acquired IPv4 address 192.168.11.5 on Interface GigabitEthernet0/0/1
Received following DHCPv4 options:
        domain-name     : cisco.com
        dns-server-ip   : 192.168.11.6

```

El equipo intenta resolver los dominos y es redirigido a _ztp.cisco.com_
```
*May  4 04:20:08.694: %PNP-3-PNP_CCO_SERVER_IP_UNRESOLVED: CCO server (devicehelper.cisco.com.) can't be resolved (1/5) by (pid=619, pname=PnP Agent Discovery, time=04:20:08 UTC Sun May 4 2025)
...
*May  4 04:20:20.696: %IOSXE_SDWAN_CONFIG-5-PNP_REDIRECT: PnP Redirect Msg: Org name "" Host "ztp.cisco.com." port 0 intf GigabitEthernet0/0/1
*May  4 04:20:42.010: %PNP-6-PNP_REDIRECTION_DONE: PnP Redirection done (1) by (pid=619, pname=PnP Agent Discovery)
*May  4 04:20:42.010: %PNP-6-PNP_SDWAN_STARTED: PnP SDWAN started (1) via (pnp-sdwan-vbond-ztp-discovery) by (pid=619, pname=PnP Agent Discovery)
*May  4 04:20:42.811: %PNP-6-PNP_DISCOVERY_DONE: PnP Discovery done successfully (PnP-VBOND-ONPREM-ZTP-IPV4) profile (pnp-zero-touch) 
```
Podemos confirmar que la resolución DNA para _ztp.cisco.com_ ocurrió de manera correctamente
```
ASR1K-2#show pnp trace | i  ztp
[05/04/25 04:20:10.695 UTC B7 619] 1: VBOND_ONPRIME_ZTP hostname ztp.cisco.com. resolved to 192.168.11.6 on interface GigabitEthernet0/0/1
[05/04/25 04:20:10.695 UTC B8 619] host_name is ztp.cisco.com. vbond_ipv4_address is 192.168.11.6, interface is GigabitEthernet0/0/1
```
Enseguida, el eouter se conecta al servidor ZTP y es redirigido al Validator
```
*May  4 04:21:17.991: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:17 control-connection-state-change severity-level:major host-name:"Router" system-ip::: personality:vedge peer-type:vbond peer-system-ip::: peer-vmanage-system-ip:0.0.0.0 public-ip:192.168.200.3 public-port:12346 src-color:default remote-color:default uptime:"0:00:00:00" new-state:up
...
*May  4 04:21:19.242: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:19 org-name-change severity-level:minor host-name:"Router" system-ip::: old-organization-name:"" new-organization-name:"SDWAN-LAB123"
*May  4 04:21:21.597: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:21 control-connection-state-change severity-level:major host-name:"Router" system-ip::: personality:vedge peer-type:vbond peer-system-ip::: peer-vmanage-system-ip:0.0.0.0 public-ip:192.168.200.1 public-port:12346 src-color:default remote-color:default uptime:"0:00:00:00" new-state:up
```
Eventualmente, el equipo se conecta al MAnager y al Controller y obtiene su configuración
```
*May  4 04:21:23.924: %Cisco-SDWAN-Router-vdaemon-6-INFO-1400002: Notification: 5/4/2025 4:21:23 control-connection-state-change severity-level:major host-name:"Router" system-ip::: personality:vedge peer-type:vmanage peer-system-ip:10.10.10.2 peer-vmanage-system-ip:0.0.0.0 public-ip:192.168.100.1 public-port:12746 src-color:default remote-color:biz-internet uptime:"0:00:00:00" new-state:up
*May  4 04:21:24.299: %Cisco-SDWAN-CSS-SDWAN-POD1-ASR1K-2-OMPD-5-NTCE-400003: Operational state changed to UP
*May  4 04:21:43.981: %DMI-5-AUTH_PASSED: R0/0: dmiauthd: User 'vmanage-admin' authenticated successfully from 10.10.10.2:42962  for netconf over ssh.
```
![](/wp-content/uploads/2025/05/ztp4.png)

```
ASR1K-2#show sdwan control connections | i up
vsmart  dtls 10.10.10.3      5          1      192.168.100.2                           12346 192.168.100.2                           12346 SDWAN-LAB123          mpls            No    up     0:02:02:51 0           
vbond   dtls 0.0.0.0         0          0      192.168.200.1                           12346 192.168.200.1                           12346 SDWAN-LAB123          mpls            -     up     0:02:02:54 0           
vmanage dtls 10.10.10.2      5          0      192.168.100.1                           12946 192.168.100.1                           12946 SDWAN-LAB123          mpls            No    up     0:02:02:49 0           
```

## Conclusion

El servidor ZTP On-Prem amplía la capacidad de incorporar dispositivos físicos en aquellas redes donde el acceso a internet está restringido.

Automatizar la incorporación de routers con ZTP On-Prem es una excelente alternativa para las organizaciones que necesitan tener control total sobre su proceso de aprovisionamiento o que operan en entornos aislados (air-gapped). Al replicar localmente el proceso de Plug and Play, eliminas la necesidad de configuraciones manuales en sitios remotos mientras mantienes aislados los entornos sensibles.

Con la configuración adecuada, incorporar nuevos sitios se convierte en un proceso sencillo y repetible que ahorra tiempo valioso, reduce errores humanos y escala las implementaciones de SD-WAN.

👉 ¿Te interesa configurar ZTP On-Prem para tu red? Déjame un comentario o contáctame; ¡me encantaría ayudarte o responder tus preguntas!

