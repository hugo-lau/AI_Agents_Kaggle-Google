# Day 2: Agent Tools & Model Context Protocol (MCP) 🛠️

## What the Whitepaper Covered

Day 2 was all about giving agents real capabilities through tools and introducing MCP as a standardization layer.

### External Tools - The Agent's "Hands and Eyes"

The core idea: **foundation models without tools are just pattern predictors**. They can't access real-time data or take actions. Tools transform them into actual agents that can:
- **Know something**: Retrieve data from APIs, databases, web searches
- **Do something**: Execute code, call external services, modify systems

### Types of Tools

**Function Tools**: Custom Python functions you define and give to your agent
**Built-in Tools**: Model-native capabilities (Google Search, Code Execution)
**Agent Tools**: Other agents acting as specialist tools (delegation pattern)

### Model Context Protocol (MCP)

MCP is basically a standardization framework for how agents talk to external tools. Think of it like USB - instead of every tool needing its own custom integration, MCP provides:
- Standard communication protocols
- Tool discovery mechanisms  
- Structured request/response formats

**Architecture components**:
- **MCP Server**: Exposes tools through standard protocol
- **MCP Client**: Connects to servers and makes tools available to agents
- **Transport Layer**: How they communicate (stdio, HTTP, WebSocket)

The whitepaper also covered security considerations - tool access needs proper guardrails, authentication, and approval workflows for high-risk operations.

### Best Practices for Tool Design

Key takeaways from the paper:
- Keep tools **granular** - single responsibility principle
- Provide **clear descriptions** - the agent needs to understand what each tool does
- Include **parameter validation** - don't trust agent outputs blindly
- Implement **approval workflows** for risky operations
- Use **timeouts** - don't let tools hang indefinitely

## What I Did in the Codelabs

### Codelab 1: Custom Tools & Currency Converter

Built a currency conversion agent with custom Python function tools. The exercise showed how to:

**Create custom tools from Python functions**:
```python
def get_exchange_rate(from_currency: str, to_currency: str) -> dict:
    """Get exchange rate between two currencies"""
    # Implementation here
    
def calculate_fees(amount: float, transfer_method: str) -> dict:
    """Calculate transfer fees based on method"""
    # Implementation here
```

**Give them to an agent**:
```python
currency_agent = LlmAgent(
    model=Gemini(model="gemini-2.5-flash-lite"),
    instruction="You help with currency conversions and fees",
    tools=[
        FunctionTool(func=get_exchange_rate),
        FunctionTool(func=calculate_fees)
    ],
)
```

**Agent Tool pattern** - Used a specialist calculation agent as a tool for the main agent. This showed the delegation pattern where complex math gets handed off to a code execution specialist rather than trusting the LLM to do arithmetic.

### Codelab 2: MCP Integration & Batch Number Processing with Approval

This one was more involved - built a batch number processing agent with human-in-the-loop approval workflow for large operations.

**Part 1: Connect to MCP Server**

Connected to an external MCP server (the "Everything Server") that provides basic math tools:
```python
mcp_server = McpToolset(
    connection_params=StdioConnectionParams(
        server_params=StdioServerParameters(
            command="npx",
            args=["-y", "@modelcontextprotocol/server-everything"],
            tool_filter=["add"],  # Security: only expose the add tool
        ),
        timeout=30,
    )
)
```

This launches an external server process and makes its tools available to the agent. The `tool_filter` is a security best practice - explicit allowlist of which tools the agent can use.

**Part 2: Long-Running Operations with Approval**

Built a custom batch addition function that implements approval logic for large batches:

```python
def batch_add(numbers: List[int], tool_context: ToolContext) -> dict:
    BATCH_THRESHOLD = 3  # More than 3 numbers requires approval
    count = len(numbers)
    
    # Scenario 1: Small batch (≤3) - auto-approve
    if count <= BATCH_THRESHOLD:
        result = sum(numbers)
        return {
            "status": "approved",
            "result": result,
            "message": f"✅ Auto: sum({numbers}) = {result}"
        }
    
    # Scenario 2: First call for large batch - request approval
    if not tool_context.tool_confirmation:
        tool_context.request_confirmation(
            hint=f"Batch add {count} numbers. OK?",
            payload={"count": count, "numbers": numbers}
        )
        return {
            "status": "pending",
            "message": f"⏸️ Approval needed for {count} numbers"
        }
    
    # Scenario 3: Resume after approval
    if tool_context.tool_confirmation.confirmed:
        result = sum(numbers)
        return {
            "status": "approved",
            "result": result,
            "message": f"✅ Approved: sum of {count} numbers = {result}"
        }
    else:
        return {
            "status": "rejected",
            "message": f"❌ Cancelled: sum of {count} numbers"
        }
```

