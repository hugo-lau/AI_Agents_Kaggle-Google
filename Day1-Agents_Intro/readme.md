# Day 1: Introduction to Agents 🤖

## What the Whitepaper Covered

Day 1's whitepaper was all about laying the foundation for understanding AI agents. Here's what stuck with me:

### Agent Taxonomy
The paper broke down different levels of agent capabilities - basically a classification system for what agents can actually do. Goes from simple task executors to more complex autonomous systems.

### AgentOps - The New Discipline
This was interesting. They're pushing for "AgentOps" as a discipline (like DevOps, but for AI agents). Makes sense when you think about it - if these agents are going to run in production, you need:
- Reliability practices
- Governance frameworks  
- Monitoring and observability

Can't just YOLO agents into production and hope for the best.

### Interoperability & Security
They emphasized that agents need to play nice with each other and with existing systems. Security-wise, they talked about:
- Identity management for agents
- Constrained policies (so agents don't go rogue)
- Access controls

Basically, treating agents like any other system component that needs proper security guardrails.

## What I Did in the Codelabs

### Codelab 1: Built My First Agent with ADK

The first exercise walked through creating a basic agent using Google's Agent Development Kit. Pretty straightforward:

1. **Set up the environment** - Got my Google API key configured, imported ADK components
2. **Defined an agent** - Created a simple agent with a name, model (Gemini 2.5 Flash Lite), and instructions
3. **Gave it a tool** - Added the `google_search` tool so it could look up current info
4. **Ran it** - Used `InMemoryRunner` to actually execute queries

The cool part was seeing the agent decide on its own when to use Google Search vs when it already knew the answer.

### Codelab 2: Multi-Agent Systems

Second codelab was about getting multiple agents working together. This one explored:

- **Creating specialized agents** - Different agents with different capabilities/purposes
- **Agent coordination** - How to get agents to collaborate on complex tasks
- **Architectural patterns** - Different ways to structure multi-agent systems

The idea being that sometimes one generalist agent isn't enough - you want a team of specialists working together.

### Bonus: ADK Web UI

Also played around with the ADK web interface. It's a built-in UI for testing and debugging agents. Used the CLI to scaffold a project:

```bash
adk create sample-agent --model gemini-2.5-flash-lite
```

Creates a proper project structure with config files, agent definition, etc. The web UI was handy for seeing what the agent was actually doing under the hood.

## Quick Takeaways

- **Agents ≠ LLMs** - An agent is an LLM + tools + orchestration logic
- **Tools are the key** - Without tools, you just have a chatbot. With tools, you have an agent that can take action
- **Production matters** - The AgentOps focus shows they're thinking about real-world deployment, not just demos
- **Multi-agent is a thing** - Complex problems might need teams of specialized agents rather than one do-it-all agent

## Resources

- 📄 [Day 1 Whitepaper: Introduction to Agents](https://www.kaggle.com/learn/5-day-agents)
- 🎙️ [Podcast Summary - NotebookLM](https://www.kaggle.com/learn/5-day-agents)
- 💻 Codelabs: Building first agent & multi-agent systems

---

*Part of the [5-Day AI Agents Intensive with Google](https://www.kaggle.com/learn/5-day-agents)
