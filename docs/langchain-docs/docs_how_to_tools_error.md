Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
Prerequisites
This guide assumes familiarity with the following concepts:
  * Chat models
  * LangChain Tools
  * How to use a model to call tools


Calling tools with an LLM is generally more reliable than pure prompting, but it isn't perfect. The model may try to call a tool that doesn't exist or fail to return arguments that match the requested schema. Strategies like keeping schemas simple, reducing the number of tools you pass at once, and having good names and descriptions can help mitigate this risk, but aren't foolproof.
This guide covers some ways to build error handling into your chains to mitigate these failure modes.
## Setup​
We'll need to install the following packages:
```
%pip install --upgrade --quiet langchain-core langchain-openai
```

If you'd like to trace your runs in LangSmith uncomment and set the following environment variables:
```
import getpassimport os# os.environ["LANGSMITH_TRACING"] = "true"# os.environ["LANGSMITH_API_KEY"] = getpass.getpass()
```

## Chain​
Suppose we have the following (dummy) tool and tool-calling chain. We'll make our tool intentionally convoluted to try and trip up the model.
Select chat model:
Groq▾
* Groq
* OpenAI
* Anthropic
* Azure
* Google Vertex
* AWS
* Cohere
* NVIDIA
* Fireworks AI
* Mistral AI
* Together AI
* IBM watsonx
* Databricks
```
pip install -qU "langchain[groq]"
```

```
import getpassimport osifnot os.environ.get("GROQ_API_KEY"): os.environ["GROQ_API_KEY"]= getpass.getpass("Enter API key for Groq: ")from langchain.chat_models import init_chat_modelllm = init_chat_model("llama3-8b-8192", model_provider="groq")
```

```
# Define toolfrom langchain_core.tools import tool@tooldefcomplex_tool(int_arg:int, float_arg:float, dict_arg:dict)->int:"""Do something complex with a complex tool."""return int_arg * float_argllm_with_tools = llm.bind_tools([complex_tool],)# Define chainchain = llm_with_tools |(lambda msg: msg.tool_calls[0]["args"])| complex_tool
```

**API Reference:**tool
We can see that when we try to invoke this chain with even a fairly explicit input, the model fails to correctly call the tool (it forgets the `dict_arg` argument).
```
chain.invoke("use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg")
```

```
---------------------------------------------------------------------------``````outputValidationError              Traceback (most recent call last)``````outputCell In[5], line 1----> 1 chain.invoke(   2   "use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"   3 )``````outputFile ~/langchain/.venv/lib/python3.11/site-packages/langchain_core/runnables/base.py:2998, in RunnableSequence.invoke(self, input, config, **kwargs)  2996       input = context.run(step.invoke, input, config, **kwargs)  2997     else:-> 2998       input = context.run(step.invoke, input, config)  2999 # finish the root run  3000 except BaseException as e:``````outputFile ~/langchain/.venv/lib/python3.11/site-packages/langchain_core/tools/base.py:456, in BaseTool.invoke(self, input, config, **kwargs)  449 def invoke(  450   self,  451   input: Union[str, Dict, ToolCall],  452   config: Optional[RunnableConfig] = None,  453   **kwargs: Any,  454 ) -> Any:  455   tool_input, kwargs = _prep_run_args(input, config, **kwargs)--> 456   return self.run(tool_input, **kwargs)``````outputFile ~/langchain/.venv/lib/python3.11/site-packages/langchain_core/tools/base.py:659, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, config, tool_call_id, **kwargs)  657 if error_to_raise:  658   run_manager.on_tool_error(error_to_raise)--> 659   raise error_to_raise  660 output = _format_output(content, artifact, tool_call_id, self.name, status)  661 run_manager.on_tool_end(output, color=color, name=self.name, **kwargs)``````outputFile ~/langchain/.venv/lib/python3.11/site-packages/langchain_core/tools/base.py:622, in BaseTool.run(self, tool_input, verbose, start_color, color, callbacks, tags, metadata, run_name, run_id, config, tool_call_id, **kwargs)  620 context = copy_context()  621 context.run(_set_config_context, child_config)--> 622 tool_args, tool_kwargs = self._to_args_and_kwargs(tool_input)  623 if signature(self._run).parameters.get("run_manager"):  624   tool_kwargs["run_manager"] = run_manager``````outputFile ~/langchain/.venv/lib/python3.11/site-packages/langchain_core/tools/base.py:545, in BaseTool._to_args_and_kwargs(self, tool_input)  544 def _to_args_and_kwargs(self, tool_input: Union[str, Dict]) -> Tuple[Tuple, Dict]:--> 545   tool_input = self._parse_input(tool_input)  546   # For backwards compatibility, if run_input is a string,  547   # pass as a positional argument.  548   if isinstance(tool_input, str):``````outputFile ~/langchain/.venv/lib/python3.11/site-packages/langchain_core/tools/base.py:487, in BaseTool._parse_input(self, tool_input)  485 if input_args is not None:  486   if issubclass(input_args, BaseModel):--> 487     result = input_args.model_validate(tool_input)  488     result_dict = result.model_dump()  489   elif issubclass(input_args, BaseModelV1):``````outputFile ~/langchain/.venv/lib/python3.11/site-packages/pydantic/main.py:568, in BaseModel.model_validate(cls, obj, strict, from_attributes, context)  566 # `__tracebackhide__` tells pytest and some other tools to omit this function from tracebacks  567 __tracebackhide__ = True--> 568 return cls.__pydantic_validator__.validate_python(  569   obj, strict=strict, from_attributes=from_attributes, context=context  570 )``````outputValidationError: 1 validation error for complex_toolSchemadict_arg Field required [type=missing, input_value={'int_arg': 5, 'float_arg': 2.1}, input_type=dict]  For further information visit https://errors.pydantic.dev/2.8/v/missing
```

