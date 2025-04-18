Skip to content 
Version Notice
This documentation is ahead of the last release by 8 commits. You may see documentation for features not yet supported in the latest release v0.0.24 2025-02-12. 
# Models
PydanticAI is Model-agnostic and has built in support for the following model providers:
  * OpenAI
  * Anthropic
  * Gemini via two different APIs: Generative Language API and VertexAI API
  * Ollama
  * Deepseek
  * Groq
  * Mistral
  * Cohere


See OpenAI-compatible models for more examples on how to use models such as OpenRouter, and Grok (xAI) that support the OpenAI SDK.
You can also add support for other models.
PydanticAI also comes with `TestModel` and `FunctionModel` for testing and development.
To use each model provider, you need to configure your local environment and make sure you have the right packages installed.
## OpenAI
### Install
To use OpenAI models, you need to either install `pydantic-ai`, or install `pydantic-ai-slim` with the `openai` optional group:
pipuv
```
pipinstall'pydantic-ai-slim[openai]'

```

```
uvadd'pydantic-ai-slim[openai]'

```

### Configuration
To use `OpenAIModel` through their main API, go to platform.openai.com and follow your nose until you find the place to generate an API key.
### Environment variable
Once you have the API key, you can set it as an environment variable:
```
exportOPENAI_API_KEY='your-api-key'

```

You can then use `OpenAIModel` by name:
openai_model_by_name.py```
from pydantic_ai import Agent
agent = Agent('openai:gpt-4o')
...

```

Or initialise the model directly with just the model name:
openai_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
model = OpenAIModel('gpt-4o')
agent = Agent(model)
...

```

### `api_key` argument
If you don't want to or can't set the environment variable, you can pass it at runtime via the `api_key` argument:
openai_model_api_key.py```
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
model = OpenAIModel('gpt-4o', api_key='your-api-key')
agent = Agent(model)
...

```

### Custom OpenAI Client
`OpenAIModel` also accepts a custom `AsyncOpenAI` client via the `openai_client` parameter, so you can customise the `organization`, `project`, `base_url` etc. as defined in the OpenAI API docs.
You could also use the `AsyncAzureOpenAI` client to use the Azure OpenAI API.
openai_azure.py```
from openai import AsyncAzureOpenAI
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
client = AsyncAzureOpenAI(
  azure_endpoint='...',
  api_version='2024-07-01-preview',
  api_key='your-api-key',
)
model = OpenAIModel('gpt-4o', openai_client=client)
agent = Agent(model)
...

```

## Anthropic
### Install
To use `AnthropicModel` models, you need to either install `pydantic-ai`, or install `pydantic-ai-slim` with the `anthropic` optional group:
pipuv
```
pipinstall'pydantic-ai-slim[anthropic]'

```

```
uvadd'pydantic-ai-slim[anthropic]'

```

### Configuration
To use Anthropic through their API, go to console.anthropic.com/settings/keys to generate an API key.
`AnthropicModelName` contains a list of available Anthropic models.
### Environment variable
Once you have the API key, you can set it as an environment variable:
```
exportANTHROPIC_API_KEY='your-api-key'

```

You can then use `AnthropicModel` by name:
anthropic_model_by_name.py```
from pydantic_ai import Agent
agent = Agent('anthropic:claude-3-5-sonnet-latest')
...

```

Or initialise the model directly with just the model name:
anthropic_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.anthropic import AnthropicModel
model = AnthropicModel('claude-3-5-sonnet-latest')
agent = Agent(model)
...

```

### `api_key` argument
If you don't want to or can't set the environment variable, you can pass it at runtime via the `api_key` argument:
anthropic_model_api_key.py```
from pydantic_ai import Agent
from pydantic_ai.models.anthropic import AnthropicModel
model = AnthropicModel('claude-3-5-sonnet-latest', api_key='your-api-key')
agent = Agent(model)
...

```

