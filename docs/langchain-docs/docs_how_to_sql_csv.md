Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
LLMs are great for building question-answering systems over various types of data sources. In this section we'll go over how to build Q&A systems over data stored in a CSV file(s). Like working with SQL databases, the key to working with CSV files is to give an LLM access to tools for querying and interacting with the data. The two main ways to do this are to either:
  * **RECOMMENDED** : Load the CSV(s) into a SQL database, and use the approaches outlined in the SQL tutorial.
  * Give the LLM access to a Python environment where it can use libraries like Pandas to interact with the data.


We will cover both approaches in this guide.
## ⚠️ Security note ⚠️​
Both approaches mentioned above carry significant risks. Using SQL requires executing model-generated SQL queries. Using a library like Pandas requires letting the model execute Python code. Since it is easier to tightly scope SQL connection permissions and sanitize SQL queries than it is to sandbox Python environments, **we HIGHLY recommend interacting with CSV data via SQL.** For more on general security best practices, see here.
## Setup​
Dependencies for this guide:
```
%pip install -qU langchain langchain-openai langchain-community langchain-experimental pandas
```

Set required environment variables:
```
# Using LangSmith is recommended but not required. Uncomment below lines to use.# import os# os.environ["LANGSMITH_TRACING"] = "true"# os.environ["LANGSMITH_API_KEY"] = getpass.getpass()
```

Download the Titanic dataset if you don't already have it:
```
!wget https://web.stanford.edu/class/archive/cs/cs109/cs109.1166/stuff/titanic.csv -O titanic.csv
```

```
import pandas as pddf = pd.read_csv("titanic.csv")print(df.shape)print(df.columns.tolist())
```

```
(887, 8)['Survived', 'Pclass', 'Name', 'Sex', 'Age', 'Siblings/Spouses Aboard', 'Parents/Children Aboard', 'Fare']
```

## SQL​
Using SQL to interact with CSV data is the recommended approach because it is easier to limit permissions and sanitize queries than with arbitrary Python.
Most SQL databases make it easy to load a CSV file in as a table (DuckDB, SQLite, etc.). Once you've done this you can use all of the chain and agent-creating techniques outlined in the SQL tutorial. Here's a quick example of how we might do this with SQLite:
```
from langchain_community.utilities import SQLDatabasefrom sqlalchemy import create_engineengine = create_engine("sqlite:///titanic.db")df.to_sql("titanic", engine, index=False)
```

**API Reference:**SQLDatabase
```
887
```

```
db = SQLDatabase(engine=engine)print(db.dialect)print(db.get_usable_table_names())print(db.run("SELECT * FROM titanic WHERE Age < 2;"))
```

```
sqlite['titanic'][(1, 2, 'Master. Alden Gates Caldwell', 'male', 0.83, 0, 2, 29.0), (0, 3, 'Master. Eino Viljami Panula', 'male', 1.0, 4, 1, 39.6875), (1, 3, 'Miss. Eleanor Ileen Johnson', 'female', 1.0, 1, 1, 11.1333), (1, 2, 'Master. Richard F Becker', 'male', 1.0, 2, 1, 39.0), (1, 1, 'Master. Hudson Trevor Allison', 'male', 0.92, 1, 2, 151.55), (1, 3, 'Miss. Maria Nakid', 'female', 1.0, 0, 2, 15.7417), (0, 3, 'Master. Sidney Leonard Goodwin', 'male', 1.0, 5, 2, 46.9), (1, 3, 'Miss. Helene Barbara Baclini', 'female', 0.75, 2, 1, 19.2583), (1, 3, 'Miss. Eugenie Baclini', 'female', 0.75, 2, 1, 19.2583), (1, 2, 'Master. Viljo Hamalainen', 'male', 0.67, 1, 1, 14.5), (1, 3, 'Master. Bertram Vere Dean', 'male', 1.0, 1, 2, 20.575), (1, 3, 'Master. Assad Alexander Thomas', 'male', 0.42, 0, 1, 8.5167), (1, 2, 'Master. Andre Mallet', 'male', 1.0, 0, 2, 37.0042), (1, 2, 'Master. George Sibley Richards', 'male', 0.83, 1, 1, 18.75)]
```

