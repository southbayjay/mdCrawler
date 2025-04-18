Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
Prerequisites
  * Chat Models


## Overview​
Messages are the unit of communication in chat models. They are used to represent the input and output of a chat model, as well as any additional context or metadata that may be associated with a conversation.
Each message has a **role** (e.g., "user", "assistant") and **content** (e.g., text, multimodal data) with additional metadata that varies depending on the chat model provider.
LangChain provides a unified message format that can be used across chat models, allowing users to work with different chat models without worrying about the specific details of the message format used by each model provider.
## What is inside a message?​
A message typically consists of the following pieces of information:
  * **Role** : The role of the message (e.g., "user", "assistant").
  * **Content** : The content of the message (e.g., text, multimodal data).
  * Additional metadata: id, name, token usage and other model-specific metadata.


### Role​
Roles are used to distinguish between different types of messages in a conversation and help the chat model understand how to respond to a given sequence of messages.
**Role**| **Description**  
---|---  
**system**|  Used to tell the chat model how to behave and provide additional context. Not supported by all chat model providers.  
**user**|  Represents input from a user interacting with the model, usually in the form of text or other interactive input.  
**assistant**|  Represents a response from the model, which can include text or a request to invoke tools.  
**tool**|  A message used to pass the results of a tool invocation back to the model after external data or processing has been retrieved. Used with chat models that support tool calling.  
**function** (legacy)| This is a legacy role, corresponding to OpenAI's legacy function-calling API. **tool** role should be used instead.  
### Content​
The content of a message text or a list of dictionaries representing multimodal data (e.g., images, audio, video). The exact format of the content can vary between different chat model providers.
Currently, most chat models support text as the primary content type, with some models also supporting multimodal data. However, support for multimodal data is still limited across most chat model providers.
For more information see:
  * SystemMessage -- for content which should be passed to direct the conversation
  * HumanMessage -- for content in the input from the user.
  * AIMessage -- for content in the response from the model.
  * Multimodality -- for more information on multimodal content.


### Other Message Data​
Depending on the chat model provider, messages can include other data such as:
  * **ID** : An optional unique identifier for the message.
  * **Name** : An optional `name` property which allows differentiate between different entities/speakers with the same role. Not all models support this!
  * **Metadata** : Additional information about the message, such as timestamps, token usage, etc.
  * **Tool Calls** : A request made by the model to call one or more tools> See tool calling for more information.


## Conversation Structure​
The sequence of messages into a chat model should follow a specific structure to ensure that the chat model can generate a valid response.
For example, a typical conversation structure might look like this:
  1. **User Message** : "Hello, how are you?"
  2. **Assistant Message** : "I'm doing well, thank you for asking."
  3. **User Message** : "Can you tell me a joke?"
  4. **Assistant Message** : "Sure! Why did the scarecrow win an award? Because he was outstanding in his field!"


Please read the chat history guide for more information on managing chat history and ensuring that the conversation structure is correct.
## LangChain Messages​
LangChain provides a unified message format that can be used across all chat models, allowing users to work with different chat models without worrying about the specific details of the message format used by each model provider.
LangChain messages are Python objects that subclass from a BaseMessage.
The five main message types are:
  * SystemMessage: corresponds to **system** role
  * HumanMessage: corresponds to **user** role
  * AIMessage: corresponds to **assistant** role
  * AIMessageChunk: corresponds to **assistant** role, used for streaming responses
  * ToolMessage: corresponds to **tool** role


Other important messages include:
  * RemoveMessage -- does not correspond to any role. This is an abstraction, mostly used in LangGraph to manage chat history.
  * **Legacy** FunctionMessage: corresponds to the **function** role in OpenAI's **legacy** function-calling API.


You can find more information about **messages** in the API Reference.
### SystemMessage​
A `SystemMessage` is used to prime the behavior of the AI model and provide additional context, such as instructing the model to adopt a specific persona or setting the tone of the conversation (e.g., "This is a conversation about cooking").
Different chat providers may support system message in one of the following ways:
  * **Through a "system" message role** : In this case, a system message is included as part of the message sequence with the role explicitly set as "system."
  * **Through a separate API parameter for system instructions** : Instead of being included as a message, system instructions are passed via a dedicated API parameter.
  * **No support for system messages** : Some models do not support system messages at all.


Most major chat model providers support system instructions via either a chat message or a separate API parameter. LangChain will automatically adapt based on the provider’s capabilities. If the provider supports a separate API parameter for system instructions, LangChain will extract the content of a system message and pass it through that parameter.
If no system message is supported by the provider, in most cases LangChain will attempt to incorporate the system message's content into a HumanMessage or raise an exception if that is not possible. However, this behavior is not yet consistently enforced across all implementations, and if using a less popular implementation of a chat model (e.g., an implementation from the `langchain-community` package) it is recommended to check the specific documentation for that model.
### HumanMessage​
The `HumanMessage` corresponds to the **"user"** role. A human message represents input from a user interacting with the model.
#### Text Content​
Most chat models expect the user input to be in the form of text.
```
from langchain_core.messages import HumanMessagemodel.invoke([HumanMessage(content="Hello, how are you?")])
```

