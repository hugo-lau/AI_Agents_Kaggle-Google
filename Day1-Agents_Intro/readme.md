# Day 1: First Steps into AI Agents 🤖

## What I Built Today

Kicked off the Kaggle x Google AI Agents course by building my first actual AI agent! Not just calling an API, but creating an autonomous agent that can think, act, and use tools to solve problems.

### The Main Projects

1. **Built My First Agent** - Created a basic agent using Google's Agent Development Kit (ADK) that can use Google Search to answer questions with current information
2. **Explored Multi-Agent Systems** - Got my hands dirty with multiple agents working together (separate codelab)

## Key Concepts I Learned

### What Even Is an Agent?

An **AI agent** isn't just an LLM chatbot. It's a system that operates in a continuous loop:
- **THINK** → The model reasons about what to do next
- **ACT** → Takes actions using tools (like searching the web, calling APIs, etc.)
- **OBSERVE** → Gets results back and decides if the mission is complete

This cycle keeps going until the agent achieves its goal. Pretty cool that it makes these decisions autonomously!

### The ADK Architecture

ADK (Agent Development Kit) has these core components:

```python
Agent(
    name="helpful_assistant",
    model=Gemini(model="gemini-2.5-flash-lite"),
    description="A simple agent that can answer general questions",
    instruction="You are a helpful assistant. Use Google Search for current info",
    tools=[google_search],  # This is the magic!
)
```

**What I found interesting**: The agent decides *when* to use tools on its own. I just gave it the capability, and it figured out when searching was necessary vs when it could answer from its training.

### The Runner & Sessions

The **Runner** is like the orchestrator - it manages the conversation flow and coordinates between you, the agent, and the tools.

I learned about **sessions** too - they're like short-term memory containers for a conversation. Each session has its own history that persists across multiple back-and-forth exchanges.

## What I Actually Built

### My First Agent

Created an agent that can answer questions by:
1. Deciding if it needs current information
2. Calling Google Search when needed
3. Synthesizing results into a coherent answer

Example question I tested: *"What is Agent Development Kit from Google? What languages is the SDK available in?"*

The agent autonomously decided to search, found that ADK is available in **Python, Java, and Go**, and gave me a solid answer about what ADK actually is.

### The ADK Web Interface

Also explored the ADK web UI - a built-in interface for testing and debugging agents. Used the CLI to scaffold a new agent project:

```bash
adk create sample-agent --model gemini-2.5-flash-lite
```

This creates a proper project structure with `.env` for API keys, `agent.py` for the agent definition, etc.

## Aha Moments 💡

1. **Tools Transform LLMs into Agents** - Without tools, an LLM is stuck with outdated knowledge. With tools (like Google Search), it becomes an agent that can access real-time information and take action.

2. **The Agent Decides** - I didn't have to tell it "first search, then answer." It figured out the strategy on its own based on the instruction and available tools.

3. **Infrastructure Matters** - Learned that everything runs on Kaggle's cloud VMs, not locally. The web UI uses proxy URLs to tunnel through Kaggle's infrastructure.

## Tech Stack for Day 1

- **ADK** (Agent Development Kit) - Google's framework for building agents
- **Gemini 2.5 Flash Lite** - The LLM model powering the agent's reasoning
- **Google Search Tool** - Built-in tool for accessing current web information
- **InMemoryRunner** - For executing agent interactions
- **InMemorySessionService** - For managing conversation state



