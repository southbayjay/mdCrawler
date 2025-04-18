Browser Use home page![light logo](https://mintlify.s3.us-west-1.amazonaws.com/browseruse-0aece648/logo/light.svg)![dark logo](https://mintlify.s3.us-west-1.amazonaws.com/browseruse-0aece648/logo/dark.svg)
Search or ask...
Ctrl K
Search...
Navigation
Customize
Browser Settings
DocumentationCloud API
##### Get Started
  * Introduction
  * Quickstart


##### Customize
  * Supported Models
  * Agent Settings
  * Browser Settings
  * Connect to your Browser
  * Output Format
  * System Prompt
  * Sensitive Data
  * Custom Functions


##### Development
  * Local Setup
  * Telemetry
  * Observability
  * Roadmap


Browser Use allows you to customize how the browser behaves through two main configuration classes: `BrowserConfig` and `BrowserContextConfig`. These settings control everything from headless mode to proxy settings and page load behavior.
We are currently working on improving how browser contexts are managed. The system will soon transition to a “1 agent, 1 browser, 1 context” model for better stability and developer experience.
# 
​
Browser Configuration
The `BrowserConfig` class controls the core browser behavior and connection settings.
Copy
```
from browser_use import BrowserConfig
# Basic configuration
config = BrowserConfig(
  headless=False,
  disable_security=True
)

```

## 
​
Core Settings
  * **headless** (default: `False`) Runs the browser without a visible UI. Note that some websites may detect headless mode.
  * **disable_security** (default: `True`) Disables browser security features. While this can fix certain functionality issues (like cross-site iFrames), it should be used cautiously, especially when visiting untrusted websites.


### 
​
Additional Settings
  * **extra_chromium_args** (default: `[]`) Additional arguments passed to the browser at launch. See the full list of available arguments.
  * **proxy** (default: `None`) Standard Playwright proxy settings for using external proxy services.
  * **new_context_config** (default: `BrowserContextConfig()`) Default settings for new browser contexts. See Context Configuration below.


For web scraping tasks on sites that restrict automated access, we recommend using external browser or proxy providers for better reliability.
## 
​
Alternative Initialization
These settings allow you to connect to external browser providers or use a local Chrome instance.
### 
​
External Browser Provider (wss)
Connect to cloud-based browser services for enhanced reliability and proxy capabilities.
Copy
```
config = BrowserConfig(
  wss_url="wss://your-browser-provider.com/ws"
)

```

  * **wss_url** (default: `None`) WebSocket URL for connecting to external browser providers (e.g., anchorbrowser.com, steel.dev, browserbase.com, browserless.io).


This overrides local browser settings and uses the provider’s configuration. Refer to their documentation for settings.
### 
​
External Browser Provider (cdp)
Connect to cloud or local Chrome instances using Chrome DevTools Protocol (CDP) for use with tools like `headless-shell` or `browserless`.
Copy
```
config = BrowserConfig(
  cdp_url="http://localhost:9222"
)

```

  * **cdp_url** (default: `None`) URL for connecting to a Chrome instance via CDP. Commonly used for debugging or connecting to locally running Chrome instances.


### 
​
Local Chrome Instance (binary)
Connect to your existing Chrome installation to access saved states and cookies.
Copy
```
config = BrowserConfig(
  chrome_instance_path="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
)

```

  * **chrome_instance_path** (default: `None`) Path to connect to an existing Chrome installation. Particularly useful for workflows requiring existing login states or browser preferences.


This will overwrite other browser settings.
# 
​
Context Configuration
The `BrowserContextConfig` class controls settings for individual browser contexts.
Copy
```
from browser_use.browser.context import BrowserContextConfig
config = BrowserContextConfig(
  cookies_file="path/to/cookies.json",
  wait_for_network_idle_page_load_time=3.0,
  browser_window_size={'width': 1280, 'height': 1100},
  locale='en-US',
  user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36',
  highlight_elements=True,
  viewport_expansion=500,
  allowed_domains=['google.com', 'wikipedia.org'],
)
browser = Browser()
context = BrowserContext(browser=browser, config=config)

async def run_search():
	agent = Agent(
		browser_context=context,
		task='Your task',
		llm=llm)

```

## 
​
Configuration Options
### 
​
Page Load Settings
  * **minimum_wait_page_load_time** (default: `0.5`) Minimum time to wait before capturing page state for LLM input.
  * **wait_for_network_idle_page_load_time** (default: `1.0`) Time to wait for network activity to cease. Increase to 3-5s for slower websites. This tracks essential content loading, not dynamic elements like videos.
  * **maximum_wait_page_load_time** (default: `5.0`) Maximum time to wait for page load before proceeding.


### 
​
Display Settings
  * **browser_window_size** (default: `{'width': 1280, 'height': 1100}`) Browser window dimensions. The default size is optimized for general use cases and interaction with common UI elements like cookie banners.
  * **locale** (default: `None`) Specify user locale, for example en-GB, de-DE, etc. Locale will affect navigator.language value, Accept-Language request header value as well as number and date formatting rules. If not provided, defaults to the system default locale.
  * **highlight_elements** (default: `True`) Highlight interactive elements on the screen with colorfull bounding boxes.
  * **viewport_expansion** (default: `500`) Viewport expansion in pixels. With this you can controll how much of the page is included in the context of the LLM. If set to -1, all elements from the entire page will be included (this leads to high token usage). If set to 0, only the elements which are visible in the viewport will be included. Default is 500 pixels, that means that we inlcude a little bit more than the visible viewport inside the context.


### 
​
Restrict URLs
  * **allowed_domains** (default: `None`) List of allowed domains that the agent can access. If None, all domains are allowed. Example: [‘google.com’, ‘wikipedia.org’] - Here the agent will only be able to access google and wikipedia.


### 
​
Debug and Recording
  * **save_recording_path** (default: `None`) Directory path for saving video recordings.
  * **trace_path** (default: `None`) Directory path for saving trace files. Files are automatically named as `{trace_path}/{context_id}.zip`.


Was this page helpful?
YesNo
Agent SettingsConnect to your Browser
On this page
  * Browser Configuration
  * Core Settings
  * Additional Settings
  * Alternative Initialization
  * External Browser Provider (wss)
  * External Browser Provider (cdp)
  * Local Chrome Instance (binary)
  * Context Configuration
  * Configuration Options
  * Page Load Settings
  * Display Settings
  * Restrict URLs
  * Debug and Recording