**API Reference:**HumanMessage
tip
When invoking a chat model with a string as input, LangChain will automatically convert the string into a `HumanMessage` object. This is mostly useful for quick testing.
```
model.invoke("Hello, how are you?")
```

#### Multi-modal Content​
Some chat models accept multimodal inputs, such as images, audio, video, or files like PDFs.
Please see the multimodality guide for more information.
### AIMessage​
`AIMessage` is used to represent a message with the role **"assistant"**. This is the response from the model, which can include text or a request to invoke tools. It could also include other media types like images, audio, or video -- though this is still uncommon at the moment.
```
from langchain_core.messages import HumanMessageai_message = model.invoke([HumanMessage("Tell me a joke")])ai_message # <-- AIMessage
```

**API Reference:**HumanMessage
An `AIMessage` has the following attributes. The attributes which are **standardized** are the ones that LangChain attempts to standardize across different chat model providers. **raw** fields are specific to the model provider and may vary.
Attribute| Standardized/Raw| Description  
---|---|---  
`content`| Raw| Usually a string, but can be a list of content blocks. See content for details.  
`tool_calls`| Standardized| Tool calls associated with the message. See tool calling for details.  
`invalid_tool_calls`| Standardized| Tool calls with parsing errors associated with the message. See tool calling for details.  
`usage_metadata`| Standardized| Usage metadata for a message, such as token counts. See Usage Metadata API Reference.  
`id`| Standardized| An optional unique identifier for the message, ideally provided by the provider/model that created the message.  
`response_metadata`| Raw| Response metadata, e.g., response headers, logprobs, token counts.  
#### content​
The **content** property of an `AIMessage` represents the response generated by the chat model.
The content is either:
  * **text** -- the norm for virtually all chat models.
  * A **list of dictionaries** -- Each dictionary represents a content block and is associated with a `type`. 
    * Used by Anthropic for surfacing agent thought process when doing tool calling.
    * Used by OpenAI for audio outputs. Please see multi-modal content for more information.


important
The **content** property is **not** standardized across different chat model providers, mostly because there are still few examples to generalize from.
### AIMessageChunk​
It is common to stream responses for the chat model as they are being generated, so the user can see the response in real-time instead of waiting for the entire response to be generated before displaying it.
It is returned from the `stream`, `astream` and `astream_events` methods of the chat model.
For example,
```
for chunk in model.stream([HumanMessage("what color is the sky?")]):print(chunk)
```

`AIMessageChunk` follows nearly the same structure as `AIMessage`, but uses a different ToolCallChunk to be able to stream tool calling in a standardized manner.
#### Aggregating​
`AIMessageChunks` support the `+` operator to merge them into a single `AIMessage`. This is useful when you want to display the final response to the user.
```
ai_message = chunk1 + chunk2 + chunk3 +...
```

### ToolMessage​
This represents a message with role "tool", which contains the result of calling a tool. In addition to `role` and `content`, this message has:
  * a `tool_call_id` field which conveys the id of the call to the tool that was called to produce this result.
  * an `artifact` field which can be used to pass along arbitrary artifacts of the tool execution which are useful to track but which should not be sent to the model.


Please see tool calling for more information.
### RemoveMessage​
This is a special message type that does not correspond to any roles. It is used for managing chat history in LangGraph.
Please see the following for more information on how to use the `RemoveMessage`:
  * Memory conceptual guide
  * How to delete messages


### (Legacy) FunctionMessage​
This is a legacy message type, corresponding to OpenAI's legacy function-calling API. `ToolMessage` should be used instead to correspond to the updated tool-calling API.
## OpenAI Format​
### Inputs​
Chat models also accept OpenAI's format as **inputs** to chat models:
```
chat_model.invoke([{"role":"user","content":"Hello, how are you?",},{"role":"assistant","content":"I'm doing well, thank you for asking.",},{"role":"user","content":"Can you tell me a joke?",}])
```

### Outputs​
At the moment, the output of the model will be in terms of LangChain messages, so you will need to convert the output to the OpenAI format if you need OpenAI format for the output as well.
The convert_to_openai_messages utility function can be used to convert from LangChain messages to OpenAI format.
#### Was this page helpful?
  * Overview
  * What is inside a message?
    * Role
    * Content
    * Other Message Data
  * Conversation Structure
  * LangChain Messages
    * SystemMessage
    * HumanMessage
    * AIMessage
    * AIMessageChunk
    * ToolMessage
    * RemoveMessage
    * (Legacy) FunctionMessage
  * OpenAI Format
    * Inputs
    * Outputs


