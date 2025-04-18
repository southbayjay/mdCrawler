PraisonAI Documentation home page![light logo](https://docs.praison.ai/images/praisonai-logo-large-dark.png)![dark logo](https://docs.praison.ai/images/praisonai-logo-large-light.png)
Search...
Ctrl K
Search...
Navigation
Other Features
PraisonAI Agents
DocumentationExamplesAgentsUIToolsJS
PraisonAI Documentation home page![light logo](https://docs.praison.ai/images/praisonai-logo-large-dark.png)![dark logo](https://docs.praison.ai/images/praisonai-logo-large-light.png)
  * Home


##### Getting Started
  * Introduction
  * Installation
  * Quick Start


##### Core Concepts
  * Agents
  * Tasks
  * Process
  * Tools
  * Memory
  * Knowledge


##### Workflows
  * Agentic Routing
  * Orchestrator Worker
  * Autonomous Workflow
  * Parallelization
  * Prompt Chaining
  * Evaluator Optimizer
  * Repetitive Agents


##### Features
  * CLI
  * AutoAgents
  * Image Generation
  * Self Reflection Agents
  * RAG Agents
  * Reasoning Extract Agents
  * Reasoning Agents
  * Multimodal Agents
  * LangChain Agents
  * Async Agents
  * Mini AI Agents
  * Generate Reasoning Data
  * Code Agent
  * Math Agent
  * Structured AI Agents
  * Callbacks Agent
  * Chat with PDF


##### Models
  * Models in PraisonAI
  * OpenAI ChatGPT
  * Ollama
  * Groq
  * Google Gemini
  * OpenRouter
  * Anthropic
  * Cohere
  * Mistral
  * DeepSeek Agents
  * Other Models


##### Tools
  * Firecrawl PraisonAI Integration


##### Other Features
  * PraisonAI Train
  * CrewAI with PraisonAI
  * AutoGen with PraisonAI
  * PraisonAI Agents


##### Monitoring
  * AgentOps PraisonAI Monitoring


##### Developers
  * Test
  * Agents Playbook
  * PraisonAI Package Integration
  * Integrate with Tools
  * Google Colab Integration
  * Google Colab Tools
  * Local Development
  * Deploy


##### Getting Started (No Code)
  * Introduction
  * TL;DR
  * Installation
  * Initialise
  * Run
  * Auto Generation Mode


##### API Reference
  * API Reference
  * Agent Module
  * Agents Module
  * AutoAgents Module
  * Task Module
  * Process Module


A lightweight package dedicated to creating and managing AI agents with advanced capabilities.
## 
​
Installation
Copy
```
pip install praisonaiagents
export OPENAI_API_KEY=xxxxxxxxxxxxxxxxxxxxxx

```

## 
​
Quick Start
Create `app.py` and add the following code:
Copy
```
from praisonaiagents import Agent, Task, PraisonAIAgents
# Create agents
researcher = Agent(
  name="Researcher",
  role="Senior Research Analyst",
  goal="Uncover cutting-edge developments in AI and data science",
  backstory="""You are an expert at a technology research group, 
  skilled in identifying trends and analyzing complex data.""",
  verbose=True,
  llm="gpt-4o",
  markdown=True
)
writer = Agent(
  name="Writer",
  role="Tech Content Strategist",
  goal="Craft compelling content on tech advancements",
  backstory="""You are a content strategist known for 
  making complex tech topics interesting and easy to understand.""",
  llm="gpt-4o",
  markdown=True
)
# Create tasks
research_task = Task(
  description="Research the latest developments in AI and data science",
  expected_output="A comprehensive report on recent AI trends and breakthroughs",
  agent=researcher
)
writing_task = Task(
  description="Create an engaging blog post about the research findings",
  expected_output="A well-structured blog post explaining AI developments",
  agent=writer,
  context=[research_task] # This task depends on research_task output
)
# Create and run the agents
agents = PraisonAIAgents(
  agents=[researcher, writer],
  tasks=[research_task, writing_task],
  verbose=True
)
result = agents.start()
print(result)

```

## 
​
Agent Configuration
### 
​
Core Attributes
  * `name`: Agent’s identifier
  * `role`: Agent’s function/expertise
  * `goal`: Individual objective
  * `backstory`: Context and personality
  * `llm`: Language model (default: OpenAI’s GPT-4)
  * `verbose`: Enable detailed logs (default: False)
  * `markdown`: Enable markdown formatting (default: True)


### 
​
Optional Attributes
  * `tools`: List of available tools
  * `memory`: Enable conversation history (default: True)
  * `max_iter`: Maximum iterations (default: 20)
  * `max_rpm`: Rate limit for API calls
  * `allow_delegation`: Enable task delegation (default: False)


## 
​
Task Configuration
### 
​
Core Attributes
  * `description`: Task details
  * `expected_output`: Desired outcome
  * `agent`: Assigned agent


### 
​
Optional Attributes
  * `context`: Dependencies on other tasks
  * `tools`: Task-specific tools
  * `async_execution`: Run asynchronously (default: False)
  * `output_file`: Save output to file
  * `callback`: Post-task function


## 
​
Advanced Features
### 
​
Tool Integration
Copy
```
from duckduckgo_search import DDGS
def search_tool(query: str) -> list:
  """
  Perform a web search using DuckDuckGo and return relevant results.
  Args:
    query (str): The search query string to look up information about.
  Returns:
    list: A list of dictionaries containing search results with the following keys:
      - title (str): The title of the search result
      - url (str): The URL of the search result
      - snippet (str): A brief excerpt or description of the search result
  """
  results = []
  ddgs = DDGS()
  for result in ddgs.text(keywords=query, max_results=10):
    results.append({
      "title": result.get("title", ""),
      "url": result.get("href", ""),
      "snippet": result.get("body", ""),
    })
  return results
agent = Agent(
  name="Researcher",
  tools=[search_tool]
)

```

### 
​
Custom Callbacks
Copy
```
def task_completed(output):
  print(f"Task completed: {output.description}")
  print(f"Output: {output.raw}")
task = Task(
  description="Analysis task",
  callback=task_completed
)

```

## 
​
Error Handling
The framework includes built-in error handling for:
  * API rate limits
  * Token limits
  * Task timeouts
  * Tool execution failures


## 
​
Best Practices
  1. **Agent Design**
     * Give clear, specific roles and goals
     * Provide detailed backstories
     * Use appropriate tools for tasks
  2. **Task Management**
     * Break complex tasks into subtasks
     * Set clear dependencies
     * Use async execution for independent tasks
  3. **Resource Optimization**
     * Enable caching when appropriate
     * Set reasonable max_iter limits
     * Use rate limiting for API calls
  4. **Error Handling**
     * Implement task callbacks
     * Set appropriate timeouts
     * Monitor execution logs


### 
​
Parallel Execution (Upcoming)
Copy
```
task1 = Task(
  description="Research task 1",
  async_execution=True
)
task2 = Task(
  description="Research task 2",
  async_execution=True
)
final_task = Task(
  description="Compile results",
  context=[task1, task2] # Waits for both tasks
)

```

### 
​
Memory Management (Upcoming)
Copy
```
agent = Agent(
  name="Analyst",
  memory=True, # Enable memory
  memory_config={
    "type": "short_term",
    "max_tokens": 4000
  }
)

```

Was this page helpful?
YesNo
AutoGen with PraisonAIAgentOps PraisonAI Monitoring
On this page
  * Installation
  * Quick Start
  * Agent Configuration
  * Core Attributes
  * Optional Attributes
  * Task Configuration
  * Core Attributes
  * Optional Attributes
  * Advanced Features
  * Tool Integration
  * Custom Callbacks
  * Error Handling
  * Best Practices
  * Parallel Execution (Upcoming)
  * Memory Management (Upcoming)