## Gemini
For prototyping only
Google themselves refer to this API as the "hobby" API, I've received 503 responses from it a number of times. The API is easy to use and useful for prototyping and simple demos, but I would not rely on it in production.
If you want to run Gemini models in production, you should use the VertexAI API described below.
### Install
To use `GeminiModel` models, you just need to install `pydantic-ai` or `pydantic-ai-slim`, no extra dependencies are required.
### Configuration
`GeminiModel` let's you use the Google's Gemini models through their Generative Language API, `generativelanguage.googleapis.com`.
`GeminiModelName` contains a list of available Gemini models that can be used through this interface.
To use `GeminiModel`, go to aistudio.google.com and follow your nose until you find the place to generate an API key.
### Environment variable
Once you have the API key, you can set it as an environment variable:
```
exportGEMINI_API_KEY=your-api-key

```

You can then use `GeminiModel` by name:
gemini_model_by_name.py```
from pydantic_ai import Agent
agent = Agent('google-gla:gemini-1.5-flash')
...

```

Note
The `google-gla` provider prefix represents the Google **G** enerative **L** anguage **A** PI for `GeminiModel`s. `google-vertex` is used with Vertex AI for `VertexAIModel`s.
Or initialise the model directly with just the model name:
gemini_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.gemini import GeminiModel
model = GeminiModel('gemini-1.5-flash')
agent = Agent(model)
...

```

### `api_key` argument
If you don't want to or can't set the environment variable, you can pass it at runtime via the `api_key` argument:
gemini_model_api_key.py```
from pydantic_ai import Agent
from pydantic_ai.models.gemini import GeminiModel
model = GeminiModel('gemini-1.5-flash', api_key='your-api-key')
agent = Agent(model)
...

```

## Gemini via VertexAI
To run Google's Gemini models in production, you should use `VertexAIModel` which uses the `*-aiplatform.googleapis.com` API.
`GeminiModelName` contains a list of available Gemini models that can be used through this interface.
### Install
To use `VertexAIModel`, you need to either install `pydantic-ai`, or install `pydantic-ai-slim` with the `vertexai` optional group:
pipuv
```
pipinstall'pydantic-ai-slim[vertexai]'

```

```
uvadd'pydantic-ai-slim[vertexai]'

```

### Configuration
This interface has a number of advantages over `generativelanguage.googleapis.com` documented above:
  1. The VertexAI API is more reliably and marginally lower latency in our experience.
  2. You can purchase provisioned throughput with VertexAI to guarantee capacity.
  3. If you're running PydanticAI inside GCP, you don't need to set up authentication, it should "just work".
  4. You can decide which region to use, which might be important from a regulatory perspective, and might improve latency.


The big disadvantage is that for local development you may need to create and configure a "service account", which I've found extremely painful to get right in the past.
Whichever way you authenticate, you'll need to have VertexAI enabled in your GCP account.
### Application default credentials
Luckily if you're running PydanticAI inside GCP, or you have the `gcloud` CLI installed and configured, you should be able to use `VertexAIModel` without any additional setup.
To use `VertexAIModel`, with application default credentials configured (e.g. with `gcloud`), you can simply use:
vertexai_application_default_credentials.py```
from pydantic_ai import Agent
from pydantic_ai.models.vertexai import VertexAIModel
model = VertexAIModel('gemini-1.5-flash')
agent = Agent(model)
...

```

Internally this uses `google.auth.default()` from the `google-auth` package to obtain credentials.
Won't fail until `agent.run()`
Because `google.auth.default()` requires network requests and can be slow, it's not run until you call `agent.run()`. Meaning any configuration or permissions error will only be raised when you try to use the model. To initialize the model for this check to be run, call `await model.ainit()`.
You may also need to pass the `project_id` argument to `VertexAIModel` if application default credentials don't set a project, if you pass `project_id` and it conflicts with the project set by application default credentials, an error is raised.
### Service account
If instead of application default credentials, you want to authenticate with a service account, you'll need to create a service account, add it to your GCP project (note: AFAIK this step is necessary even if you created the service account within the project), give that service account the "Vertex AI Service Agent" role, and download the service account JSON file.
Once you have the JSON file, you can use it thus:
vertexai_service_account.py```
from pydantic_ai import Agent
from pydantic_ai.models.vertexai import VertexAIModel
model = VertexAIModel(
  'gemini-1.5-flash',
  service_account_file='path/to/service-account.json',
)
agent = Agent(model)
...