And create a SQL agent to interact with it:
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
from langchain_community.agent_toolkits import create_sql_agentagent_executor = create_sql_agent(llm, db=db, agent_type="openai-tools", verbose=True)
```

**API Reference:**create_sql_agent
```
agent_executor.invoke({"input":"what's the average age of survivors"})
```

```
[1m> Entering new SQL Agent Executor chain...[0m[32;1m[1;3mInvoking: `sql_db_list_tables` with `{}`[0m[38;5;200m[1;3mtitanic[0m[32;1m[1;3mInvoking: `sql_db_schema` with `{'table_names': 'titanic'}`[0m[33;1m[1;3mCREATE TABLE titanic (	"Survived" BIGINT, 	"Pclass" BIGINT, 	"Name" TEXT, 	"Sex" TEXT, 	"Age" FLOAT, 	"Siblings/Spouses Aboard" BIGINT, 	"Parents/Children Aboard" BIGINT, 	"Fare" FLOAT)/*3 rows from titanic table:Survived	Pclass	Name	Sex	Age	Siblings/Spouses Aboard	Parents/Children Aboard	Fare0	3	Mr. Owen Harris Braund	male	22.0	1	0	7.251	1	Mrs. John Bradley (Florence Briggs Thayer) Cumings	female	38.0	1	0	71.28331	3	Miss. Laina Heikkinen	female	26.0	0	0	7.925*/[0m[32;1m[1;3mInvoking: `sql_db_query` with `{'query': 'SELECT AVG(Age) AS Average_Age FROM titanic WHERE Survived = 1'}`[0m[36;1m[1;3m[(28.408391812865496,)][0m[32;1m[1;3mThe average age of survivors in the Titanic dataset is approximately 28.41 years.[0m[1m> Finished chain.[0m
```

```
{'input': "what's the average age of survivors", 'output': 'The average age of survivors in the Titanic dataset is approximately 28.41 years.'}
```

This approach easily generalizes to multiple CSVs, since we can just load each of them into our database as its own table. See the Multiple CSVs section below.
## Pandas​
Instead of SQL we can also use data analysis libraries like pandas and the code generating abilities of LLMs to interact with CSV data. Again, **this approach is not fit for production use cases unless you have extensive safeguards in place**. For this reason, our code-execution utilities and constructors live in the `langchain-experimental` package.
### Chain​
Most LLMs have been trained on enough pandas Python code that they can generate it just by being asked to:
```
ai_msg = llm.invoke("I have a pandas DataFrame 'df' with columns 'Age' and 'Fare'. Write code to compute the correlation between the two columns. Return Markdown for a Python code snippet and nothing else.")print(ai_msg.content)
```

```
\`\`\`pythoncorrelation = df['Age'].corr(df['Fare'])correlation\`\`\`
```

We can combine this ability with a Python-executing tool to create a simple data analysis chain. We'll first want to load our CSV table as a dataframe, and give the tool access to this dataframe:
```
import pandas as pdfrom langchain_core.prompts import ChatPromptTemplatefrom langchain_experimental.tools import PythonAstREPLTooldf = pd.read_csv("titanic.csv")tool = PythonAstREPLTool(locals={"df": df})tool.invoke("df['Fare'].mean()")
```

**API Reference:**ChatPromptTemplate | PythonAstREPLTool
```
32.30542018038331
```

To help enforce proper use of our Python tool, we'll using tool calling:
```
llm_with_tools = llm.bind_tools([tool], tool_choice=tool.name)response = llm_with_tools.invoke("I have a dataframe 'df' and want to know the correlation between the 'Age' and 'Fare' columns")response
```

```
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_SBrK246yUbdnJemXFC8Iod05', 'function': {'arguments': '{"query":"df.corr()[\'Age\'][\'Fare\']"}', 'name': 'python_repl_ast'}, 'type': 'function'}]}, response_metadata={'token_usage': {'completion_tokens': 13, 'prompt_tokens': 125, 'total_tokens': 138}, 'model_name': 'gpt-3.5-turbo', 'system_fingerprint': 'fp_3b956da36b', 'finish_reason': 'stop', 'logprobs': None}, id='run-1fd332ba-fa72-4351-8182-d464e7368311-0', tool_calls=[{'name': 'python_repl_ast', 'args': {'query': "df.corr()['Age']['Fare']"}, 'id': 'call_SBrK246yUbdnJemXFC8Iod05'}])
```

```
response.tool_calls
```

```
[{'name': 'python_repl_ast', 'args': {'query': "df.corr()['Age']['Fare']"}, 'id': 'call_SBrK246yUbdnJemXFC8Iod05'}]
```

We'll add a tools output parser to extract the function call as a dict:
```
from langchain_core.output_parsers.openai_tools import JsonOutputKeyToolsParserparser = JsonOutputKeyToolsParser(key_name=tool.name, first_tool_only=True)(llm_with_tools | parser).invoke("I have a dataframe 'df' and want to know the correlation between the 'Age' and 'Fare' columns")
```

**API Reference:**JsonOutputKeyToolsParser
```
{'query': "df[['Age', 'Fare']].corr()"}
```

And combine with a prompt so that we can just specify a question without needing to specify the dataframe info every invocation:
```
system =f"""You have access to a pandas dataframe `df`. \Here is the output of `df.head().to_markdown()`:\`\`\`{df.head().to_markdown()}\`\`\`Given a user question, write the Python code to answer it. \Return ONLY the valid Python code and nothing else. \Don't assume you have access to any libraries other than built-in Python ones and pandas."""prompt = ChatPromptTemplate.from_messages([("system", system),("human","{question}")])code_chain = prompt | llm_with_tools | parsercode_chain.invoke({"question":"What's the correlation between age and fare"})
```

```
{'query': "df[['Age', 'Fare']].corr()"}
```

And lastly we'll add our Python tool so that the generated code is actually executed:
```
chain = prompt | llm_with_tools | parser | toolchain.invoke({"question":"What's the correlation between age and fare"})
```

```
0.11232863699941621
```

And just like that we have a simple data analysis chain. We can take a peak at the intermediate steps by looking at the LangSmith trace: https://smith.langchain.com/public/b1309290-7212-49b7-bde2-75b39a32b49a/r
We could add an additional LLM call at the end to generate a conversational response, so that we're not just responding with the tool output. For this we'll want to add a chat history `MessagesPlaceholder` to our prompt:
```
from operator import itemgetterfrom langchain_core.messages import ToolMessagefrom langchain_core.output_parsers import StrOutputParserfrom langchain_core.prompts import MessagesPlaceholderfrom langchain_core.runnables import RunnablePassthroughsystem =f"""You have access to a pandas dataframe `df`. \Here is the output of `df.head().to_markdown()`:\`\`\`{df.head().to_markdown()}\`\`\`Given a user question, write the Python code to answer it. \Don't assume you have access to any libraries other than built-in Python ones and pandas.Respond directly to the question once you have enough information to answer it."""prompt = ChatPromptTemplate.from_messages([("system",      system,),("human","{question}"),# This MessagesPlaceholder allows us to optionally append an arbitrary number of messages# at the end of the prompt using the 'chat_history' arg.    MessagesPlaceholder("chat_history", optional=True),])def_get_chat_history(x:dict)->list:"""Parse the chain output up to this point into a list of chat history messages to insert in the prompt."""  ai_msg = x["ai_msg"]  tool_call_id = x["ai_msg"].additional_kwargs["tool_calls"][0]["id"]  tool_msg = ToolMessage(tool_call_id=tool_call_id, content=str(x["tool_output"]))return[ai_msg, tool_msg]chain =(  RunnablePassthrough.assign(ai_msg=prompt | llm_with_tools).assign(tool_output=itemgetter("ai_msg")| parser | tool).assign(chat_history=_get_chat_history).assign(response=prompt | llm | StrOutputParser()).pick(["tool_output","response"]))
```

**API Reference:**ToolMessage | StrOutputParser | MessagesPlaceholder | RunnablePassthrough
```
chain.invoke({"question":"What's the correlation between age and fare"})
```

```
{'tool_output': 0.11232863699941616, 'response': 'The correlation between age and fare is approximately 0.1123.'}
```

Here's the LangSmith trace for this run: https://smith.langchain.com/public/14e38d70-45b1-4b81-8477-9fd2b7c07ea6/r
### Agent​
For complex questions it can be helpful for an LLM to be able to iteratively execute code while maintaining the inputs and outputs of its previous executions. This is where Agents come into play. They allow an LLM to decide how many times a tool needs to be invoked and keep track of the executions it's made so far. The create_pandas_dataframe_agent is a built-in agent that makes it easy to work with dataframes:
```
from langchain_experimental.agents import create_pandas_dataframe_agentagent = create_pandas_dataframe_agent(  llm, df, agent_type="openai-tools", verbose=True, allow_dangerous_code=True)agent.invoke({"input":"What's the correlation between age and fare? is that greater than the correlation between fare and survival?"})
```

**API Reference:**create_pandas_dataframe_agent
```
[1m> Entering new AgentExecutor chain...[0m[32;1m[1;3mInvoking: `python_repl_ast` with `{'query': "df[['Age', 'Fare']].corr().iloc[0,1]"}`[0m[36;1m[1;3m0.11232863699941621[0m[32;1m[1;3mInvoking: `python_repl_ast` with `{'query': "df[['Fare', 'Survived']].corr().iloc[0,1]"}`[0m[36;1m[1;3m0.2561785496289603[0m[32;1m[1;3mThe correlation between Age and Fare is approximately 0.112, and the correlation between Fare and Survival is approximately 0.256.Therefore, the correlation between Fare and Survival (0.256) is greater than the correlation between Age and Fare (0.112).[0m[1m> Finished chain.[0m
```

```
{'input': "What's the correlation between age and fare? is that greater than the correlation between fare and survival?", 'output': 'The correlation between Age and Fare is approximately 0.112, and the correlation between Fare and Survival is approximately 0.256.\n\nTherefore, the correlation between Fare and Survival (0.256) is greater than the correlation between Age and Fare (0.112).'}
```

Here's the LangSmith trace for this run: https://smith.langchain.com/public/6a86aee2-4f22-474a-9264-bd4c7283e665/r
### Multiple CSVs​
To handle multiple CSVs (or dataframes) we just need to pass multiple dataframes to our Python tool. Our `create_pandas_dataframe_agent` constructor can do this out of the box, we can pass in a list of dataframes instead of just one. If we're constructing a chain ourselves, we can do something like:
```
df_1 = df[["Age","Fare"]]df_2 = df[["Fare","Survived"]]tool = PythonAstREPLTool(locals={"df_1": df_1,"df_2": df_2})llm_with_tool = llm.bind_tools(tools=[tool], tool_choice=tool.name)df_template ="""\`\`\`python{df_name}.head().to_markdown()>>> {df_head}\`\`\`"""df_context ="\n\n".join(  df_template.format(df_head=_df.head().to_markdown(), df_name=df_name)for _df, df_name in[(df_1,"df_1"),(df_2,"df_2")])system =f"""You have access to a number of pandas dataframes. \Here is a sample of rows from each dataframe and the python code that was used to generate the sample:{df_context}Given a user question about the dataframes, write the Python code to answer it. \Don't assume you have access to any libraries other than built-in Python ones and pandas. \Make sure to refer only to the variables mentioned above."""prompt = ChatPromptTemplate.from_messages([("system", system),("human","{question}")])chain = prompt | llm_with_tool | parser | toolchain.invoke({"question":"return the difference in the correlation between age and fare and the correlation between fare and survival"})
```

```
0.14384991262954416
```

Here's the LangSmith trace for this run: https://smith.langchain.com/public/cc2a7d7f-7c5a-4e77-a10c-7b5420fcd07f/r
### Sandboxed code execution​
There are a number of tools like E2B and Bearly that provide sandboxed environments for Python code execution, to allow for safer code-executing chains and agents.
## Next steps​
For more advanced data analysis applications we recommend checking out:
  * SQL tutorial: Many of the challenges of working with SQL db's and CSV's are generic to any structured data type, so it's useful to read the SQL techniques even if you're using Pandas for CSV data analysis.
  * Tool use: Guides on general best practices when working with chains and agents that invoke tools
  * Agents: Understand the fundamentals of building LLM agents.
  * Integrations: Sandboxed envs like E2B and Bearly, utilities like SQLDatabase, related agents like Spark DataFrame agent.


#### Was this page helpful?
  * ⚠️ Security note ⚠️
  * Setup
  * SQL
  * Pandas
    * Chain
    * Agent
    * Multiple CSVs
    * Sandboxed code execution
  * Next steps


