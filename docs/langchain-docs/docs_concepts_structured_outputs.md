Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
## Overview​
For many applications, such as chatbots, models need to respond to users directly in natural language. However, there are scenarios where we need models to output in a _structured format_. For example, we might want to store the model output in a database and ensure that the output conforms to the database schema. This need motivates the concept of structured output, where models can be instructed to respond with a particular output structure.
![Structured output](https://python.langchain.com/assets/images/structured_output-2c42953cee807dedd6e96f3e1db17f69.png)
## Key concepts​
**(1) Schema definition:** The output structure is represented as a schema, which can be defined in several ways. **(2) Returning structured output:** The model is given this schema, and is instructed to return output that conforms to it.
## Recommended usage​
This pseudo-code illustrates the recommended workflow when using structured output. LangChain provides a method, `with_structured_output()`, that automates the process of binding the schema to the model and parsing the output. This helper function is available for all model providers that support structured output.
```
# Define schemaschema ={"foo":"bar"}# Bind schema to modelmodel_with_structure = model.with_structured_output(schema)# Invoke the model to produce structured output that matches the schemastructured_output = model_with_structure.invoke(user_input)
```

## Schema definition​
The central concept is that the output structure of model responses needs to be represented in some way. While types of objects you can use depend on the model you're working with, there are common types of objects that are typically allowed or recommended for structured output in Python.
The simplest and most common format for structured output is a JSON-like structure, which in Python can be represented as a dictionary (dict) or list (list). JSON objects (or dicts in Python) are often used directly when the tool requires raw, flexible, and minimal-overhead structured data.
```
{"answer":"The answer to the user's question","followup_question":"A followup question the user could ask"}
```

As a second example, Pydantic is particularly useful for defining structured output schemas because it offers type hints and validation. Here's an example of a Pydantic schema:
```
from pydantic import BaseModel, FieldclassResponseFormatter(BaseModel):"""Always use this tool to structure your response to the user."""  answer:str= Field(description="The answer to the user's question")  followup_question:str= Field(description="A followup question the user could ask")
```

## Returning structured output​
With a schema defined, we need a way to instruct the model to use it. While one approach is to include this schema in the prompt and _ask nicely_ for the model to use it, this is not recommended. Several more powerful methods that utilizes native features in the model provider's API are available.
### Using tool calling​
Many model providers support tool calling, a concept discussed in more detail in our tool calling guide. In short, tool calling involves binding a tool to a model and, when appropriate, the model can _decide_ to call this tool and ensure its response conforms to the tool's schema. With this in mind, the central concept is straightforward: _simply bind our schema to a model as a tool!_ Here is an example using the `ResponseFormatter` schema defined above:
```
from langchain_openai import ChatOpenAImodel = ChatOpenAI(model="gpt-4o", temperature=0)# Bind responseformatter schema as a tool to the modelmodel_with_tools = model.bind_tools([ResponseFormatter])# Invoke the modelai_msg = model_with_tools.invoke("What is the powerhouse of the cell?")
```

**API Reference:**ChatOpenAI
The arguments of the tool call are already extracted as a dictionary. This dictionary can be optionally parsed into a Pydantic object, matching our original `ResponseFormatter` schema.
```
# Get the tool call argumentsai_msg.tool_calls[0]["args"]{'answer':"The powerhouse of the cell is the mitochondrion. Mitochondria are organelles that generate most of the cell's supply of adenosine triphosphate (ATP), which is used as a source of chemical energy.",'followup_question':'What is the function of ATP in the cell?'}# Parse the dictionary into a pydantic objectpydantic_object = ResponseFormatter.model_validate(ai_msg.tool_calls[0]["args"])
```

### JSON mode​
In addition to tool calling, some model providers support a feature called `JSON mode`. This supports JSON schema definition as input and enforces the model to produce a conforming JSON output. You can find a table of model providers that support JSON mode here. Here is an example of how to use JSON mode with OpenAI:
```
from langchain_openai import ChatOpenAImodel = ChatOpenAI(model="gpt-4o", model_kwargs={"response_format":{"type":"json_object"}})ai_msg = model.invoke("Return a JSON object with key 'random_ints' and a value of 10 random ints in [0-99]")ai_msg.content'\n{\n "random_ints": [23, 47, 89, 15, 34, 76, 58, 3, 62, 91]\n}'
```

**API Reference:**ChatOpenAI
One important point to flag: the model _still_ returns a string, which needs to be parsed into a JSON object. This can, of course, simply use the `json` library or a JSON output parser if you need more advanced functionality. See this how-to guide on the JSON output parser for more details.
```
import jsonjson_object = json.loads(ai_msg.content){'random_ints':[23,47,89,15,34,76,58,3,62,91]}
```

## Structured output method​
There are a few challenges when producing structured output with the above methods:
(1) When tool calling is used, tool call arguments needs to be parsed from a dictionary back to the original schema.
(2) In addition, the model needs to be instructed to _always_ use the tool when we want to enforce structured output, which is a provider specific setting.
(3) When JSON mode is used, the output needs to be parsed into a JSON object.
With these challenges in mind, LangChain provides a helper function (`with_structured_output()`) to streamline the process.
![Diagram of with structured output](https://python.langchain.com/assets/images/with_structured_output-4fd0fdc94f644554d52c6a8dee96ea21.png)
This both binds the schema to the model as a tool and parses the output to the specified output schema.
```
# Bind the schema to the modelmodel_with_structure = model.with_structured_output(ResponseFormatter)# Invoke the modelstructured_output = model_with_structure.invoke("What is the powerhouse of the cell?")# Get back the pydantic objectstructured_outputResponseFormatter(answer="The powerhouse of the cell is the mitochondrion. Mitochondria are organelles that generate most of the cell's supply of adenosine triphosphate (ATP), which is used as a source of chemical energy.", followup_question='What is the function of ATP in the cell?')
```

Further reading
For more details on usage, see our how-to guide.
#### Was this page helpful?
  * Overview
  * Key concepts
  * Recommended usage
  * Schema definition
  * Returning structured output
    * Using tool calling
    * JSON mode
  * Structured output method


