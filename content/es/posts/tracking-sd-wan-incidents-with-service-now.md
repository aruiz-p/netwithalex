---
author: Alex
category:
  - automatización
  - SD-WAN
date: "2024-03-15T18:06:34+00:00"
title: Seguimiento de incidentes SD-WAN con ServiceNow
summary: Aprenda a integrar Cisco SD-WAN con ServiceNow y automatiza la gestión de incidentes
url: /rastreando-incidentes-de-sdwan-con-servicenow/

---
### Introducción

A medida que las redes evolucionan para ofrecer una mejor experiencia de usuario y se introducen nuevas tecnologías para gestionar la red, mantener todo funcionando sin problemas se ha vuelto cada vez más difícil. Una de las responsabilidades más críticas del equipo de operaciones es hacer un seguimiento de los problemas que ocurren en toda la red. Identificarlos es solo el comienzo, luego deben ser registrados y gestionados hasta su resolución. ¡Multiplica la cantidad de acciones por incidente y tendrás suficiente para mantener ocupado a tu equipo de TI todo el día!

En este post, te mostraré lo que necesitas saber para integrar el SD-WAN Manager con ServiceNow para la gestión de incidentes. Veremos algunos de los problemas más comunes en SD-WAN.

### Configuración de laboratorio

Estoy utilizando la versión 20.12.1 de SD-WAN Manager y tengo una instancia de desarrollador de ServiceNow. Mi servidor de Webhook funciona en Ubuntu 20.04 LTS y construí el receptor de Webhook en lenguaje Go.

Para simplificar las cosas, tengo comunicación directa entre todos los elementos de mi laboratorio.

![](/wp-content/uploads/2024/03/snow-1.png)

### Webhooks

Los _Webhooks_ son una manera en que las aplicaciones web se comunican entre sí en tiempo real. Permiten que una aplicación envíe notificaciones automáticas a otra aplicación cuando ocurre un evento específico, lo que se conoce como el `modelo push`. Esto facilita la integración entre diferentes sistemas y puede usarse para activar actividades automatizadas posteriores. Los Webhooks típicamente utilizan callbacks HTTP para compartir notificaciones/información.

En nuestro escenario, el SD-WAN Manager monitoreará eventos en BR10 y enviará solicitudes HTTP POST a nuestro servidor de Webhook cuando ocurran eventos específicos. Esto nos permitirá gestionar los incidentes en ServiceNow.

#### Anatomía de una notificación webhook

Comprendamos la estructura y la información que el SD-WAN Manager compartirá con nuestro servidor Webhook.

Este es un ejemplo de la información enviada cuando alguna interfaz va **abajo**

```
{
  "suppressed": false,
  "devices": [
    {
      "system-ip": "1.1.10.1"
    }
  ],
  "eventname": "interface-state-change",
  "type": "interface-state-change",
  "rulename": "interface-state-change",
  "component": "VPN",
  "entry_time": 1709277345253,
  "statcycletime": 1709277345253,
  "message": "The interface oper-state changed to down",
  "severity": "Critical",
  "severity_number": 1,
  "uuid": "9e2f7630-d504-4cdf-b808-fc8e29a6dd47",
  "values": [
    {
      "host-name": "BR10",
      "system-ip": "1.1.10.1",
      "if-name": "GigabitEthernet2",
      "new-state": "down",
      "vpn-id": "0"
    }
  ],
  "rule_name_display": "Interface_State_Change",
  "receive_time": 1708843127894,
  "values_short_display": [
    {
      "host-name": "BR10",
      "system-ip": "1.1.10.1",
      "if-name": "GigabitEthernet2",
      "new-state": "down"
    }
  ],
  "system_ip": "1.1.10.1",
  "host_name": "BR10",
  "acknowledged": false,
  "active": true
}
```

Veamos la información más importante para nosotros:

- ` "active": true ` **-** ¿Tenemos un problema? Sí, indicado por el estado activo
- ` "message": "The interface oper..." ` **-** ¿Qué está pasando?
- ` "severity_number": 1  ` **-** ¿Qué tan malo es? (escogí el numero y no el string a propósito)
- ` "uuid": "9e2f7630-d504...d47" ` **-** Identificador del evento usado por el SD-WAN Manager
- ` "system_ip": "1.1.10.1" ` **-** ¿Qué equipo originó la alerta?
- ` "host_name": "BR10" ` **-** Identificador de equipo más amigable para los humanos