```

### Customising region
Whichever way you authenticate, you can specify which region requests will be sent to via the `region` argument.
Using a region close to your application can improve latency and might be important from a regulatory perspective.
vertexai_region.py```
from pydantic_ai import Agent
from pydantic_ai.models.vertexai import VertexAIModel
model = VertexAIModel('gemini-1.5-flash', region='asia-east1')
agent = Agent(model)
...

```

`VertexAiRegion` contains a list of available regions.
## Groq
### Install
To use `GroqModel`, you need to either install `pydantic-ai`, or install `pydantic-ai-slim` with the `groq` optional group:
pipuv
```
pipinstall'pydantic-ai-slim[groq]'

```

```
uvadd'pydantic-ai-slim[groq]'

```

### Configuration
To use Groq through their API, go to console.groq.com/keys and follow your nose until you find the place to generate an API key.
`GroqModelName` contains a list of available Groq models.
### Environment variable
Once you have the API key, you can set it as an environment variable:
```
exportGROQ_API_KEY='your-api-key'

```

You can then use `GroqModel` by name:
groq_model_by_name.py```
from pydantic_ai import Agent
agent = Agent('groq:llama-3.3-70b-versatile')
...

```

Or initialise the model directly with just the model name:
groq_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.groq import GroqModel
model = GroqModel('llama-3.3-70b-versatile')
agent = Agent(model)
...

```

### `api_key` argument
If you don't want to or can't set the environment variable, you can pass it at runtime via the `api_key` argument:
groq_model_api_key.py```
from pydantic_ai import Agent
from pydantic_ai.models.groq import GroqModel
model = GroqModel('llama-3.3-70b-versatile', api_key='your-api-key')
agent = Agent(model)
...

```

## Mistral
### Install
To use `MistralModel`, you need to either install `pydantic-ai`, or install `pydantic-ai-slim` with the `mistral` optional group:
pipuv
```
pipinstall'pydantic-ai-slim[mistral]'

```

```
uvadd'pydantic-ai-slim[mistral]'

```

### Configuration
To use Mistral through their API, go to console.mistral.ai/api-keys/ and follow your nose until you find the place to generate an API key.
`MistralModelName` contains a list of the most popular Mistral models.
### Environment variable
Once you have the API key, you can set it as an environment variable:
```
exportMISTRAL_API_KEY='your-api-key'

```

You can then use `MistralModel` by name:
mistral_model_by_name.py```
from pydantic_ai import Agent
agent = Agent('mistral:mistral-large-latest')
...

```

Or initialise the model directly with just the model name:
mistral_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.mistral import MistralModel
model = MistralModel('mistral-small-latest')
agent = Agent(model)
...

```

### `api_key` argument
If you don't want to or can't set the environment variable, you can pass it at runtime via the `api_key` argument:
mistral_model_api_key.py```
from pydantic_ai import Agent
from pydantic_ai.models.mistral import MistralModel
model = MistralModel('mistral-small-latest', api_key='your-api-key')
agent = Agent(model)
...

```

## Cohere
### Install
To use `CohereModel`, you need to either install `pydantic-ai`, or install `pydantic-ai-slim` with the `cohere` optional group:
pipuv
```
pipinstall'pydantic-ai-slim[cohere]'

```

```
uvadd'pydantic-ai-slim[cohere]'

```

### Configuration
To use Cohere through their API, go to dashboard.cohere.com/api-keys and follow your nose until you find the place to generate an API key.
`CohereModelName` contains a list of the most popular Cohere models.
### Environment variable
Once you have the API key, you can set it as an environment variable:
```
exportCO_API_KEY='your-api-key'

```

You can then use `CohereModel` by name:
cohere_model_by_name.py```
from pydantic_ai import Agent
agent = Agent('cohere:command')
...

```

Or initialise the model directly with just the model name:
cohere_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.cohere import CohereModel
model = CohereModel('command', api_key='your-api-key')
agent = Agent(model)
...

```

### `api_key` argument
If you don't want to or can't set the environment variable, you can pass it at runtime via the `api_key` argument:
cohere_model_api_key.py```
from pydantic_ai import Agent
from pydantic_ai.models.cohere import CohereModel
model = CohereModel('command', api_key='your-api-key')
agent = Agent(model)
...

```

