Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
Prerequisites
This guide assumes familiarity with the following concepts:
  * LangChain Expression Language (LCEL)
  * Prompt templates
  * Chat models
  * Output parser


One point about LangChain Expression Language is that any two runnables can be "chained" together into sequences. The output of the previous runnable's `.invoke()` call is passed as input to the next runnable. This can be done using the pipe operator (`|`), or the more explicit `.pipe()` method, which does the same thing.
The resulting `RunnableSequence` is itself a runnable, which means it can be invoked, streamed, or further chained just like any other runnable. Advantages of chaining runnables in this way are efficient streaming (the sequence will stream output as soon as it is available), and debugging and tracing with tools like LangSmith.
## The pipe operator: `|`​
To show off how this works, let's go through an example. We'll walk through a common pattern in LangChain: using a prompt template to format input into a chat model, and finally converting the chat message output into a string with an output parser.
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
import getpassimport osifnot os.environ.get("GROQ_API_KEY"): os.environ["GROQ_API_KEY"]= getpass.getpass("Enter API key for Groq: ")from langchain.chat_models import init_chat_modelmodel = init_chat_model("llama3-8b-8192", model_provider="groq")
```

```
from langchain_core.output_parsers import StrOutputParserfrom langchain_core.prompts import ChatPromptTemplateprompt = ChatPromptTemplate.from_template("tell me a joke about {topic}")chain = prompt | model | StrOutputParser()
```

**API Reference:**StrOutputParser | ChatPromptTemplate
Prompts and models are both runnable, and the output type from the prompt call is the same as the input type of the chat model, so we can chain them together. We can then invoke the resulting sequence like any other runnable:
```
chain.invoke({"topic":"bears"})
```

```
"Here's a bear joke for you:\n\nWhy did the bear dissolve in water?\nBecause it was a polar bear!"
```

### Coercion​
We can even combine this chain with more runnables to create another chain. This may involve some input/output formatting using other types of runnables, depending on the required inputs and outputs of the chain components.
For example, let's say we wanted to compose the joke generating chain with another chain that evaluates whether or not the generated joke was funny.
We would need to be careful with how we format the input into the next chain. In the below example, the dict in the chain is automatically parsed and converted into a `RunnableParallel`, which runs all of its values in parallel and returns a dict with the results.
This happens to be the same format the next prompt template expects. Here it is in action:
```
from langchain_core.output_parsers import StrOutputParseranalysis_prompt = ChatPromptTemplate.from_template("is this a funny joke? {joke}")composed_chain ={"joke": chain}| analysis_prompt | model | StrOutputParser()composed_chain.invoke({"topic":"bears"})
```

**API Reference:**StrOutputParser
```
'Haha, that\'s a clever play on words! Using "polar" to imply the bear dissolved or became polar/polarized when put in water. Not the most hilarious joke ever, but it has a cute, groan-worthy pun that makes it mildly amusing. I appreciate a good pun or wordplay joke.'
```

Functions will also be coerced into runnables, so you can add custom logic to your chains too. The below chain results in the same logical flow as before:
```
composed_chain_with_lambda =(  chain|(lambdainput:{"joke":input})| analysis_prompt| model| StrOutputParser())composed_chain_with_lambda.invoke({"topic":"beets"})
```

```
"Haha, that's a cute and punny joke! I like how it plays on the idea of beets blushing or turning red like someone blushing. Food puns can be quite amusing. While not a total knee-slapper, it's a light-hearted, groan-worthy dad joke that would make me chuckle and shake my head. Simple vegetable humor!"
```

However, keep in mind that using functions like this may interfere with operations like streaming. See this section for more information.
## The `.pipe()` method​
We could also compose the same sequence using the `.pipe()` method. Here's what that looks like:
```
from langchain_core.runnables import RunnableParallelcomposed_chain_with_pipe =(  RunnableParallel({"joke": chain}).pipe(analysis_prompt).pipe(model).pipe(StrOutputParser()))composed_chain_with_pipe.invoke({"topic":"battlestar galactica"})
```

**API Reference:**RunnableParallel
```
"I cannot reproduce any copyrighted material verbatim, but I can try to analyze the humor in the joke you provided without quoting it directly.\n\nThe joke plays on the idea that the Cylon raiders, who are the antagonists in the Battlestar Galactica universe, failed to locate the human survivors after attacking their home planets (the Twelve Colonies) due to using an outdated and poorly performing operating system (Windows Vista) for their targeting systems.\n\nThe humor stems from the juxtaposition of a futuristic science fiction setting with a relatable real-world frustration – the use of buggy, slow, or unreliable software or technology. It pokes fun at the perceived inadequacies of Windows Vista, which was widely criticized for its performance issues and other problems when it was released.\n\nBy attributing the Cylons' failure to locate the humans to their use of Vista, the joke creates an amusing and unexpected connection between a fictional advanced race of robots and a familiar technological annoyance experienced by many people in the real world.\n\nOverall, the joke relies on incongruity and relatability to generate humor, but without reproducing any copyrighted material directly."
```

Or the abbreviated:
```
composed_chain_with_pipe = RunnableParallel({"joke": chain}).pipe(  analysis_prompt, model, StrOutputParser())
```

## Related​
  * Streaming: Check out the streaming guide to understand the streaming behavior of a chain


#### Was this page helpful?
  * The pipe operator: `|`
    * Coercion
  * The `.pipe()` method
  * Related


