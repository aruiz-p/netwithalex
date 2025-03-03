---
author: alex
category:
  - ai
  - sd-wan
date: "2024-07-28T14:56:45+00:00"
guid: https://netwithalex.blog/?p=749
tag:
  - genai
  - langchain
  - langgraph
  - llm
  - openai
  - slm
title: Improving my SD-WAN Assistant - Multiple agents
summary: Learn how an LLM agentic approach can be used to troubleshoot your SD-WAN network
url: /improving-my-sd-wan-assistant-multiple-agents/

---
## Introduction

In my [last post](/building-my-first-sd-wan-ai-assistant-with-langchain/), I created a Cisco SD-WAN assistant to help me run NWPI traces and troubleshoot the network. The interaction with the assistant required the user to answer questions until receiving information about a particular flow and potential problems. In this post my goal is to use multiple agents and see if I can get to the same conclusion with less human interaction. Repository can be found [here](https://github.com/aruiz-p/sdwan-langgraph)

## Getting Started

To achieve this, I will use [LangGraph](https://langchain-ai.github.io/langgraph/) officially defined as:

>Una librería basada en LangChain que permite construir aplicaciones multiagente con estados persistentes

There are [different approaches](https://langchain-ai.github.io/langgraph/tutorials/#multi-agent-systems), but I decided to build a structure where there is a supervisor that orchestrates the workflow and decides who should act next. The idea is to build a graph that represents the agents and how they are connected. The graph illustrates the order in which agents can be executed.



In my case I have 3 agents:

1. **Supervisor** \- This agent is in charge of receiving user input and deciding who should act next. Also, once others agents finish their tasks, they will report back to it and a new routing decision will be made. The supervisor is the only agent that can decide when to go back to the user with a response.
1. **Reviewer** \- This agent will review the information that will be sent back to the user, perform some summarization and resolve questions or situations the tracer might raise.
1. **Tracer** \- This is the agent that will run the traces and retrieve the information that the user is looking for. It will report back to the supervisor when done or if any questions need to be answered.

I had to modify the prompt of the tracer a little bit, so I could get a behavior that is better for this approach. Also, each agent can have its own tools. Currently, the tracer has more tools than the rest of agents.

Visualized, the graph looks like this:

![](/wp-content/uploads/2024/07/graph.png)

The dotted arrow indicate a conditional edge, meaning that the supervisor can decide what the next agent should be or if ending is appropriate.

The continuous arrow indicate the next step that **must** be taken. For example, after "start" the next agent must be the supervisor. Tracer and Reviewer must go to the supervisor.

Even though this is a simple graph, this supervisor approach is very powerful.

### How the supervisor routes?

The supervisor plays a critical role as it determines who should act next. This is defined in the following [function](https://platform.openai.com/docs/guides/function-calling):

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

With this, every time the supervisor receives any input, it will come up with a "next" value from the available options and this will represent the next node on the graph.

## Demo

We will use the following topology to test

![](/wp-content/uploads/2024/07/Topology-2.png)

I tell the assistant information about the issue and inform that there is traffic passing already (traffic is required in order for NWPI to generate the insights we are looking for).

![](/wp-content/uploads/2024/07/query.png)

This is what the Assistant came back with

![](/wp-content/uploads/2024/07/agent-resp.png)

The response presents information in a condensed way, indicating the hops and path the traffic is taking and some events detected for this communication. The details of one of the flows is also present, however is has less information than before. This is because I have asked the `reviewer ` agent to keep what it considers more relevant and send it to the user. In the end, we can see that the DROP\_REPORT is mentioned and there is a suggestion to review ACLs 🎉

We can play with the `reviewer` prompt to display more information about the details of the flow.

## Behind the scenes

Ok, the assistant came back with a good response, but let's see in more detail what happened under the hood.

Using LangSmith we can get details and insights about the workflow followed. Here is the complete process.

![](/wp-content/uploads/2024/07/agent-workflow.png)

First, the `supervisor` receives the user's query and pass it to the `tracer`.

![](/wp-content/uploads/2024/07/sup1.png)

Next, the `tracer` uses the available tools to starts the trace, waits to capture some flows and retrieves information. Reports to the `supervisor`. Note that the order in which tools are executed is up to the agent.

![](/wp-content/uploads/2024/07/Tracer.png)

Then, the `supervisor` receives the information and decides the `reviewer` should act next.

![](/wp-content/uploads/2024/07/sup2.png)

Next, the `reviewer` receives the information and rewrites what was received from the tracer. Goes back to the `supervisor`.

![](/wp-content/uploads/2024/07/rev1.png)

The `supervisor` decides the information is ready to be sent to the user. This is when we receive the message back on Webex.

![](/wp-content/uploads/2024/07/Sup3.png)

## Lessons Learned

- Since agents can make decisions, it is not always easy to understand what they are doing or why they return what they return, using LangSmith definitely helped with this. Not only we can see the order and tools used, but also there is some metadata that provides additional valuable information.
- I got to some situations where the supervisor was calling the tracer multiple times due to some error retrieving information. In the end, this was caused by error on the code and, fortunately, my assistant isn't expensive. However, if your use case consumes a lot of tokens you should consider add some sort of safeguard to prevent a loop that grows your API consumption.
- Talking about cost, small language models are a good alternative. After running out of quota, I remembered gpt-4o-mini model is out and decided to give it a try. After some testing I saw it performed very well and was much cheaper, so I stuck to it.

## Conclusion

Using multi-agent deployment we can achieve more complex tasks and have more flexibility. If needed, user interaction can be added on certain decisions that are important. Also, there is some added complexity as the prompts have to be refined to achieve the results we expect. In my case I had to do multiple iterations and refinements to all agents' prompts before getting an output that I considered to be good enough. I am interested in testing other approaches to multi-agent deployments and adding additional info for agents to provide more accurate information through RAG.