## OpenAI-compatible Models
Many of the models are compatible with OpenAI API, and thus can be used with `OpenAIModel` in PydanticAI. Before getting started, check the OpenAI section for installation and configuration instructions.
To use another OpenAI-compatible API, you can make use of the `base_url` and `api_key` arguments:
openai_model_base_url.py```
from pydantic_ai.models.openai import OpenAIModel
model = OpenAIModel(
  'model_name',
  base_url='https://<openai-compatible-api-endpoint>.com',
  api_key='your-api-key',
)
...

```

### Ollama
To use Ollama, you must first download the Ollama client, and then download a model using the Ollama model library.
You must also ensure the Ollama server is running when trying to make requests to it. For more information, please see the Ollama documentation.
#### Example local usage
With `ollama` installed, you can run the server with the model you want to use:
terminal-run-ollama```
ollamarunllama3.2

```

(this will pull the `llama3.2` model if you don't already have it downloaded) 
Then run your code, here's a minimal example:
ollama_example.py```
from pydantic import BaseModel
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel

class CityLocation(BaseModel):
  city: str
  country: str

ollama_model = OpenAIModel(model_name='llama3.2', base_url='http://localhost:11434/v1')
agent = Agent(ollama_model, result_type=CityLocation)
result = agent.run_sync('Where were the olympics held in 2012?')
print(result.data)
#> city='London' country='United Kingdom'
print(result.usage())
"""
Usage(requests=1, request_tokens=57, response_tokens=8, total_tokens=65, details=None)
"""

```

#### Example using a remote server
ollama_example_with_remote_server.py```
from pydantic import BaseModel
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
ollama_model = OpenAIModel(
  model_name='qwen2.5-coder:7b', 
  base_url='http://192.168.1.74:11434/v1', 
)

class CityLocation(BaseModel):
  city: str
  country: str

agent = Agent(model=ollama_model, result_type=CityLocation)
result = agent.run_sync('Where were the olympics held in 2012?')
print(result.data)
#> city='London' country='United Kingdom'
print(result.usage())
"""
Usage(requests=1, request_tokens=57, response_tokens=8, total_tokens=65, details=None)
"""

```

### OpenRouter
To use OpenRouter, first create an API key at openrouter.ai/keys.
Once you have the API key, you can pass it to `OpenAIModel` as the `api_key` argument:
openrouter_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
model = OpenAIModel(
  'anthropic/claude-3.5-sonnet',
  base_url='https://openrouter.ai/api/v1',
  api_key='your-openrouter-api-key',
)
agent = Agent(model)
...

```

### Grok (xAI)
Go to xAI API Console and create an API key. Once you have the API key, follow the xAI API Documentation, and set the `base_url` and `api_key` arguments appropriately:
grok_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
model = OpenAIModel(
  'grok-2-1212',
  base_url='https://api.x.ai/v1',
  api_key='your-xai-api-key',
)
agent = Agent(model)
...

```

### DeepSeek
Go to DeepSeek API Platform and create an API key. Once you have the API key, follow the DeepSeek API Documentation, and set the `base_url` and `api_key` arguments appropriately:
deepseek_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
model = OpenAIModel(
  'deepseek-chat',
  base_url='https://api.deepseek.com',
  api_key='your-deepseek-api-key',
)
agent = Agent(model)
...

```

### Perplexity
Follow the Perplexity getting started guide to create an API key. Then, you can query the Perplexity API with the following:
perplexity_model_init.py```
from pydantic_ai import Agent
from pydantic_ai.models.openai import OpenAIModel
model = OpenAIModel(
  'sonar-pro',
  base_url='https://api.perplexity.ai',
  api_key='your-perplexity-api-key',
)
agent = Agent(model)
...

```

## Implementing Custom Models
To implement support for models not already supported, you will need to subclass the `Model` abstract base class.
For streaming, you'll also need to implement the following abstract base class:
  * `StreamedResponse`


The best place to start is to review the source code for existing implementations, e.g. `OpenAIModel`.
For details on when we'll accept contributions adding new models to PydanticAI, see the contributing guidelines.
