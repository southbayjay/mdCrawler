Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
info
Head to Integrations for documentation on built-in integrations with 3rd-party vector stores.
One of the most common ways to store and search over unstructured data is to embed it and store the resulting embedding vectors, and then at query time to embed the unstructured query and retrieve the embedding vectors that are 'most similar' to the embedded query. A vector store takes care of storing embedded data and performing vector search for you.
## Get started​
This guide showcases basic functionality related to vector stores. A key part of working with vector stores is creating the vector to put in them, which is usually created via embeddings. Therefore, it is recommended that you familiarize yourself with the text embedding model interfaces before diving into this.
Before using the vectorstore at all, we need to load some data and initialize an embedding model.
We want to use OpenAIEmbeddings so we have to get the OpenAI API Key.
```
import osimport getpassos.environ['OPENAI_API_KEY']= getpass.getpass('OpenAI API Key:')
```

```
from langchain_community.document_loaders import TextLoaderfrom langchain_openai import OpenAIEmbeddingsfrom langchain_text_splitters import CharacterTextSplitter# Load the document, split it into chunks, embed each chunk and load it into the vector store.raw_documents = TextLoader('state_of_the_union.txt').load()text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)documents = text_splitter.split_documents(raw_documents)
```

**API Reference:**TextLoader | OpenAIEmbeddings | CharacterTextSplitter
There are many great vector store options, here are a few that are free, open-source, and run entirely on your local machine. Review all integrations for many great hosted offerings.
  * Chroma
  * FAISS
  * Lance


This walkthrough uses the `chroma` vector database, which runs on your local machine as a library.
```
pip install langchain-chroma
```

```
from langchain_chroma import Chromadb = Chroma.from_documents(documents, OpenAIEmbeddings())
```

This walkthrough uses the `FAISS` vector database, which makes use of the Facebook AI Similarity Search (FAISS) library.
```
pip install faiss-cpu
```

```
from langchain_community.vectorstores import FAISSdb = FAISS.from_documents(documents, OpenAIEmbeddings())
```

**API Reference:**FAISS
This notebook shows how to use functionality related to the LanceDB vector database based on the Lance data format.
```
pip install lancedb
```

```
from langchain_community.vectorstores import LanceDBimport lancedbdb = lancedb.connect("/tmp/lancedb")table = db.create_table("my_table",  data=[{"vector": embeddings.embed_query("Hello World"),"text":"Hello World","id":"1",}],  mode="overwrite",)db = LanceDB.from_documents(documents, OpenAIEmbeddings())
```

**API Reference:**LanceDB
## Similarity search​
All vectorstores expose a `similarity_search` method. This will take incoming documents, create an embedding of them, and then find all documents with the most similar embedding.
```
query ="What did the president say about Ketanji Brown Jackson"docs = db.similarity_search(query)print(docs[0].page_content)
```

```
  Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections.  Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service.  One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court.  And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

### Similarity search by vector​
It is also possible to do a search for documents similar to a given embedding vector using `similarity_search_by_vector` which accepts an embedding vector as a parameter instead of a string.
```
embedding_vector = OpenAIEmbeddings().embed_query(query)docs = db.similarity_search_by_vector(embedding_vector)print(docs[0].page_content)
```

```
  Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections.  Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service.  One of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court.  And I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.
```

## Async Operations​
Vector stores are usually run as a separate service that requires some IO operations, and therefore they might be called asynchronously. That gives performance benefits as you don't waste time waiting for responses from external services. That might also be important if you work with an asynchronous framework, such as FastAPI.
LangChain supports async operation on vector stores. All the methods might be called using their async counterparts, with the prefix `a`, meaning `async`.
```
docs =await db.asimilarity_search(query)docs
```

```
[Document(page_content='Tonight. I call on the Senate to: Pass the Freedom to Vote Act. Pass the John Lewis Voting Rights Act. And while you’re at it, pass the Disclose Act so Americans can know who is funding our elections. \n\nTonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court. \n\nAnd I did that 4 days ago, when I nominated Circuit Court of Appeals Judge Ketanji Brown Jackson. One of our nation’s top legal minds, who will continue Justice Breyer’s legacy of excellence.', metadata={'source': 'state_of_the_union.txt'}), Document(page_content='A former top litigator in private practice. A former federal public defender. And from a family of public school educators and police officers. A consensus builder. Since she’s been nominated, she’s received a broad range of support—from the Fraternal Order of Police to former judges appointed by Democrats and Republicans. \n\nAnd if we are to advance liberty and justice, we need to secure the Border and fix the immigration system. \n\nWe can do both. At our border, we’ve installed new technology like cutting-edge scanners to better detect drug smuggling. \n\nWe’ve set up joint patrols with Mexico and Guatemala to catch more human traffickers. \n\nWe’re putting in place dedicated immigration judges so families fleeing persecution and violence can have their cases heard faster. \n\nWe’re securing commitments and supporting partners in South and Central America to host more refugees and secure their own borders.', metadata={'source': 'state_of_the_union.txt'}), Document(page_content='And for our LGBTQ+ Americans, let’s finally get the bipartisan Equality Act to my desk. The onslaught of state laws targeting transgender Americans and their families is wrong. \n\nAs I said last year, especially to our younger transgender Americans, I will always have your back as your President, so you can be yourself and reach your God-given potential. \n\nWhile it often appears that we never agree, that isn’t true. I signed 80 bipartisan bills into law last year. From preventing government shutdowns to protecting Asian-Americans from still-too-common hate crimes to reforming military justice. \n\nAnd soon, we’ll strengthen the Violence Against Women Act that I first wrote three decades ago. It is important for us to show the nation that we can come together and do big things. \n\nSo tonight I’m offering a Unity Agenda for the Nation. Four big things we can do together. \n\nFirst, beat the opioid epidemic.', metadata={'source': 'state_of_the_union.txt'}), Document(page_content='Tonight, I’m announcing a crackdown on these companies overcharging American businesses and consumers. \n\nAnd as Wall Street firms take over more nursing homes, quality in those homes has gone down and costs have gone up. \n\nThat ends on my watch. \n\nMedicare is going to set higher standards for nursing homes and make sure your loved ones get the care they deserve and expect. \n\nWe’ll also cut costs and keep the economy going strong by giving workers a fair shot, provide more training and apprenticeships, hire them based on their skills not degrees. \n\nLet’s pass the Paycheck Fairness Act and paid leave. \n\nRaise the minimum wage to $15 an hour and extend the Child Tax Credit, so no one has to raise a family in poverty. \n\nLet’s increase Pell Grants and increase our historic support of HBCUs, and invest in what Jill—our First Lady who teaches full-time—calls America’s best-kept secret: community colleges.', metadata={'source': 'state_of_the_union.txt'})]
```

#### Was this page helpful?
  * Get started
  * Similarity search
    * Similarity search by vector
  * Async Operations


