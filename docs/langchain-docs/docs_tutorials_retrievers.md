Skip to main content
**Join us at Interrupt: The Agent AI Conference by LangChain on May 13 & 14 in San Francisco!**
On this page
![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)![Open on GitHub](https://img.shields.io/badge/Open%20on%20GitHub-grey?logo=github&logoColor=white)
This tutorial will familiarize you with LangChain's document loader, embedding, and vector store abstractions. These abstractions are designed to support retrieval of data-- from (vector) databases and other sources-- for integration with LLM workflows. They are important for applications that fetch data to be reasoned over as part of model inference, as in the case of retrieval-augmented generation, or RAG (see our RAG tutorial here).
Here we will build a search engine over a PDF document. This will allow us to retrieve passages in the PDF that are similar to an input query.
## Concepts​
This guide focuses on retrieval of text data. We will cover the following concepts:
  * Documents and document loaders;
  * Text splitters;
  * Embeddings;
  * Vector stores and retrievers.


## Setup​
### Jupyter Notebook​
This and other tutorials are perhaps most conveniently run in a Jupyter notebook. See here for instructions on how to install.
### Installation​
This tutorial requires the `langchain-community` and `pypdf` packages:
  * Pip
  * Conda


```
pip install langchain-community pypdf
```

```
conda install langchain-community pypdf -c conda-forge
```

For more details, see our Installation guide.
### LangSmith​
Many of the applications you build with LangChain will contain multiple steps with multiple invocations of LLM calls. As these applications get more and more complex, it becomes crucial to be able to inspect what exactly is going on inside your chain or agent. The best way to do this is with LangSmith.
After you sign up at the link above, make sure to set your environment variables to start logging traces:
```
export LANGSMITH_TRACING="true"export LANGSMITH_API_KEY="..."
```

Or, if in a notebook, you can set them with:
```
import getpassimport osos.environ["LANGSMITH_TRACING"]="true"os.environ["LANGSMITH_API_KEY"]= getpass.getpass()
```

## Documents and Document Loaders​
LangChain implements a Document abstraction, which is intended to represent a unit of text and associated metadata. It has three attributes:
  * `page_content`: a string representing the content;
  * `metadata`: a dict containing arbitrary metadata;
  * `id`: (optional) a string identifier for the document.


The `metadata` attribute can capture information about the source of the document, its relationship to other documents, and other information. Note that an individual `Document` object often represents a chunk of a larger document.
We can generate sample documents when desired:
```
from langchain_core.documents import Documentdocuments =[  Document(    page_content="Dogs are great companions, known for their loyalty and friendliness.",    metadata={"source":"mammal-pets-doc"},),  Document(    page_content="Cats are independent pets that often enjoy their own space.",    metadata={"source":"mammal-pets-doc"},),]
```

**API Reference:**Document
However, the LangChain ecosystem implements document loaders that integrate with hundreds of common sources. This makes it easy to incorporate data from these sources into your AI application.
### Loading documents​
Let's load a PDF into a sequence of `Document` objects. There is a sample PDF in the LangChain repo here -- a 10-k filing for Nike from 2023. We can consult the LangChain documentation for available PDF document loaders. Let's select PyPDFLoader, which is fairly lightweight.
```
from langchain_community.document_loaders import PyPDFLoaderfile_path ="../example_data/nke-10k-2023.pdf"loader = PyPDFLoader(file_path)docs = loader.load()print(len(docs))
```

**API Reference:**PyPDFLoader
```
107
```

tip
See this guide for more detail on PDF document loaders.
`PyPDFLoader` loads one `Document` object per PDF page. For each, we can easily access:
  * The string content of the page;
  * Metadata containing the file name and page number.


```
print(f"{docs[0].page_content[:200]}\n")print(docs[0].metadata)
```

```
Table of ContentsUNITED STATESSECURITIES AND EXCHANGE COMMISSIONWashington, D.C. 20549FORM 10-K(Mark One)☑ ANNUAL REPORT PURSUANT TO SECTION 13 OR 15(D) OF THE SECURITIES EXCHANGE ACT OF 1934FO{'source': '../example_data/nke-10k-2023.pdf', 'page': 0}
```

### Splitting​
For both information retrieval and downstream question-answering purposes, a page may be too coarse a representation. Our goal in the end will be to retrieve `Document` objects that answer an input query, and further splitting our PDF will help ensure that the meanings of relevant portions of the document are not "washed out" by surrounding text.
We can use text splitters for this purpose. Here we will use a simple text splitter that partitions based on characters. We will split our documents into chunks of 1000 characters with 200 characters of overlap between chunks. The overlap helps mitigate the possibility of separating a statement from important context related to it. We use the RecursiveCharacterTextSplitter, which will recursively split the document using common separators like new lines until each chunk is the appropriate size. This is the recommended text splitter for generic text use cases.
We set `add_start_index=True` so that the character index where each split Document starts within the initial Document is preserved as metadata attribute “start_index”.
See this guide for more detail about working with PDFs, including how to extract text from specific sections and images.
```
from langchain_text_splitters import RecursiveCharacterTextSplittertext_splitter = RecursiveCharacterTextSplitter(  chunk_size=1000, chunk_overlap=200, add_start_index=True)all_splits = text_splitter.split_documents(docs)len(all_splits)
```

**API Reference:**RecursiveCharacterTextSplitter
```
514
```

## Embeddings​
Vector search is a common way to store and search over unstructured data (such as unstructured text). The idea is to store numeric vectors that are associated with the text. Given a query, we can embed it as a vector of the same dimension and use vector similarity metrics (such as cosine similarity) to identify related text.
LangChain supports embeddings from dozens of providers. These models specify how text should be converted into a numeric vector. Let's select a model:
Select embeddings model:
OpenAI▾
* OpenAI
* Azure
* Google
* AWS
* HuggingFace
* Ollama
* Cohere
* MistralAI
* Nomic
* NVIDIA
* Voyage AI
* IBM watsonx
* Fake
```
pip install -qU langchain-openai
```

```
import getpassimport osifnot os.environ.get("OPENAI_API_KEY"): os.environ["OPENAI_API_KEY"]= getpass.getpass("Enter API key for OpenAI: ")from langchain_openai import OpenAIEmbeddingsembeddings = OpenAIEmbeddings(model="text-embedding-3-large")
```

```
vector_1 = embeddings.embed_query(all_splits[0].page_content)vector_2 = embeddings.embed_query(all_splits[1].page_content)assertlen(vector_1)==len(vector_2)print(f"Generated vectors of length {len(vector_1)}\n")print(vector_1[:10])
```

```
Generated vectors of length 1536[-0.008586574345827103, -0.03341241180896759, -0.008936782367527485, -0.0036674530711025, 0.010564599186182022, 0.009598285891115665, -0.028587326407432556, -0.015824200585484505, 0.0030416189692914486, -0.012899317778646946]
```

Armed with a model for generating text embeddings, we can next store them in a special data structure that supports efficient similarity search.
## Vector stores​
LangChain VectorStore objects contain methods for adding text and `Document` objects to the store, and querying them using various similarity metrics. They are often initialized with embedding models, which determine how text data is translated to numeric vectors.
LangChain includes a suite of integrations with different vector store technologies. Some vector stores are hosted by a provider (e.g., various cloud providers) and require specific credentials to use; some (such as Postgres) run in separate infrastructure that can be run locally or via a third-party; others can run in-memory for lightweight workloads. Let's select a vector store:
Select vector store:
In-memory▾
* In-memory
* AstraDB
* Chroma
* FAISS
* Milvus
* MongoDB
* PGVector
* Pinecone
* Qdrant
```
pip install -qU langchain-core
```

```
from langchain_core.vectorstores import InMemoryVectorStorevector_store = InMemoryVectorStore(embeddings)
```

Having instantiated our vector store, we can now index the documents.
```
ids = vector_store.add_documents(documents=all_splits)
```

Note that most vector store implementations will allow you to connect to an existing vector store-- e.g., by providing a client, index name, or other information. See the documentation for a specific integration for more detail.
Once we've instantiated a `VectorStore` that contains documents, we can query it. VectorStore includes methods for querying:
  * Synchronously and asynchronously;
  * By string query and by vector;
  * With and without returning similarity scores;
  * By similarity and maximum marginal relevance (to balance similarity with query to diversity in retrieved results).


The methods will generally include a list of Document objects in their outputs.
### Usage​
Embeddings typically represent text as a "dense" vector such that texts with similar meanings are geometrically close. This lets us retrieve relevant information just by passing in a question, without knowledge of any specific key-terms used in the document.
Return documents based on similarity to a string query:
```
results = vector_store.similarity_search("How many distribution centers does Nike have in the US?")print(results[0])
```

```
page_content='direct to consumer operations sell products through the following number of retail stores in the United States:U.S. RETAIL STORES NUMBERNIKE Brand factory stores 213 NIKE Brand in-line stores (including employee-only stores) 74 Converse stores (including factory stores) 82 TOTAL 369 In the United States, NIKE has eight significant distribution centers. Refer to Item 2. Properties for further information.2023 FORM 10-K 2' metadata={'page': 4, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 3125}
```

Async query:
```
results =await vector_store.asimilarity_search("When was Nike incorporated?")print(results[0])
```

```
page_content='Table of ContentsPART IITEM 1. BUSINESSGENERALNIKE, Inc. was incorporated in 1967 under the laws of the State of Oregon. As used in this Annual Report on Form 10-K (this "Annual Report"), the terms "we," "us," "our,""NIKE" and the "Company" refer to NIKE, Inc. and its predecessors, subsidiaries and affiliates, collectively, unless the context indicates otherwise.Our principal business activity is the design, development and worldwide marketing and selling of athletic footwear, apparel, equipment, accessories and services. NIKE isthe largest seller of athletic footwear and apparel in the world. We sell our products through NIKE Direct operations, which are comprised of both NIKE-owned retail storesand sales through our digital platforms (also referred to as "NIKE Brand Digital"), to retail accounts and to a mix of independent distributors, licensees and sales' metadata={'page': 3, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 0}
```

Return scores:
```
# Note that providers implement different scores; the score here# is a distance metric that varies inversely with similarity.results = vector_store.similarity_search_with_score("What was Nike's revenue in 2023?")doc, score = results[0]print(f"Score: {score}\n")print(doc)
```

```
Score: 0.23699893057346344page_content='Table of ContentsFISCAL 2023 NIKE BRAND REVENUE HIGHLIGHTSThe following tables present NIKE Brand revenues disaggregated by reportable operating segment, distribution channel and major product line:FISCAL 2023 COMPARED TO FISCAL 2022•NIKE, Inc. Revenues were $51.2 billion in fiscal 2023, which increased 10% and 16% compared to fiscal 2022 on a reported and currency-neutral basis, respectively.The increase was due to higher revenues in North America, Europe, Middle East & Africa ("EMEA"), APLA and Greater China, which contributed approximately 7, 6,2 and 1 percentage points to NIKE, Inc. Revenues, respectively.•NIKE Brand revenues, which represented over 90% of NIKE, Inc. Revenues, increased 10% and 16% on a reported and currency-neutral basis, respectively. Thisincrease was primarily due to higher revenues in Men's, the Jordan Brand, Women's and Kids' which grew 17%, 35%,11% and 10%, respectively, on a wholesaleequivalent basis.' metadata={'page': 35, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 0}
```

Return documents based on similarity to an embedded query:
```
embedding = embeddings.embed_query("How were Nike's margins impacted in 2023?")results = vector_store.similarity_search_by_vector(embedding)print(results[0])
```

```
page_content='Table of ContentsGROSS MARGINFISCAL 2023 COMPARED TO FISCAL 2022For fiscal 2023, our consolidated gross profit increased 4% to $22,292 million compared to $21,479 million for fiscal 2022. Gross margin decreased 250 basis points to43.5% for fiscal 2023 compared to 46.0% for fiscal 2022 due to the following:*Wholesale equivalentThe decrease in gross margin for fiscal 2023 was primarily due to:•Higher NIKE Brand product costs, on a wholesale equivalent basis, primarily due to higher input costs and elevated inbound freight and logistics costs as well asproduct mix;•Lower margin in our NIKE Direct business, driven by higher promotional activity to liquidate inventory in the current period compared to lower promotional activity inthe prior period resulting from lower available inventory supply;•Unfavorable changes in net foreign currency exchange rates, including hedges; and•Lower off-price margin, on a wholesale equivalent basis.This was partially offset by:' metadata={'page': 36, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 0}
```

Learn more:
  * API reference
  * How-to guide
  * Integration-specific docs


## Retrievers​
LangChain `VectorStore` objects do not subclass Runnable. LangChain Retrievers are Runnables, so they implement a standard set of methods (e.g., synchronous and asynchronous `invoke` and `batch` operations). Although we can construct retrievers from vector stores, retrievers can interface with non-vector store sources of data, as well (such as external APIs).
We can create a simple version of this ourselves, without subclassing `Retriever`. If we choose what method we wish to use to retrieve documents, we can create a runnable easily. Below we will build one around the `similarity_search` method:
```
from typing import Listfrom langchain_core.documents import Documentfrom langchain_core.runnables import chain@chaindefretriever(query:str)-> List[Document]:return vector_store.similarity_search(query, k=1)retriever.batch(["How many distribution centers does Nike have in the US?","When was Nike incorporated?",],)
```

**API Reference:**Document | chain
```
[[Document(metadata={'page': 4, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 3125}, page_content='direct to consumer operations sell products through the following number of retail stores in the United States:\nU.S. RETAIL STORES NUMBER\nNIKE Brand factory stores 213 \nNIKE Brand in-line stores (including employee-only stores) 74 \nConverse stores (including factory stores) 82 \nTOTAL 369 \nIn the United States, NIKE has eight significant distribution centers. Refer to Item 2. Properties for further information.\n2023 FORM 10-K 2')], [Document(metadata={'page': 3, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 0}, page_content='Table of Contents\nPART I\nITEM 1. BUSINESS\nGENERAL\nNIKE, Inc. was incorporated in 1967 under the laws of the State of Oregon. As used in this Annual Report on Form 10-K (this "Annual Report"), the terms "we," "us," "our,"\n"NIKE" and the "Company" refer to NIKE, Inc. and its predecessors, subsidiaries and affiliates, collectively, unless the context indicates otherwise.\nOur principal business activity is the design, development and worldwide marketing and selling of athletic footwear, apparel, equipment, accessories and services. NIKE is\nthe largest seller of athletic footwear and apparel in the world. We sell our products through NIKE Direct operations, which are comprised of both NIKE-owned retail stores\nand sales through our digital platforms (also referred to as "NIKE Brand Digital"), to retail accounts and to a mix of independent distributors, licensees and sales')]]
```

Vectorstores implement an `as_retriever` method that will generate a Retriever, specifically a VectorStoreRetriever. These retrievers include specific `search_type` and `search_kwargs` attributes that identify what methods of the underlying vector store to call, and how to parameterize them. For instance, we can replicate the above with the following:
```
retriever = vector_store.as_retriever(  search_type="similarity",  search_kwargs={"k":1},)retriever.batch(["How many distribution centers does Nike have in the US?","When was Nike incorporated?",],)
```

```
[[Document(metadata={'page': 4, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 3125}, page_content='direct to consumer operations sell products through the following number of retail stores in the United States:\nU.S. RETAIL STORES NUMBER\nNIKE Brand factory stores 213 \nNIKE Brand in-line stores (including employee-only stores) 74 \nConverse stores (including factory stores) 82 \nTOTAL 369 \nIn the United States, NIKE has eight significant distribution centers. Refer to Item 2. Properties for further information.\n2023 FORM 10-K 2')], [Document(metadata={'page': 3, 'source': '../example_data/nke-10k-2023.pdf', 'start_index': 0}, page_content='Table of Contents\nPART I\nITEM 1. BUSINESS\nGENERAL\nNIKE, Inc. was incorporated in 1967 under the laws of the State of Oregon. As used in this Annual Report on Form 10-K (this "Annual Report"), the terms "we," "us," "our,"\n"NIKE" and the "Company" refer to NIKE, Inc. and its predecessors, subsidiaries and affiliates, collectively, unless the context indicates otherwise.\nOur principal business activity is the design, development and worldwide marketing and selling of athletic footwear, apparel, equipment, accessories and services. NIKE is\nthe largest seller of athletic footwear and apparel in the world. We sell our products through NIKE Direct operations, which are comprised of both NIKE-owned retail stores\nand sales through our digital platforms (also referred to as "NIKE Brand Digital"), to retail accounts and to a mix of independent distributors, licensees and sales')]]
```

`VectorStoreRetriever` supports search types of `"similarity"` (default), `"mmr"` (maximum marginal relevance, described above), and `"similarity_score_threshold"`. We can use the latter to threshold documents output by the retriever by similarity score.
Retrievers can easily be incorporated into more complex applications, such as retrieval-augmented generation (RAG) applications that combine a given question with retrieved context into a prompt for a LLM. To learn more about building such an application, check out the RAG tutorial tutorial.
### Learn more:​
Retrieval strategies can be rich and complex. For example:
  * We can infer hard rules and filters from a query (e.g., "using documents published after 2020");
  * We can return documents that are linked to the retrieved context in some way (e.g., via some document taxonomy);
  * We can generate multiple embeddings for each unit of context;
  * We can ensemble results from multiple retrievers;
  * We can assign weights to documents, e.g., to weigh recent documents higher.


The retrievers section of the how-to guides covers these and other built-in retrieval strategies.
It is also straightforward to extend the BaseRetriever class in order to implement custom retrievers. See our how-to guide here.
## Next steps​
You've now seen how to build a semantic search engine over a PDF document.
For more on document loaders:
  * Conceptual guide
  * How-to guides
  * Available integrations


For more on embeddings:
  * Conceptual guide
  * How-to guides
  * Available integrations


For more on vector stores:
  * Conceptual guide
  * How-to guides
  * Available integrations


For more on RAG, see:
  * Build a Retrieval Augmented Generation (RAG) App
  * Related how-to guides


#### Was this page helpful?
  * Concepts
  * Setup
    * Jupyter Notebook
    * Installation
    * LangSmith
  * Documents and Document Loaders
    * Loading documents
    * Splitting
  * Embeddings
  * Vector stores
    * Usage
  * Retrievers
    * Learn more:
  * Next steps


