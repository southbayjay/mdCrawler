Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
All `LLM`s implement the Runnable interface, which comes with **default** implementations of standard runnable methods (i.e. `ainvoke`, `batch`, `abatch`, `stream`, `astream`, `astream_events`).
The **default** streaming implementations provide an`Iterator` (or `AsyncIterator` for asynchronous streaming) that yields a single value: the final output from the underlying chat model provider.
The ability to stream the output token-by-token depends on whether the provider has implemented proper streaming support.
See which integrations support token-by-token streaming here.
note
The **default** implementation does **not** provide support for token-by-token streaming, but it ensures that the model can be swapped in for any other model as it supports the same standard interface.
## Sync stream​
Below we use a `|` to help visualize the delimiter between tokens.
```
from langchain_openai import OpenAIllm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)for chunk in llm.stream("Write me a 1 verse song about sparkling water."):print(chunk, end="|", flush=True)
```

**API Reference:**OpenAI
```
|Spark|ling| water|,| oh| so clear||Bubbles dancing|,| without| fear||Refreshing| taste|,| a| pure| delight||Spark|ling| water|,| my| thirst|'s| delight||
```

## Async streaming​
Let's see how to stream in an async setting using `astream`.
```
from langchain_openai import OpenAIllm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)asyncfor chunk in llm.astream("Write me a 1 verse song about sparkling water."):print(chunk, end="|", flush=True)
```

**API Reference:**OpenAI
```
|Spark|ling| water|,| oh| so clear||Bubbles dancing|,| without| fear||Refreshing| taste|,| a| pure| delight||Spark|ling| water|,| my| thirst|'s| delight||
```

## Async event streaming​
LLMs also support the standard astream events method.
tip
`astream_events` is most useful when implementing streaming in a larger LLM application that contains multiple steps (e.g., an application that involves an `agent`).
```
from langchain_openai import OpenAIllm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0, max_tokens=512)idx =0asyncfor event in llm.astream_events("Write me a 1 verse song about goldfish on the moon", version="v1"):  idx +=1if idx >=5:# Truncate the outputprint("...Truncated")breakprint(event)
```

**API Reference:**OpenAI
#### Was this page helpful?
  * Sync stream
  * Async streaming
  * Async event streaming


