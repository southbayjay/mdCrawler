Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
For more complex tool use it's very useful to add few-shot examples to the prompt. We can do this by adding `AIMessage`s with `ToolCall`s and corresponding `ToolMessage`s to our prompt.
First let's define our tools and model.
```
from langchain_core.tools import tool@tooldefadd(a:int, b:int)->int:"""Adds a and b."""return a + b@tooldefmultiply(a:int, b:int)->int:"""Multiplies a and b."""return a * btools =[add, multiply]
```

**API Reference:**tool
```
import osfrom getpass import getpassfrom langchain_openai import ChatOpenAIif"OPENAI_API_KEY"notin os.environ:  os.environ["OPENAI_API_KEY"]= getpass()llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)llm_with_tools = llm.bind_tools(tools)
```

**API Reference:**ChatOpenAI
Let's run our model where we can notice that even with some special instructions our model can get tripped up by order of operations.
```
llm_with_tools.invoke("Whats 119 times 8 minus 20. Don't do any math yourself, only use tools for math. Respect order of operations").tool_calls
```

```
[{'name': 'Multiply', 'args': {'a': 119, 'b': 8}, 'id': 'call_T88XN6ECucTgbXXkyDeC2CQj'}, {'name': 'Add', 'args': {'a': 952, 'b': -20}, 'id': 'call_licdlmGsRqzup8rhqJSb1yZ4'}]
```

The model shouldn't be trying to add anything yet, since it technically can't know the results of 119 * 8 yet.
By adding a prompt with some examples we can correct this behavior:
```
from langchain_core.messages import AIMessage, HumanMessage, ToolMessagefrom langchain_core.prompts import ChatPromptTemplatefrom langchain_core.runnables import RunnablePassthroughexamples =[  HumanMessage("What's the product of 317253 and 128472 plus four", name="example_user"),  AIMessage("",    name="example_assistant",    tool_calls=[{"name":"Multiply","args":{"x":317253,"y":128472},"id":"1"}],),  ToolMessage("16505054784", tool_call_id="1"),  AIMessage("",    name="example_assistant",    tool_calls=[{"name":"Add","args":{"x":16505054784,"y":4},"id":"2"}],),  ToolMessage("16505054788", tool_call_id="2"),  AIMessage("The product of 317253 and 128472 plus four is 16505054788",    name="example_assistant",),]system ="""You are bad at math but are an expert at using a calculator. Use past tool usage as an example of how to correctly use the tools."""few_shot_prompt = ChatPromptTemplate.from_messages([("system", system),*examples,("human","{query}"),])chain ={"query": RunnablePassthrough()}| few_shot_prompt | llm_with_toolschain.invoke("Whats 119 times 8 minus 20").tool_calls
```

**API Reference:**AIMessage | HumanMessage | ToolMessage | ChatPromptTemplate | RunnablePassthrough
```
[{'name': 'Multiply', 'args': {'a': 119, 'b': 8}, 'id': 'call_9MvuwQqg7dlJupJcoTWiEsDo'}]
```

And we get the correct output this time.
Here's what the LangSmith trace looks like.
#### Was this page helpful?