Veamos la notificación cuando la interfaz se va **arriba**

```
{
  "suppressed": false,
  "devices": [
    {
      "system-ip": "1.1.10.1"
    }
  ],
  "eventname": "interface-state-change",
  "type": "interface-state-change",
  "rulename": "interface-state-change",
  "component": "VPN",
  "entry_time": 1709277482508,
  "statcycletime": 1709277482508,
  "message": "The interface oper-state changed to up",
  "severity": "Medium",
  "severity_number": 3,
  "uuid": "5486325c-d189-4467-9b5a-16acb1f28ec9",
  "values": [
    {
      "host-name": "BR10",
      "system-ip": "1.1.10.1",
      "if-name": "GigabitEthernet2",
      "new-state": "up",
      "vpn-id": "0"
    }
  ],
  "rule_name_display": "Interface_State_Change",
  "receive_time": 1708843265147,
  "values_short_display": [
    {
      "host-name": "BR10",
      "system-ip": "1.1.10.1",
      "if-name": "GigabitEthernet2",
      "new-state": "up"
    }
  ],
  "system_ip": "1.1.10.1",
  "host_name": "BR10",
  "acknowledged": false,
  "cleared_events": [
    "9e2f7630-d504-4cdf-b808-fc8e29a6dd47"
  ],
  "active": false
}
```

Las dos cosas más importantes son:

- ` "active": false ` **-**  ya no está activo o presente
- ` "cleared_events": ["9e2f7630-d504...d47]" ` **-** ID del evento que se resuelve

Una cosa que debe saber es que no todos los eventos se comportarán de la misma manera. Algunos de ellos no tendrán una entrada de `cleared_events`, por lo que tendríamos que manejarlos de manera diferente si queremos cerrarlos automáticamente. Otros pueden carecer de cierta información dependiendo de lo que estemos monitoreando.

Antes de codificar cualquier tipo de aplicación, es importante saber lo que deseas monitorear y lo que obtendrás del SD-WAN Manager para que puedas manejarlo correctamente.

### Configuración de Webhooks en SD-WAN Manager

El 20.12, es muy simple configurarlos, primero habilita la configuración de notificación desde _`Administration > Settings > Alarm Notifications`_

![](/wp-content/uploads/2024/03/webhook-2.png)

Definamos las reglas para activar nuestros webhooks. Desde _`Monitor > Logs > Alarm notification > Add Alarm Notification`_

![](/wp-content/uploads/2024/03/webhook-4.png)

Cosas que tienes que saber:

- Puedes elegir monitorear sitios o dispositivos.
- La severidad es crucial. Para abrir incidentes, generalmente querrás monitorear incidentes de severidad crítica o alta, y para cerrarlos necesitarás monitorear severidades más bajas.
- Las alarmas para las que deseas generar webhooks, en nuestro caso:
    - BFD nodo fuera de servicio/activo
    - Omp nodo fuera de servicio/activo
    - Nodo de control fuera de servicio/activo
    - Interfaz fuera de servicio/activa
- Ten en cuenta que solo estoy usando HTTP en mi URL de webhook, se recomienda usar HTTPS para mayor seguridad. El `8080:/webhook` proviene de la aplicación que construimos.
- El _threshold_ limitará la cantidad de notificaciones enviadas por minuto a esta URL. 15 es suficiente para mí, pero probablemente no lo sea para un entorno de producción.
Por último, el usuario y la contraseña en caso de que tu aplicación de webhook lo requiera. Estos valores se codificarían y se enviarían en los encabezados. Usa valores ficticios si no son necesarios.

