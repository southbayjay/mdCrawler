Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
Prerequisites
  * Messages
  * Chat models
  * Tool calling


Chat history is a record of the conversation between the user and the chat model. It is used to maintain context and state throughout the conversation. The chat history is sequence of messages, each of which is associated with a specific role, such as "user", "assistant", "system", or "tool".
## Conversation patterns​
![Conversation patterns](https://python.langchain.com/assets/images/conversation_patterns-0e4c2311b54fae7412f74b1408615432.png)
Most conversations start with a **system message** that sets the context for the conversation. This is followed by a **user message** containing the user's input, and then an **assistant message** containing the model's response.
The **assistant** may respond directly to the user or if configured with tools request that a tool be invoked to perform a specific task.
A full conversation often involves a combination of two patterns of alternating messages:
  1. The **user** and the **assistant** representing a back-and-forth conversation.
  2. The **assistant** and **tool messages** representing an "agentic" workflow where the assistant is invoking tools to perform specific tasks.


## Managing chat history​
Since chat models have a maximum limit on input size, it's important to manage chat history and trim it as needed to avoid exceeding the context window.
While processing chat history, it's essential to preserve a correct conversation structure.
Key guidelines for managing chat history:
  * The conversation should follow one of these structures: 
    * The first message is either a "user" message or a "system" message, followed by a "user" and then an "assistant" message.
    * The last message should be either a "user" message or a "tool" message containing the result of a tool call.
  * When using tool calling, a "tool" message should only follow an "assistant" message that requested the tool invocation.


tip
Understanding correct conversation structure is essential for being able to properly implement memory in chat models.
## Related resources​
  * How to trim messages
  * Memory guide for information on implementing short-term and long-term memory in chat models using LangGraph.


#### Was this page helpful?
  * Conversation patterns
  * Managing chat history
  * Related resources


