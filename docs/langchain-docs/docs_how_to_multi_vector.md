Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
It can often be useful to store multiple vectors per document. There are multiple use cases where this is beneficial. For example, we can embed multiple chunks of a document and associate those embeddings with the parent document, allowing retriever hits on the chunks to return the larger document.
LangChain implements a base MultiVectorRetriever, which simplifies this process. Much of the complexity lies in how to create the multiple vectors per document. This notebook covers some of the common ways to create those vectors and use the `MultiVectorRetriever`.
The methods to create multiple vectors per document include:
  * Smaller chunks: split a document into smaller chunks, and embed those (this is ParentDocumentRetriever).
  * Summary: create a summary for each document, embed that along with (or instead of) the document.
  * Hypothetical questions: create hypothetical questions that each document would be appropriate to answer, embed those along with (or instead of) the document.


Note that this also enables another method of adding embeddings - manually. This is useful because you can explicitly add questions or queries that should lead to a document being recovered, giving you more control.
Below we walk through an example. First we instantiate some documents. We will index them in an (in-memory) Chroma vector store using OpenAI embeddings, but any LangChain vector store or embeddings model will suffice.
```
%pip install --upgrade --quiet langchain-chroma langchain langchain-openai >/dev/null
```

```
from langchain.storage import InMemoryByteStorefrom langchain_chroma import Chromafrom langchain_community.document_loaders import TextLoaderfrom langchain_openai import OpenAIEmbeddingsfrom langchain_text_splitters import RecursiveCharacterTextSplitterloaders =[  TextLoader("paul_graham_essay.txt"),  TextLoader("state_of_the_union.txt"),]docs =[]for loader in loaders:  docs.extend(loader.load())text_splitter = RecursiveCharacterTextSplitter(chunk_size=10000)docs = text_splitter.split_documents(docs)# The vectorstore to use to index the child chunksvectorstore = Chroma(  collection_name="full_documents", embedding_function=OpenAIEmbeddings())
```

**API Reference:**InMemoryByteStore | TextLoader | OpenAIEmbeddings | RecursiveCharacterTextSplitter
## Smaller chunks​
Often times it can be useful to retrieve larger chunks of information, but embed smaller chunks. This allows for embeddings to capture the semantic meaning as closely as possible, but for as much context as possible to be passed downstream. Note that this is what the ParentDocumentRetriever does. Here we show what is going on under the hood.
We will make a distinction between the vector store, which indexes embeddings of the (sub) documents, and the document store, which houses the "parent" documents and associates them with an identifier.
```
import uuidfrom langchain.retrievers.multi_vector import MultiVectorRetriever# The storage layer for the parent documentsstore = InMemoryByteStore()id_key ="doc_id"# The retriever (empty to start)retriever = MultiVectorRetriever(  vectorstore=vectorstore,  byte_store=store,  id_key=id_key,)doc_ids =[str(uuid.uuid4())for _ in docs]
```

**API Reference:**MultiVectorRetriever
We next generate the "sub" documents by splitting the original documents. Note that we store the document identifier in the `metadata` of the corresponding Document object.
```
# The splitter to use to create smaller chunkschild_text_splitter = RecursiveCharacterTextSplitter(chunk_size=400)sub_docs =[]for i, doc inenumerate(docs):  _id = doc_ids[i]  _sub_docs = child_text_splitter.split_documents([doc])for _doc in _sub_docs:    _doc.metadata[id_key]= _id  sub_docs.extend(_sub_docs)
```

Finally, we index the documents in our vector store and document store:
```
retriever.vectorstore.add_documents(sub_docs)retriever.docstore.mset(list(zip(doc_ids, docs)))
```

The vector store alone will retrieve small chunks:
```
retriever.vectorstore.similarity_search("justice breyer")[0]
```

```
Document(page_content='Tonight, I’d like to honor someone who has dedicated his life to serve this country: Justice Stephen Breyer—an Army veteran, Constitutional scholar, and retiring Justice of the United States Supreme Court. Justice Breyer, thank you for your service. \n\nOne of the most serious constitutional responsibilities a President has is nominating someone to serve on the United States Supreme Court.', metadata={'doc_id': '064eca46-a4c4-4789-8e3b-583f9597e54f', 'source': 'state_of_the_union.txt'})
```

Whereas the retriever will return the larger parent document:
```
len(retriever.invoke("justice breyer")[0].page_content)
```

```
9875
```

The default search type the retriever performs on the vector database is a similarity search. LangChain vector stores also support searching via Max Marginal Relevance. This can be controlled via the `search_type` parameter of the retriever:
```
from langchain.retrievers.multi_vector import SearchTyperetriever.search_type = SearchType.mmrlen(retriever.invoke("justice breyer")[0].page_content)
```

**API Reference:**SearchType
```
9875
```