**The ToolContext Magic**: 
- Allows tools to pause execution
- Request human approval
- Resume from exact same point after approval
- All state is preserved automatically

**Wrapped in Resumable App**:
```python
number_app = App(
    name="number_app",
    root_agent=number_agent,
    resumability_config=ResumabilityConfig(is_resumable=True),
)
```

This enables the pause/resume capability. Without it, agents can't stop mid-execution and wait for external input.

**Approval Workflow**:
1. User: "Sum 10 numbers: 1,2,3,4,5,6,7,8,9,10"
2. Agent calls `batch_add(numbers=[1,2,3,4,5,6,7,8,9,10])`
3. Tool detects large batch (10 > 3), pauses, returns confirmation request
4. System shows user: "Batch add 10 numbers. OK?"
5. Execution pauses - state saved to session service
6. User approves
7. System resumes - calls tool again with approval context
8. Tool computes sum and returns result: 55
9. Agent shows result to user

**Testing Three Scenarios**:
```python
# Demo 1: Small batch - auto-approved
await run_number_workflow("Sum these: 1, 2, 3, 4")

# Demo 2: Large batch - approved
await run_number_workflow("Sum 10 numbers: 1,2,3,4,5,6,7,8,9,10", approved=True)

# Demo 3: Large batch - rejected
await run_number_workflow("Multiply 8 numbers: 2,3,4,5,6,7,8,9", approved=False)
```

## Key Concepts I Learned

### The ToolContext Parameter

When ADK calls your function, it automatically provides a `ToolContext` object. This gives you:
- **request_confirmation()**: Pause and ask for approval
- **tool_confirmation**: Check approval status when resumed
- **invocation_id**: Track which specific call this is

This is how long-running operations work - the tool can pause itself, wait for external input, then resume with context intact.

### Resumability Architecture

For pause/resume to work, you need:
1. **App wrapper** - not just a bare agent
2. **ResumabilityConfig(is_resumable=True)** - enables state persistence
3. **Session service** - stores state during pauses
4. **Runner with app** - orchestrates the workflow

Without this infrastructure, agents can only do synchronous, run-to-completion operations.

### MCP Server Lifecycle

1. **Launch**: Client spawns server process (via npx, python, etc)
2. **Connect**: Establish transport (stdio, HTTP, WebSocket)
3. **Discover**: Client asks "what tools do you have?"
4. **Execute**: Agent calls tools through the client
5. **Return**: Server sends structured responses back

The beauty of MCP is that once a tool has an MCP server, any MCP client can use it without custom integration code.

## Challenges & Gotchas

**Rate Limits**: Hit API quota limits when testing multiple workflows quickly. Free tier Gemini has limits (15 RPM for Flash). Had to add 30-second delays between demo runs to avoid quota errors.

**Timeout Issues**: MCP tools that take too long can timeout. Need to balance timeout values with actual operation time.

**State Management**: Resumable workflows require careful state handling. The session service persists everything, but you need to structure your code to handle the pause/resume correctly.

**Type Hints**: Had to use proper type hints (`List[int]` instead of just `list`) to avoid validation errors with ADK's function tools.

**Invocation ID Tracking**: The `invocation_id` is critical for resuming the correct execution. Without it, the system doesn't know which paused operation to continue.

## Tech Stack for Day 2

- **ADK Function Tools** - Wrapping custom Python functions
- **MCP Toolset** - Connecting to external MCP servers
- **ToolContext** - Enabling pause/resume in tools
- **Resumable Apps** - State persistence across pauses
- **Session Services** - In-memory state storage
- **Stdio Transport** - Local process communication with MCP servers



---

*Part of the [5-Day AI Agents Intensive with Google](https://www.kaggle.com/learn/5-day-agents) | November 2025*