### Construyendo el servidor webhook

 El código que uso esta publicado en el siguiente [Repo de Github](https://github.com/aruiz-p/sdwan-snow-integración), aquí lo explicaré de una manera más simple.

**Paso 1:** Las solicitudes se esperan en el puerto `8080` al endpoint `/webhook`.

```
// Listen upcoming requests

http.HandleFunc("/webhook", handleWebhook)
fmt.Println("Server listening on port 8080...")
if err := http.ListenAndServe("0.0.0.0:8080", nil); err != nil {
		fmt.Printf("Failed to start server: %v", err)
}
```

**Paso 2.** Comprobar si la solicitudes tiene un estado activo. En caso afirmativo, creo un incidente en ServiceNow.

```
//Verify if issue is active and create incident

if data["active"] == true {
	fmt.Println("Opening Service Now incident...")
	err := createIncident(data)
```

**Paso 2.1** Para abrir un incidente en ServiceNow, necesitamos reformatear la información que se enviará allí.

```
func createIncident(data map[string]interface{}) error {

        // Retrieve information to open incident
	issueId := data["uuid"].(string)
	ruleName := data["rule_name_display"].(string)
	title := data["message"].(string)
	severity := data["severity_number"].(float64)
	severityStr := strconv.FormatFloat(severity, 'f', -1, 64)
	device := ". Device " + data["host_name"].(string) +
		", System-ip " + data["system_ip"].(string)

	// Construct JSON payload for creating incident in SNOW
	incidentData := map[string]interface{}{
		"category":          "network",
		"caller_id":         "vManage",
		"short_description": issueId,
		"description":       ruleName + " - " + title + device,
		"urgency":           severityStr,
		"impact":            severityStr,
	}
...
```

Observa que almaceno el `uuid` proveniente de vManage y lo usamos como `short_description` para ServiceNow.

**Paso 3.a** Si el estado **no es** activo, verificamos si hay algún `cleared_events` incluido, esto nos dará una alta precisión al cerrar incidentes.

```
if _, ok := data["cleared_events"]; ok {
...
	clearedEvents, ok := data["cleared_events"].([]interface{})
        eventId := clearedEvents[0].(string)
        incidentExists, incident_id, err := getIncidentWithId(eventId)
        ...
        if incidentExists {
		err := closeIncident(incident_id)
```

Para cerrar un caso, necesitamos tener el identificador de ServiceNow llamado _**sys\_id**_. Para obtenerlo, usamos la función `getIncidentWithId`.

```
func getIncidentWithId(issueId string) (bool, string, error) {
...
        // Store Service Now "short_description"
        shortDescription, ok := incidentMap["short_description"].(string)

        // Compare the short_description with the issueId
	if strings.Contains(shortDescription, issueId) {
	        // If the short_description matches the issueId, return the incident id
		sys_id, ok := incidentMap["sys_id"].(string)
			if !ok {
				continue
			}
	return true, sys_id, nil
```

**Paso 3.b** Si el estado **no es** activo y no hay `_cleared events_` incluidos, vamos a intentar encontrar el incidente que se abrió. Hay tres cosas que verificamos:

- **Rule Name** **-** Los nombres de reglas para los eventos que estamos monitoreando tendrán la siguiente estructura <feature>\_node\_<state>. Por ejemplo, `BFD_Node_Down`. Si estamos viendo BFD\_Node\_Up, cambiaremos "Up" por "Down" y buscaremos _BFD\_Node\_Down_ en los incidentes devueltos desde ServiceNow.
- **System Ip** **-** Almacene el `System-IP` contenido en la notificación y coincida con la descripción de cada incidente devuelto.
- **Time** **-** Almaceno el `opened_at` y verificamos si tiene menos de 12 horas. Esta es una medida totalmente subjetiva, pero mi idea es que los problemas que tardan más de 12 horas en resolverse, tendrían que ser verificados por algún humano.

```
func getIncidentWoutId(ruleName, sysIp string, openTime float64) (bool, string, error) {
...
      // IncidentMap holds the incidents from Service Now
      description := incidentMap["description"].(string)
      snowTime := incidentMap["opened_at"].(string)
      newRuleName := strings.Replace(ruleName, "Up", "Down", -1)

      // Compare Rule Name, system ip and time
      if strings.Contains(description, newRuleName) &&
	      strings.Contains(description, sysIp) &&
	      diffHours < 12 {
	      sys_id, ok := incidentMap["sys_id"].(string)
              sys_id, ok := incidentMap["sys_id"].(string)

              if !ok {
	              continue
	      }
	      return true, sys_id, nil
```

**Paso 4.** Cierra el caso con el sys\_id obtenido a través de la función `getIncidentWoutId`.

```
if incidentExists {
	err := closeIncident(incident_id)
	if err != nil {
		fmt.Printf("Error closing incident: %v\n", err)
		// Handle the error accordingly (e.g., log it, return, etc.)
		return
	}
} else {
	fmt.Printf("Incident doesn't exist or is older than 12 hours")
}
```

### Demo

Comenzaremos ejecutando la aplicación en Go. Nota que no estoy usando VS Code para ejecutarla (Ctrl + F5), sino la terminal para permitir las conexiones entrantes.

![](/wp-content/uploads/2024/03/webhook-3.png)

#### Notificación de interfaz

Apagamos una de las interfaces de servicio en el router:

```
BR10-1#config-transaction
BR10-1(config)# interface GigabitEthernet 2
BR10-1(config-if)# shutdown
BR10-1(config-if)# commit
Commit complete.
```

El servidor recibe la notificación y abre el incidente. El número de incidente es el identificador ServiceNow (sys_id)

![](/wp-content/uploads/2024/03/webhook-5-1.png)

![](/wp-content/uploads/2024/03/webhook-6.png)

Traigo la interfaz arriba y el incidente se cierra

![](/WP-Content/uploads/2024/03/Webhook-7-1.png)

![](/wp-content/uploads/2024/03/webhook-9.png)

Observe que esta notificación tiene la información de `cleared_events`, por lo que es muy fácil encontrar ese incidente en ServiceNow. Además, la severidad es 'Medium', por eso es importante establecer el valor correcto en la configuración de _Alarm Notifications_.

Observa que _**State**_ es `Resolved` y _**Resolution Notes**_ indica que el incidente se cerró automáticamente a través de Webhooks.

#### Notificaciones de BFD y Control Connections 

Bajemos las sesiones de control y BFD cerrando la apagando la interfaz de transporte

```
BR10-1(config)# interface GigabitEthernet 1
BR10-1(config-if)# sh
BR10-1(config-if)# commit
Commit complete.
```

Se reciben notificaciones y se crean incidentes

![](/wp-content/uploads/2024/03/Webhook-10.png)

![](/wp-content/uploads/2024/03/webhook-11.png)

Cuando volvemos a habilitar la interfaz, recibimos las notificaciones y también registramos el estado de la interfaz para la interfaz Gig 1 y Tunnel1. Todo esto se refleja en ServiceNow. Estas notificaciones de interfaz no fueron entregadas antes porque se perdió la conectividad con el SD-WAN Manager.

![](/wp-content/uploads/2024/03/webhook-12-1.png)

![](/wp-content/uploads/2024/03/webhook-1.png)

### Lecciones aprendidas

1. Es muy importante monitorear el nivel de severidad correcto, de lo contrario podríamos perder notificaciones necesarias para cerrar los incidentes adecuadamente.
2. El SD-WAN Manager puede generar muchas alertas, por lo que el umbral del webhook se vuelve muy importante. Deberías probar para encontrar el número que funcione para tu entorno.
3. ServiceNow asignará una **prioridad** en función de la urgencia e impacto utilizados para crear el ticket.
4. ServiceNow puede tener políticas que te impidan cerrar incidentes si cierta información no está presente en la interfaz. Familiarízate un poco con las políticas en la interfaz gráfica y Data Policies.
5. Aunque puedes establecer manualmente el _"sys_id"_ en ServiceNow mediante APIs, sugiero dejarlo como está, ya que poner valores manuales podría causar problemas en el futuro, y este campo debe ser único en tu instancia. Te recomiendo usar el valor generado automáticamente.
6. Puedes usar sitios públicos [como este](https://webhook.site) para ver fácilmente el contenido de las notificaciones mientras planificas tus casos de uso.

### Conclusión 
Los webhooks son una excelente manera de monitorear nuestro entorno. Al ser notificado exactamente cuándo ocurre un problema en lugar de confiar en encuestas continuas, aumente nuestra capacidad de iniciar sesión y reaccionar rápidamente a lo que esté sucediendo. Puede combinar webhooks con otro tipo de alertas, como correos electrónicos o incluso chat (webex, holgura, etc.) en caso de que necesite alertar a diferentes equipos. Espero que esta publicación te haya dado algunas ideas o active tu curiosidad.

¡Gracias por leerme!