## Try/except tool call​
The simplest way to more gracefully handle errors is to try/except the tool-calling step and return a helpful message on errors:
```
from typing import Anyfrom langchain_core.runnables import Runnable, RunnableConfigdeftry_except_tool(tool_args:dict, config: RunnableConfig)-> Runnable:try:    complex_tool.invoke(tool_args, config=config)except Exception as e:returnf"Calling tool with arguments:\n\n{tool_args}\n\nraised the following error:\n\n{type(e)}: {e}"chain = llm_with_tools |(lambda msg: msg.tool_calls[0]["args"])| try_except_toolprint(  chain.invoke("use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"))
```

**API Reference:**Runnable | RunnableConfig
```
Calling tool with arguments:{'int_arg': 5, 'float_arg': 2.1}raised the following error:<class 'pydantic_core._pydantic_core.ValidationError'>: 1 validation error for complex_toolSchemadict_arg Field required [type=missing, input_value={'int_arg': 5, 'float_arg': 2.1}, input_type=dict]  For further information visit https://errors.pydantic.dev/2.8/v/missing
```

## Fallbacks​
We can also try to fallback to a better model in the event of a tool invocation error. In this case we'll fall back to an identical chain that uses `gpt-4-1106-preview` instead of `gpt-3.5-turbo`.
```
chain = llm_with_tools |(lambda msg: msg.tool_calls[0]["args"])| complex_toolbetter_model = ChatOpenAI(model="gpt-4-1106-preview", temperature=0).bind_tools([complex_tool], tool_choice="complex_tool")better_chain = better_model |(lambda msg: msg.tool_calls[0]["args"])| complex_toolchain_with_fallback = chain.with_fallbacks([better_chain])chain_with_fallback.invoke("use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg")
```

```
10.5
```

Looking at the LangSmith trace for this chain run, we can see that the first chain call fails as expected and it's the fallback that succeeds.
## Retry with exception​
To take things one step further, we can try to automatically re-run the chain with the exception passed in, so that the model may be able to correct its behavior:
```
from langchain_core.messages import AIMessage, HumanMessage, ToolCall, ToolMessagefrom langchain_core.prompts import ChatPromptTemplateclassCustomToolException(Exception):"""Custom LangChain tool exception."""def__init__(self, tool_call: ToolCall, exception: Exception)->None:super().__init__()    self.tool_call = tool_call    self.exception = exceptiondeftool_custom_exception(msg: AIMessage, config: RunnableConfig)-> Runnable:try:return complex_tool.invoke(msg.tool_calls[0]["args"], config=config)except Exception as e:raise CustomToolException(msg.tool_calls[0], e)defexception_to_messages(inputs:dict)->dict:  exception = inputs.pop("exception")# Add historical messages to the original input, so the model knows that it made a mistake with the last tool call.  messages =[    AIMessage(content="", tool_calls=[exception.tool_call]),    ToolMessage(      tool_call_id=exception.tool_call["id"], content=str(exception.exception)),    HumanMessage(      content="The last tool call raised an exception. Try calling the tool again with corrected arguments. Do not repeat mistakes."),]  inputs["last_output"]= messagesreturn inputs# We add a last_output MessagesPlaceholder to our prompt which if not passed in doesn't# affect the prompt at all, but gives us the option to insert an arbitrary list of Messages# into the prompt if needed. We'll use this on retries to insert the error message.prompt = ChatPromptTemplate.from_messages([("human","{input}"),("placeholder","{last_output}")])chain = prompt | llm_with_tools | tool_custom_exception# If the initial chain call fails, we rerun it withe the exception passed in as a message.self_correcting_chain = chain.with_fallbacks([exception_to_messages | chain], exception_key="exception")
```

**API Reference:**AIMessage | HumanMessage | ToolCall | ToolMessage | ChatPromptTemplate
```
self_correcting_chain.invoke({"input":"use complex tool. the args are 5, 2.1, empty dictionary. don't forget dict_arg"})
```

```
10.5
```

And our chain succeeds! Looking at the LangSmith trace, we can see that indeed our initial chain still fails, and it's only on retrying that the chain succeeds.
## Next steps​
Now you've seen some strategies how to handle tool calling errors. Next, you can learn more about how to use tools:
  * Few shot prompting with tools
  * Stream tool calls
  * Pass runtime values to tools


You can also check out some more specific uses of tool calling:
  * Getting structured outputs from models


#### Was this page helpful?
  * Setup
  * Chain
  * Try/except tool call
  * Fallbacks
  * Retry with exception
  * Next steps


