[![Exploring AI Agent Frameworks](./images/lesson-2-thumbnail.png)](https://youtu.be/ODwF-EZo_O8?si=1xoy_B9RNQfrYdF7)

> _(Click the image above to view video of this lesson)_

# Explore AI Agent Frameworks

AI agent frameworks are software platforms designed to simplify the creation, deployment, and management of AI agents. These frameworks provide developers with pre-built components, abstractions, and tools that streamline the development of complex AI systems.

These frameworks help developers focus on the unique aspects of their applications by providing standardized approaches to common challenges in AI agent development. They enhance scalability, accessibility, and efficiency in building AI systems.

## Introduction

This lesson will cover:

- What are AI Agent Frameworks and what do they enable developers to achieve?
- How can teams use these to quickly prototype, iterate, and improve their agent’s capabilities?
- What are the differences between the frameworks and tools created by Microsoft (<a href="https://aka.ms/ai-agents-beginners/ai-agent-service" target="_blank">Azure AI Agent Service</a> and the <a href="https://learn.microsoft.com/azure/ai-services/openai/how-to/responses" target="_blank">Microsoft Agent Framework</a>)?
- Can I integrate my existing Azure ecosystem tools directly, or do I need standalone solutions?
- What is Azure AI Agents service and how is this helping me?

## Learning goals

The goals of this lesson are to help you understand:

- The role of AI Agent Frameworks in AI development.
- How to leverage AI Agent Frameworks to build intelligent agents.
- Key capabilities enabled by AI Agent Frameworks.
- The differences between the Microsoft Agent Framework and Azure AI Agent Service.

## What are AI Agent Frameworks and what do they enable developers to do?

Traditional AI Frameworks can help you integrate AI into your apps and make these apps better in the following ways:

- **Personalization**: AI can analyze user behavior and preferences to provide personalized recommendations, content, and experiences.
Example: Streaming services like Netflix use AI to suggest movies and shows based on viewing history, enhancing user engagement and satisfaction.
- **Automation and Efficiency**: AI can automate repetitive tasks, streamline workflows, and improve operational efficiency.
Example: Customer service apps use AI-powered chatbots to handle common inquiries, reducing response times and freeing up human agents for more complex issues.
- **Enhanced User Experience**: AI can improve the overall user experience by providing intelligent features such as voice recognition, natural language processing, and predictive text.
Example: Virtual assistants like Siri and Google Assistant use AI to understand and respond to voice commands, making it easier for users to interact with their devices.

### That all sounds great right, so why do we need the AI Agent Framework?

AI Agent frameworks represent something more than just AI frameworks. They are designed to enable the creation of intelligent agents that can interact with users, other agents, and the environment to achieve specific goals. These agents can exhibit autonomous behavior, make decisions, and adapt to changing conditions. Let's look at some key capabilities enabled by AI Agent Frameworks:

- **Agent Collaboration and Coordination**: Enable the creation of multiple AI agents that can work together, communicate, and coordinate to solve complex tasks.
- **Task Automation and Management**: Provide mechanisms for automating multi-step workflows, task delegation, and dynamic task management among agents.
- **Contextual Understanding and Adaptation**: Equip agents with the ability to understand context, adapt to changing environments, and make decisions based on real-time information.

So in summary, agents allow you to do more, to take automation to the next level, to create more intelligent systems that can adapt and learn from their environment.

## How to quickly prototype, iterate, and improve the agent’s capabilities?

This is a fast-moving landscape, but there are some things that are common across most AI Agent Frameworks that can help you quickly prototype and iterate namely module components, collaborative tools, and real-time learning. Let's dive into these:

- **Use Modular Components**: AI SDKs offer pre-built components such as AI and Memory connectors, function calling using natural language or code plugins, prompt templates, and more.
- **Leverage Collaborative Tools**: Design agents with specific roles and tasks, enabling them to test and refine collaborative workflows.
- **Learn in Real-Time**: Implement feedback loops where agents learn from interactions and adjust their behavior dynamically.

### Use Modular Components

SDKs like the Microsoft Agent Framework offer pre-built components such as chat clients, tool definitions, typed agents, and workflow builders.

**How teams can use these**: Teams can quickly assemble these components to create a functional prototype without starting from scratch, allowing for rapid experimentation and iteration.

**How it works in practice**: You can use a typed `Agent` primitive together with a `FoundryChatClient` (or any other chat client that implements `SupportsChatGetResponse`), register tool functions with simple Python type annotations, and let the framework handle the tool-dispatch loop and conversation history for you — without having to build these components from scratch.

**Example code**. Let's look at an example of how you can use the Microsoft Agent Framework with `FoundryChatClient` and the `Agent` primitive to have the model respond to user input with tool calling:

```python
# Microsoft Agent Framework Python Example

import asyncio
import os
from typing import Annotated

from pydantic import Field
from dotenv import load_dotenv

from agent_framework import Agent, tool
from agent_framework.foundry import FoundryChatClient
from azure.identity.aio import AzureCliCredential


@tool(name="book_flight", description="Book a flight for a given date and destination.")
def book_flight(
    date: Annotated[str, Field(description="Travel date in ISO format, e.g. 2025-01-01.")],
    location: Annotated[str, Field(description="Destination city or airport.")],
) -> str:
    """Book travel given location and date."""
    # Replace with your real booking integration.
    return f"Travel was booked to {location} on {date}"


async def main() -> None:
    load_dotenv()

    async with AzureCliCredential() as credential:
        client = FoundryChatClient(
            project_endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
            model=os.environ["FOUNDRY_MODEL"],
            credential=credential,
        )

        agent = Agent(
            client=client,
            name="travel_agent",
            instructions=(
                "You help the user book travel. "
                "Call the book_flight tool once you have a date and destination."
            ),
            tools=[book_flight],
        )

        # Non-streaming
        result = await agent.run("I'd like to go to New York on January 1, 2025")
        print(result)

        # Streaming alternative:
        # async for chunk in agent.run("...", stream=True):
        #     if chunk.text:
        #         print(chunk.text, end="", flush=True)


if __name__ == "__main__":
    asyncio.run(main())
```

What you can see from this example is how you can leverage the `@tool` decorator together with `Annotated` and Pydantic `Field` descriptions to expose a clean JSON schema to the model, so the agent can extract key information from user input (such as the date and destination of a flight booking request) and call the tool with the right arguments. This modular approach allows you to focus on the high-level logic.

### Leverage Collaborative Tools

Frameworks like the Microsoft Agent Framework facilitate the creation of multiple agents that can work together through first-class orchestration primitives such as `SequentialBuilder` (pipeline), `ConcurrentBuilder` (fan-out/fan-in), and `WorkflowBuilder` (custom graphs).

**How teams can use these**: Teams can design agents with specific roles and tasks, then wire them into a workflow that handles message passing, context, and intermediate outputs — enabling rapid testing and refinement of collaborative patterns and improving overall system efficiency.

**How it works in practice**: You can create a team of agents where each agent has a specialized function, such as data retrieval, analysis, or decision-making. A `SequentialBuilder` connects them in a pipeline; each agent receives the prior conversation (or only the prior agent's response, if you set `chain_only_agent_responses=True`) and contributes its turn toward a shared goal.

**Example code (Microsoft Agent Framework)**:

```python
# Creating multiple agents that work together using the Microsoft Agent Framework

import asyncio
import os
from typing import Annotated

from pydantic import Field
from dotenv import load_dotenv

from agent_framework import Agent, AgentResponse, SequentialBuilder, tool
from agent_framework.foundry import FoundryChatClient
from azure.identity.aio import AzureCliCredential


# --- Tools ----------------------------------------------------------------

@tool(name="retrieve_sales", description="Retrieve sales records for a given period.")
def retrieve_tool(
    period: Annotated[str, Field(description="Reporting period, e.g. 'Q4 2024'.")],
) -> str:
    """Stub — replace with your real data-fetch (SAP, AuditBoard, SQL, etc.)."""
    return (
        f"Sales for {period}: "
        "North 1.2M, South 0.9M, East 1.4M, West 0.7M. "
        "YoY growth: +8%. Top product: SKU-447."
    )


@tool(name="run_analysis", description="Run descriptive analytics on a dataset summary.")
def analyze_tool(
    data: Annotated[str, Field(description="Raw or summarized data to analyze.")],
) -> str:
    """Stub — replace with your real analytics (pandas, Benford, outliers, etc.)."""
    return f"Analysis of: {data[:80]}... → mean/variance computed, no anomalies."


# --- Workflow -------------------------------------------------------------

async def main() -> None:
    load_dotenv()

    async with AzureCliCredential() as credential:
        # One shared client — both agents use it.
        client = FoundryChatClient(
            project_endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
            model=os.environ["FOUNDRY_MODEL"],
            credential=credential,
        )

        retrieve_agent = Agent(
            client=client,
            name="dataretrieval",
            instructions="Retrieve relevant data using available tools. Return a concise summary.",
            tools=[retrieve_tool],
        )

        analyze_agent = Agent(
            client=client,
            name="dataanalysis",
            instructions="Analyze the retrieved data and provide insights and recommendations.",
            tools=[analyze_tool],
        )

        # Sequential pipeline. By default each agent sees the full prior conversation;
        # use chain_only_agent_responses=True to pass only the prior agent's reply.
        workflow = SequentialBuilder(
            participants=[retrieve_agent, analyze_agent],
        ).build()

        events = await workflow.run("Retrieve sales data for Q4 and analyze it.")
        outputs = events.get_outputs()

        if outputs:
            final: AgentResponse = outputs[0]
            print("===== Final Response =====")
            for msg in final.messages:
                author = msg.author_name or "assistant"
                print(f"[{author}]\n{msg.text}\n")


if __name__ == "__main__":
    asyncio.run(main())
```

What you see in the previous code is how you can compose a task that involves multiple agents working together to analyze data. Each agent performs a specific function, and the `SequentialBuilder` coordinates them — passing messages between participants, preserving author/role metadata, and surfacing the final `AgentResponse` as the workflow output. By creating dedicated agents with specialized roles, you can improve task efficiency and performance.

### Learn in Real-Time

Advanced frameworks provide capabilities for real-time context understanding and adaptation.

**How teams can use these**: Teams can implement feedback loops where agents learn from interactions and adjust their behavior dynamically, leading to continuous improvement and refinement of capabilities.

**How it works in practice**: Agents can analyze user feedback, environmental data, and task outcomes to update their knowledge base, adjust decision-making algorithms, and improve performance over time. This iterative learning process enables agents to adapt to changing conditions and user preferences, enhancing overall system effectiveness.

## What are the differences between the Microsoft Agent Framework and Azure AI Agent Service?

There are many ways to compare these approaches, but let's look at some key differences in terms of their design, capabilities, and target use cases:

## Microsoft Agent Framework (MAF)

The Microsoft Agent Framework provides a streamlined SDK for building AI agents. It exposes typed primitives — `Agent`, `tool`, `SequentialBuilder`, `ConcurrentBuilder`, `WorkflowBuilder` — that wrap the underlying model API, the tool-dispatch loop, and conversation history into composable pieces. Pair it with a chat client such as `FoundryChatClient` (for Microsoft Foundry project endpoints) or `OpenAIChatCompletionClient` (for Azure OpenAI / OpenAI endpoints) to leverage hosted models with built-in tool calling, conversation management, and enterprise-grade security through Azure identity.

**Use Cases**: Building production-ready AI agents with tool use, multi-step workflows, and enterprise integration scenarios.

Here are some important core concepts of the Microsoft Agent Framework:

- **Agents**. An agent is created by constructing an `Agent` with a chat client, a name, instructions, and tools. The agent can:
  - **Process user messages** and generate responses using the configured model.
  - **Call tools** automatically based on the conversation context.
  - **Maintain conversation state** across multiple interactions (via local history or service-managed sessions).

  Here is a code snippet showing how to create an agent:

    ```python
    import os
    from agent_framework import Agent
    from agent_framework.foundry import FoundryChatClient
    from azure.identity.aio import AzureCliCredential

    async with AzureCliCredential() as credential:
        client = FoundryChatClient(
            project_endpoint=os.environ["FOUNDRY_PROJECT_ENDPOINT"],
            model=os.environ["FOUNDRY_MODEL"],
            credential=credential,
        )

        agent = Agent(
            client=client,
            name="my_agent",
            instructions="You are a helpful assistant.",
        )

        response = await agent.run("Hello, World!")
        print(response)
    ```

- **Tools**. The framework supports defining tools as Python functions that the agent can invoke automatically. Use `Annotated` and Pydantic `Field` to give the model rich parameter descriptions, and optionally the `@tool` decorator to set an explicit name and description. Tools are registered when constructing the agent:

    ```python
    from typing import Annotated
    from pydantic import Field
    from agent_framework import Agent, tool

    @tool(name="get_weather", description="Get the current weather for a location.")
    def get_weather(
        location: Annotated[str, Field(description="The city or location to check.")],
    ) -> str:
        return f"The weather in {location} is sunny, 72°F."

    agent = Agent(
        client=client,
        name="weather_agent",
        instructions="Help users check the weather.",
        tools=[get_weather],
    )
    ```

- **Multi-Agent Coordination**. You can create multiple agents with different specializations and coordinate their work using a workflow builder, rather than piping strings between `run()` calls manually:

    ```python
    from agent_framework import Agent, SequentialBuilder

    planner = Agent(
        client=client,
        name="planner",
        instructions="Break down complex tasks into steps.",
    )

    executor = Agent(
        client=client,
        name="executor",
        instructions="Execute the planned steps using available tools.",
        tools=[execute_tool],
    )

    workflow = SequentialBuilder(participants=[planner, executor]).build()
    events = await workflow.run("Plan a trip to Paris and then execute the plan.")
    ```

- **Azure Identity Integration**. The framework uses `AzureCliCredential` (or `DefaultAzureCredential`) for secure, keyless authentication, eliminating the need to manage API keys directly. For production deployments — Container Apps, Azure Functions, AKS — `DefaultAzureCredential` will automatically pick up managed identity.

## Azure AI Agent Service

Azure AI Agent Service (now offered through Microsoft Foundry as the Foundry Agent Service) was introduced at Microsoft Ignite 2024. It allows for the development and deployment of AI agents with more flexible models, such as directly calling open-source LLMs like Llama 3, Mistral, and Cohere.

Azure AI Agent Service provides stronger enterprise security mechanisms and data storage methods, making it suitable for enterprise applications.

It works out-of-the-box with the Microsoft Agent Framework: when the agent definition (instructions, tools, model) lives in the Foundry portal and the service manages the session, you connect to it from your Python code using `FoundryAgent` from `agent_framework.foundry`.

This service is currently in Public Preview and supports Python and C# for building agents.

Using the Microsoft Agent Framework, we can connect to a Foundry-defined agent and run it like any other agent:

```python
import asyncio
import os

from dotenv import load_dotenv
from agent_framework.foundry import FoundryAgent
from azure.identity.aio import DefaultAzureCredential


async def main() -> None:
    load_dotenv()

    async with DefaultAzureCredential() as credential:
        # The agent definition (name, instructions, tools like get_specials and
        # get_item_price) lives in the Foundry portal. Here we just connect to it.
        agent = FoundryAgent(
            agent_name=os.environ["FOUNDRY_AGENT_NAME"],     # e.g. "Host"
            agent_version=os.environ["FOUNDRY_AGENT_VERSION"],  # e.g. "1.0"
            credential=credential,
            allow_preview=True,
        )

        # The service creates and manages the conversation session for you.
        session = await agent.get_session()

        user_inputs = [
            "Hello",
            "What is the special soup?",
            "How much does that cost?",
            "Thank you",
        ]

        for user_input in user_inputs:
            print(f"# User: '{user_input}'")
            response = await agent.run(user_input, session=session)
            print(f"# Agent: {response}")


if __name__ == "__main__":
    asyncio.run(main())
```

### Core concepts

Azure AI Agent Service has the following core concepts:

- **Agent**. Azure AI Agent Service integrates with Microsoft Foundry. Within Foundry, an AI Agent acts as a "smart" microservice that can be used to answer questions (RAG), perform actions, or completely automate workflows. It achieves this by combining the power of generative AI models with tools that allow it to access and interact with real-world data sources. With the Microsoft Agent Framework, you connect to a service-managed agent by name:

    ```python
    from agent_framework.foundry import FoundryAgent
    from azure.identity.aio import DefaultAzureCredential

    agent = FoundryAgent(
        agent_name="my-agent",
        agent_version="1.0",
        credential=DefaultAzureCredential(),
        allow_preview=True,
    )
    ```

    In this example, an agent named `my-agent` (defined in Foundry with its model, instructions, and tools — including hosted tools such as code interpreter, file search, and Bing) is referenced by name. The agent's full configuration lives server-side, and your client code only needs the name and version.

- **Sessions and messages**. A session represents a conversation between an agent and a user. Sessions track the progress of a conversation, store context information, and manage the state of the interaction — and because the Foundry Agent Service manages them server-side, conversations survive across processes and deployments. Here's an example:

    ```python
    # Create a service-managed session for this conversation.
    session = await agent.get_session()

    response = await agent.run(
        "Could you please create a bar chart for the operating profit using the "
        "following data and provide the file to me? "
        "Company A: $1.2 million, Company B: $2.5 million, "
        "Company C: $3.0 million, Company D: $1.8 million",
        session=session,
    )
    print(response)
    ```

    In the previous code, a session is created, then a user message is sent and the agent runs against it. Responses can contain different content types — text, images, or files — depending on what tools the Foundry agent has access to (for example, the code interpreter tool can produce charts as file outputs). As a developer, you can then use this information to further process the response or present it to the user.

- **Integrates with the Microsoft Agent Framework**. Azure AI Agent Service works seamlessly with the Microsoft Agent Framework: use `Agent(client=FoundryChatClient(...))` when your application owns the agent definition (instructions, tools, conversation loop), and use `FoundryAgent` when the agent definition lives in Foundry and the service owns the session.

**Use Cases**: Azure AI Agent Service is designed for enterprise applications that require secure, scalable, and flexible AI agent deployment.

## What's the difference between these approaches?

It does sound like there is overlap, but there are some key differences in terms of their design, capabilities, and target use cases:

- **Microsoft Agent Framework (MAF)**: Is a production-ready SDK for building AI agents in your own application code. It provides typed primitives — `Agent`, `tool`, `SequentialBuilder`, `ConcurrentBuilder`, `WorkflowBuilder` — for creating agents with tool calling, conversation management, multi-agent orchestration, and Azure identity integration.
- **Azure AI Agent Service**: Is a platform and deployment service in Microsoft Foundry for agents. It offers built-in connectivity to services like Azure OpenAI, Azure AI Search, Bing Search, and code execution, and lets you define and version your agent in the Foundry portal.

Still not sure which one to choose?

### Use Cases

Let's see if we can help you by going through some common use cases:

> Q: I'm building production AI agent applications and want to get started quickly
>
> A: The Microsoft Agent Framework is a great choice. It provides a simple, Pythonic API via `Agent(client=FoundryChatClient(...))` that lets you define agents with tools and instructions in just a few lines of code.

> Q: I need enterprise-grade deployment with Azure integrations like Search and code execution
>
> A: Azure AI Agent Service is the best fit. It's a platform service that provides built-in capabilities for multiple models, Azure AI Search, Bing Search, and Azure Functions. It makes it easy to build your agents in the Foundry Portal and deploy them at scale — and connect to them from code via `FoundryAgent`.

> Q: I'm still confused, just give me one option
>
> A: Start with the Microsoft Agent Framework to build your agents (with `Agent` + `FoundryChatClient`), and then use Azure AI Agent Service when you need to deploy and scale them in production (connecting via `FoundryAgent`). This approach lets you iterate quickly on your agent logic while having a clear path to enterprise deployment.

Let's summarize the key differences in a table:

| Framework | Focus | Core Concepts | Use Cases |
| --- | --- | --- | --- |
| Microsoft Agent Framework | Streamlined agent SDK with tool calling and orchestration | `Agent`, `tool`, `SequentialBuilder` / `ConcurrentBuilder` / `WorkflowBuilder`, Chat Clients (`FoundryChatClient`, `OpenAIChatCompletionClient`), Azure Identity | Building AI agents, tool use, multi-step workflows, multi-agent orchestration |
| Azure AI Agent Service | Flexible models, enterprise security, code generation, hosted tools | Service-managed Agents and Sessions, hosted tools (code interpreter, file search, Bing), connected via `FoundryAgent` | Secure, scalable, and flexible AI agent deployment |

## Can I integrate my existing Azure ecosystem tools directly, or do I need standalone solutions?

The answer is yes, you can integrate your existing Azure ecosystem tools directly with Azure AI Agent Service especially, as it has been built to work seamlessly with other Azure services. You could for example integrate Bing, Azure AI Search, and Azure Functions. There's also deep integration with Microsoft Foundry.

The Microsoft Agent Framework also integrates with Azure services through chat clients such as `FoundryChatClient` and `OpenAIChatCompletionClient`, and uses Azure identity (`AzureCliCredential`, `DefaultAzureCredential`, managed identity) for keyless authentication — letting you call Azure services directly from your agent tools.

## Sample Codes

- Python: [Agent Framework](./code_samples/02-python-agent-framework.ipynb)
- .NET: [Agent Framework](./code_samples/02-dotnet-agent-framework.md)

## Got More Questions about AI Agent Frameworks?

Join the [Microsoft Foundry Discord](https://aka.ms/ai-agents/discord) to meet with other learners, attend office hours and get your AI Agents questions answered.

## References

- <a href="https://techcommunity.microsoft.com/blog/azure-ai-services-blog/introducing-azure-ai-agent-service/4298357" target="_blank">Azure Agent Service</a>
- <a href="https://learn.microsoft.com/azure/ai-services/openai/how-to/responses" target="_blank">Microsoft Agent Framework - Azure OpenAI Responses</a>
- <a href="https://learn.microsoft.com/azure/ai-services/agents/overview" target="_blank">Azure AI Agent service</a>

## Previous Lesson

[Introduction to AI Agents and Agent Use Cases](../01-intro-to-ai-agents/README.md)

## Next Lesson

[Understanding Agentic Design Patterns](../03-agentic-design-patterns/README.md)