## Associating summaries with a document for retrieval​
A summary may be able to distill more accurately what a chunk is about, leading to better retrieval. Here we show how to create summaries, and then embed those.
We construct a simple chain that will receive an input Document object and generate a summary using a LLM.
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
import uuidfrom langchain_core.documents import Documentfrom langchain_core.output_parsers import StrOutputParserfrom langchain_core.prompts import ChatPromptTemplatechain =({"doc":lambda x: x.page_content}| ChatPromptTemplate.from_template("Summarize the following document:\n\n{doc}")| llm| StrOutputParser())
```

**API Reference:**Document | StrOutputParser | ChatPromptTemplate
Note that we can batch the chain across documents:
```
summaries = chain.batch(docs,{"max_concurrency":5})
```

We can then initialize a `MultiVectorRetriever` as before, indexing the summaries in our vector store, and retaining the original documents in our document store:
```
# The vectorstore to use to index the child chunksvectorstore = Chroma(collection_name="summaries", embedding_function=OpenAIEmbeddings())# The storage layer for the parent documentsstore = InMemoryByteStore()id_key ="doc_id"# The retriever (empty to start)retriever = MultiVectorRetriever(  vectorstore=vectorstore,  byte_store=store,  id_key=id_key,)doc_ids =[str(uuid.uuid4())for _ in docs]summary_docs =[  Document(page_content=s, metadata={id_key: doc_ids[i]})for i, s inenumerate(summaries)]retriever.vectorstore.add_documents(summary_docs)retriever.docstore.mset(list(zip(doc_ids, docs)))
```

```
# # We can also add the original chunks to the vectorstore if we so want# for i, doc in enumerate(docs):#   doc.metadata[id_key] = doc_ids[i]# retriever.vectorstore.add_documents(docs)
```

Querying the vector store will return summaries:
```
sub_docs = retriever.vectorstore.similarity_search("justice breyer")sub_docs[0]
```

```
Document(page_content="President Biden recently nominated Judge Ketanji Brown Jackson to serve on the United States Supreme Court, emphasizing her qualifications and broad support. The President also outlined a plan to secure the border, fix the immigration system, protect women's rights, support LGBTQ+ Americans, and advance mental health services. He highlighted the importance of bipartisan unity in passing legislation, such as the Violence Against Women Act. The President also addressed supporting veterans, particularly those impacted by exposure to burn pits, and announced plans to expand benefits for veterans with respiratory cancers. Additionally, he proposed a plan to end cancer as we know it through the Cancer Moonshot initiative. President Biden expressed optimism about the future of America and emphasized the strength of the American people in overcoming challenges.", metadata={'doc_id': '84015b1b-980e-400a-94d8-cf95d7e079bd'})
```

Whereas the retriever will return the larger source document:
```
retrieved_docs = retriever.invoke("justice breyer")len(retrieved_docs[0].page_content)
```

```
9194
```

## Hypothetical Queries​
An LLM can also be used to generate a list of hypothetical questions that could be asked of a particular document, which might bear close semantic similarity to relevant queries in a RAG application. These questions can then be embedded and associated with the documents to improve retrieval.
Below, we use the with_structured_output method to structure the LLM output into a list of strings.
```
from typing import Listfrom pydantic import BaseModel, FieldclassHypotheticalQuestions(BaseModel):"""Generate hypothetical questions."""  questions: List[str]= Field(..., description="List of questions")chain =({"doc":lambda x: x.page_content}# Only asking for 3 hypothetical questions, but this could be adjusted| ChatPromptTemplate.from_template("Generate a list of exactly 3 hypothetical questions that the below document could be used to answer:\n\n{doc}")| ChatOpenAI(max_retries=0, model="gpt-4o").with_structured_output(    HypotheticalQuestions)|(lambda x: x.questions))
```

Invoking the chain on a single document demonstrates that it outputs a list of questions:
```
chain.invoke(docs[0])
```

```
["What impact did the IBM 1401 have on the author's early programming experiences?", "How did the transition from using the IBM 1401 to microcomputers influence the author's programming journey?", "What role did Lisp play in shaping the author's understanding and approach to AI?"]
```

We can batch then batch the chain over all documents and assemble our vector store and document store as before:
```
# Batch chain over documents to generate hypothetical questionshypothetical_questions = chain.batch(docs,{"max_concurrency":5})# The vectorstore to use to index the child chunksvectorstore = Chroma(  collection_name="hypo-questions", embedding_function=OpenAIEmbeddings())# The storage layer for the parent documentsstore = InMemoryByteStore()id_key ="doc_id"# The retriever (empty to start)retriever = MultiVectorRetriever(  vectorstore=vectorstore,  byte_store=store,  id_key=id_key,)doc_ids =[str(uuid.uuid4())for _ in docs]# Generate Document objects from hypothetical questionsquestion_docs =[]for i, question_list inenumerate(hypothetical_questions):  question_docs.extend([Document(page_content=s, metadata={id_key: doc_ids[i]})for s in question_list])retriever.vectorstore.add_documents(question_docs)retriever.docstore.mset(list(zip(doc_ids, docs)))
```

Note that querying the underlying vector store will retrieve hypothetical questions that are semantically similar to the input query:
```
sub_docs = retriever.vectorstore.similarity_search("justice breyer")sub_docs
```

```
[Document(page_content='What might be the potential benefits of nominating Circuit Court of Appeals Judge Ketanji Brown Jackson to the United States Supreme Court?', metadata={'doc_id': '43292b74-d1b8-4200-8a8b-ea0cb57fbcdb'}), Document(page_content='How might the Bipartisan Infrastructure Law impact the economic competition between the U.S. and China?', metadata={'doc_id': '66174780-d00c-4166-9791-f0069846e734'}), Document(page_content='What factors led to the creation of Y Combinator?', metadata={'doc_id': '72003c4e-4cc9-4f09-a787-0b541a65b38c'}), Document(page_content='How did the ability to publish essays online change the landscape for writers and thinkers?', metadata={'doc_id': 'e8d2c648-f245-4bcc-b8d3-14e64a164b64'})]
```

And invoking the retriever will return the corresponding document:
```
retrieved_docs = retriever.invoke("justice breyer")len(retrieved_docs[0].page_content)
```

```
9194
```

#### Was this page helpful?
  * Smaller chunks
  * Associating summaries with a document for retrieval
  * Hypothetical Queries


