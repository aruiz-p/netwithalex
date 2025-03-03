---
author: Alex
category:
  - AI
  - SD-WAN
date: "2024-07-28T14:56:45+00:00"
tag:
  - genai
  - Langchain
  - langgraph
  - openai
  - llm
title: Mejora de mi asistente de SD-WAN - Múltiples agentes
summary: Aprende cómo utilizar un enfoque "agentic" para hacer troubleshooting de tu red SD-WAN con LLMs
url: /mejorando-mi-asistente-sd-wan-multiples-agentes/

---
## Introducción

En mi [Última publicación](/creando-mi-primer-asistente-ai-con-langchain/), creé un asistente de IA de Cisco SD-WAN para ayudarme a ejecutar trazas NWPI y solucionar problemas en la red. La interacción con el asistente requería que el usuario respondiera preguntas hasta obtener información sobre un flujo particular y posibles problemas. En esta publicación, mi objetivo es usar múltiples agentes y ver si puedo llegar a la misma conclusión con menos interacción humana. Se puede encontrar el repositorio [aquí](https://github.com/aruiz-p/sdwan-langgraph)

## Primeros Pasos

Para lograr esto, usaré [LangGraph](https://langchain-ai.github.io/langgraph/) oficialmente definido como:

> Una biblioteca para construir aplicaciones de actores múltiples estatales con LLMS, utilizada para crear flujos de trabajo de agentes y múltiples agentes

Hay [diferentes enfoques](https://langchain-ai.github.io/langgraph/tutorials/#multi-agent-systems), pero decidí construir una estructura donde hay un supervisor que orquesta el flujo de trabajo y decide quién debe actuar a continuación. La idea es construir un gráfico que represente a los agentes y cómo están conectados. El gráfico ilustra el orden en el que los agentes pueden ejecutarse.

En mi caso tengo 3 agentes:

1. **Supervisor** \- Este agente está a cargo de recibir _input_ del usuario y decidir quién debe actuar a continuación. Además, una vez que otros agentes terminen sus tareas, le informarán y se tomará una nueva decisión de enrutamiento. El supervisor es el único agente que puede decidir cuándo volver al usuario con una respuesta.
1. **Reviewer** \- Este agente revisará la información que se enviará al usuario, realiza algunas preguntas o resúmenes y resuelve preguntas o situaciones que el _tracer_ pueda plantear.
1. **Tracer** \- Este es el agente que ejecutará las trazas y recuperará la información que el usuario está buscando. Se informará al supervisor cuando se haga o si se debe responder alguna pregunta.

Tuve que modificar un poco el prompt del tracer, para poder obtener un comportamiento que sea mejor para este enfoque. Además, cada agente puede tener sus propias herramientas. Actualmente, el Tracer tiene más herramientas que el resto de los agentes.

Visualizado, el gráfico se ve así:

![](/wp-content/uploads/2024/07/graph.png)

La flecha punteada indica un borde condicional, lo que significa que el _supervisor_ puede decidir cuál debe ser el próximo agente o si terminar es apropiado.

La flecha continua indica el siguiente paso que **debe seguirse** Por ejemplo, después de "START", el siguiente agente debe ser el _supervisor_. _Tracer_ y _Reviewer_ deben ir al _Supervisor_.

Aunque este es un gráfico simple, este enfoque es muy poderoso.

### ¿Cómo decide el supervisor?

El supervisor juega un papel fundamental, ya que determina quién debe actuar a continuación. Esto se define en la siguiente [función](https://platform.openai.com/docs/guides/function-calling):

```
options = ["FINISH"] + members
function_def = {
    "name": "route",
    "description": "Select the next role.",
    "parameters": {
        "title": "routeSchema",
        "type": "object",
        "properties": {
            "next": {
                "title": "Next",
                "anyOf": [
                    {"enum": options},
                ],
            }
        },
        "required": ["next"],
    },
}
```

Con esto, cada vez que el supervisor reciba cualquier input, obtendrá un valor de "next" de las opciones disponibles y esto representará el siguiente nodo en el gráfico.

## Demo

Usaremos la siguiente topología para probar

![](/wp-content/uploads/2024/07/topology-2.png)

Le doy información al asistente sobre el problema e informo que ya hay tráfico en la red (se requiere tráfico para que NWPI genere los _insights_ que estamos buscando).

![](/wp-content/uploads/2024/07/QUERY.PNG)

Esto es con lo que regresó el asistente

![](/wp-content/uploads/2024/07/agent-resp.png)

La respuesta presenta información de manera condensada, que indica el flujo y el camino que el tráfico está tomando y algunos eventos detectados para esta comunicación. Los detalles de uno de los flujos también están presentes, sin embargo, tiene menos información que antes. Esto se debe a que he pedido al agente _reviewer_ que mantenga lo que considera más relevante y lo envíe al usuario. Al final, podemos ver que se menciona el _Drop Report_ y hay una sugerencia para revisar las ACL 🎉

Podemos jugar con el `reviewer` para mostrar más información sobre los detalles del flujo.

## Detrás de cámaras

Ok, el asistente regresó con una buena respuesta, pero veamos con más detalle lo que sucedió. Usando LangSmith podemos obtener detalles e _insights_ sobre el workflow. Aquí está el proceso completo.

![](/wp-content/uploads/2024/07/agent-workflow.png)

Primero, el _supervisor_ recibe la consulta del usuario y la pasa al _Tracer_.

![](/wp-content/uploads/2024/07/sup1.png)

A continuación, el _Tracer_ utiliza las herramientas disponibles para iniciar la traza, espera capturar algunos flujos y recupera información. Informa al _Supervisor_. Ten en cuenta que el orden en que se ejecutan las herramientas depende del agente.

![](/wp-content/uploads/2024/07/tracer.png)

Luego, el _supervisor_ recibe la información y decide que el _reviewer_ debería actuar a continuación.

![](/wp-content/uploads/2024/07/sup2.png)

A continuación, el _reviewer_ recibe la información y reescribe lo que se recibió del _tracer_. Informa al _supervisor_.

![](/wp-content/uploads/2024/07/rev1.png)

Finalmenete, el _supervisor_ decide que la información está lista para enviarse al usuario. Esto es cuando recibimos el mensaje en Webex.

![](/wp-content/uploads/2024/07/sup3.png)

## Lecciones aprendidas

- Dado que los agentes pueden tomar decisiones, no siempre es fácil entender lo que están haciendo o por qué devuelven lo que devuelven, usando LangSmith definitivamente ayudó con esto. No solo podemos ver el request y las herramientas utilizadas, sino que también hay algunos metadatos que proporcionan información valiosa adicional.
- Llegué a algunas situaciones en las que el supervisor llamaba al tracer varias veces debido a algún error al  recuperar la información. Al final, esto fue causado por un error en el código y, afortunadamente, mi asistente no es costoso. Sin embargo, si tu caso de uso consume muchos tokens, debes considerar agregar algún tipo de protección para evitar un loop que aumente el consumo de APIs.
- Hablando de costos, los modelos de lenguaje pequeño (SLM) son una buena alternativa. Después de quedarme sin cuota, recordé que el modelo GPT-4o-mini está disponible y decidí intentarlo. Después de algunas pruebas, vi que funcionó muy bien y era mucho más barato, así que continué con ese.

## Conclusión

Usando la estrategia de múltiples agentes, podemos lograr tareas más complejas y tener más flexibilidad. Si es necesario, la interacción del usuario se puede agregar en ciertas decisiones que son importantes. Además, existe cierta complejidad adicional, ya que prompts deben refinarse para lograr los resultados que esperamos. En mi caso, tuve que hacer múltiples iteraciones y refinamientos a los prompts de todos los agentes antes de obtener un resultado que consideraba lo suficientemente bueno. 

Me interesa probar otros enfoques para las implementaciones de múltiples agentes y agregar información adicional para que los agentes proporcionen información más precisa a través de RAG.