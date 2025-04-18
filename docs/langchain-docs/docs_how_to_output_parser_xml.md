Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
Prerequisites
This guide assumes familiarity with the following concepts:
  * Chat models
  * Output parsers
  * Prompt templates
  * Structured output
  * Chaining runnables together


LLMs from different providers often have different strengths depending on the specific data they are trained on. This also means that some may be "better" and more reliable at generating output in formats other than JSON.
This guide shows you how to use the `XMLOutputParser` to prompt models for XML output, then and parse that output into a usable format.
note
Keep in mind that large language models are leaky abstractions! You'll have to use an LLM with sufficient capacity to generate well-formed XML.
In the following examples, we use Anthropic's Claude-2 model (https://docs.anthropic.com/claude/docs), which is one such model that is optimized for XML tags.
```
%pip install -qU langchain langchain-anthropicimport osfrom getpass import getpassif"ANTHROPIC_API_KEY"notin os.environ:  os.environ["ANTHROPIC_API_KEY"]= getpass()
```

Let's start with a simple request to the model.
```
from langchain_anthropic import ChatAnthropicfrom langchain_core.output_parsers import XMLOutputParserfrom langchain_core.prompts import PromptTemplatemodel = ChatAnthropic(model="claude-2.1", max_tokens_to_sample=512, temperature=0.1)actor_query ="Generate the shortened filmography for Tom Hanks."output = model.invoke(f"""{actor_query}Please enclose the movies in <movie></movie> tags""")print(output.content)
```

**API Reference:**ChatAnthropic | XMLOutputParser | PromptTemplate
```
Here is the shortened filmography for Tom Hanks, with movies enclosed in XML tags:<movie>Splash</movie><movie>Big</movie><movie>A League of Their Own</movie><movie>Sleepless in Seattle</movie><movie>Forrest Gump</movie><movie>Toy Story</movie><movie>Apollo 13</movie><movie>Saving Private Ryan</movie><movie>Cast Away</movie><movie>The Da Vinci Code</movie>
```

This actually worked pretty well! But it would be nice to parse that XML into a more easily usable format. We can use the `XMLOutputParser` to both add default format instructions to the prompt and parse outputted XML into a dict:
```
parser = XMLOutputParser()# We will add these instructions to the prompt belowparser.get_format_instructions()
```

```
'The output should be formatted as a XML file.\n1. Output should conform to the tags below. \n2. If tags are not given, make them on your own.\n3. Remember to always open and close all the tags.\n\nAs an example, for the tags ["foo", "bar", "baz"]:\n1. String "<foo>\n  <bar>\n   <baz></baz>\n  </bar>\n</foo>" is a well-formatted instance of the schema. \n2. String "<foo>\n  <bar>\n  </foo>" is a badly-formatted instance.\n3. String "<foo>\n  <tag>\n  </tag>\n</foo>" is a badly-formatted instance.\n\nHere are the output tags:\n\`\`\`\nNone\n\`\`\`'
```

```
prompt = PromptTemplate(  template="""{query}\n{format_instructions}""",  input_variables=["query"],  partial_variables={"format_instructions": parser.get_format_instructions()},)chain = prompt | model | parseroutput = chain.invoke({"query": actor_query})print(output)
```

```
{'filmography': [{'movie': [{'title': 'Big'}, {'year': '1988'}]}, {'movie': [{'title': 'Forrest Gump'}, {'year': '1994'}]}, {'movie': [{'title': 'Toy Story'}, {'year': '1995'}]}, {'movie': [{'title': 'Saving Private Ryan'}, {'year': '1998'}]}, {'movie': [{'title': 'Cast Away'}, {'year': '2000'}]}]}
```

We can also add some tags to tailor the output to our needs. You can and should experiment with adding your own formatting hints in the other parts of your prompt to either augment or replace the default instructions:
```
parser = XMLOutputParser(tags=["movies","actor","film","name","genre"])# We will add these instructions to the prompt belowparser.get_format_instructions()
```

```
'The output should be formatted as a XML file.\n1. Output should conform to the tags below. \n2. If tags are not given, make them on your own.\n3. Remember to always open and close all the tags.\n\nAs an example, for the tags ["foo", "bar", "baz"]:\n1. String "<foo>\n  <bar>\n   <baz></baz>\n  </bar>\n</foo>" is a well-formatted instance of the schema. \n2. String "<foo>\n  <bar>\n  </foo>" is a badly-formatted instance.\n3. String "<foo>\n  <tag>\n  </tag>\n</foo>" is a badly-formatted instance.\n\nHere are the output tags:\n\`\`\`\n[\'movies\', \'actor\', \'film\', \'name\', \'genre\']\n\`\`\`'
```

```
prompt = PromptTemplate(  template="""{query}\n{format_instructions}""",  input_variables=["query"],  partial_variables={"format_instructions": parser.get_format_instructions()},)chain = prompt | model | parseroutput = chain.invoke({"query": actor_query})print(output)
```

```
{'movies': [{'actor': [{'name': 'Tom Hanks'}, {'film': [{'name': 'Forrest Gump'}, {'genre': 'Drama'}]}, {'film': [{'name': 'Cast Away'}, {'genre': 'Adventure'}]}, {'film': [{'name': 'Saving Private Ryan'}, {'genre': 'War'}]}]}]}
```

This output parser also supports streaming of partial chunks. Here's an example:
```
for s in chain.stream({"query": actor_query}):print(s)
```

```
{'movies': [{'actor': [{'name': 'Tom Hanks'}]}]}{'movies': [{'actor': [{'film': [{'name': 'Forrest Gump'}]}]}]}{'movies': [{'actor': [{'film': [{'genre': 'Drama'}]}]}]}{'movies': [{'actor': [{'film': [{'name': 'Cast Away'}]}]}]}{'movies': [{'actor': [{'film': [{'genre': 'Adventure'}]}]}]}{'movies': [{'actor': [{'film': [{'name': 'Saving Private Ryan'}]}]}]}{'movies': [{'actor': [{'film': [{'genre': 'War'}]}]}]}
```

## Next steps​
You've now learned how to prompt a model to return XML. Next, check out the broader guide on obtaining structured output for other related techniques.
#### Was this page helpful?
  * Next steps


