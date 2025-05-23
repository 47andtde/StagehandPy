<div id="toc" align="center">
  <ul style="list-style: none">
    <a href="https://stagehand.dev">
      <picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://stagehand.dev/logo-dark.svg" />
        <img alt="Stagehand" src="https://stagehand.dev/logo-light.svg" />
      </picture>
    </a>
  </ul>
</div>

<p align="center">
  An AI web browsing framework focused on simplicity and extensibility.<br>
</p>

<p align="center">
  <a href="https://pypi.org/project/stagehand-py">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://img.shields.io/pypi/v/stagehand-py.svg?style=for-the-badge" />
      <img alt="PyPI version" src="https://img.shields.io/pypi/v/stagehand-py.svg?style=for-the-badge" />
    </picture>
  </a>
  <a href="https://github.com/browserbase/stagehand/tree/main?tab=MIT-1-ov-file#MIT-1-ov-file">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://stagehand.dev/api/assets/license?mode=dark" />
      <img alt="MIT License" src="https://stagehand.dev/api/assets/license?mode=light" />
    </picture>
  </a>
  <a href="https://join.slack.com/t/stagehand-dev/shared_invite/zt-2tdncfgkk-fF8y5U0uJzR2y2_M9c9OJA">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://stagehand.dev/api/assets/slack?mode=dark" />
      <img alt="Slack Community" src="https://stagehand.dev/api/assets/slack?mode=light" />
    </picture>
  </a>
</p>

<div class="note" style="background-color: #808096; border-left: 5px solid #ffeb3b; padding: 15px; margin: 10px 0; color: white;">
  <strong>NOTE:</strong> This is a Python SDK for Stagehand. The original implementation is in TypeScript and is available <a href="https://github.com/browserbase/stagehand" style="color: blue;">here</a>.
</div>

---

Stagehand is the easiest way to build browser automations with AI-powered interactions.

- **act** — Instruct the AI to perform actions (e.g. click a button or scroll).
```python
await stagehand.page.act("click on the 'Quickstart' button")
```
- **extract** — Extract and validate data from a page using a JSON schema (generated either manually or via a Pydantic model).
```python
await stagehand.page.extract("the summary of the first paragraph")
```
- **observe** — Get natural language interpretations to, for example, identify selectors or elements from the DOM.
```python
await stagehand.page.observe("find the search bar")
```
- **agent** — Execute autonomous multi-step tasks with provider-specific agents (OpenAI, Anthropic, etc.).
```python
await stagehand.agent.execute("book a reservation for 2 people for a trip to the Maldives")
```

## Installation

Install the Python package via pip:

```bash
pip install stagehand-py
```
## Requirements

- Python 3.7+
- httpx (for async client)
- requests (for sync client)
- asyncio (for async client)
- pydantic
- python-dotenv (optional, for .env support)
- playwright
- rich (for `examples/` terminal support)

You can simply run:

```bash
pip install -r requirements.txt
```


## Environment Variables

Before running your script, set the following environment variables:

```bash
export BROWSERBASE_API_KEY="your-api-key"
export BROWSERBASE_PROJECT_ID="your-project-id"
export MODEL_API_KEY="your-openai-api-key"  # or your preferred model's API key
export STAGEHAND_SERVER_URL="url-of-stagehand-server"
```

You can also make a copy of `.env.example` and add these to your `.env` file. 

## Quickstart

Stagehand supports both synchronous and asynchronous usage. Here are examples for both approaches:

### Sync Client

```python
import os
from stagehand.sync import Stagehand, StagehandConfig
from dotenv import load_dotenv

load_dotenv()

def main():
    # Configure Stagehand
    config = StagehandConfig(
        env="BROWSERBASE",
        api_key=os.getenv("BROWSERBASE_API_KEY"),
        project_id=os.getenv("BROWSERBASE_PROJECT_ID"),
        model_name="gpt-4o",
        model_client_options={"apiKey": os.getenv("MODEL_API_KEY")}
    )

    # Initialize Stagehand
    stagehand = Stagehand(config=config, server_url=os.getenv("STAGEHAND_SERVER_URL"))
    stagehand.init()
    print(f"Session created: {stagehand.session_id}")

    # Navigate to a page
    stagehand.page.goto("https://google.com/")

    # Use Stagehand AI primitives
    stagehand.page.act("search for openai")

    # Combine with Playwright
    stagehand.page.keyboard.press("Enter")

    # Observe elements on the page
    observed = stagehand.page.observe("find the news button")
    if observed:
        stagehand.page.act(observed[0])  # Act on the first observed element

    # Extract data from the page
    data = stagehand.page.extract("extract the first result from the search")
    print(f"Extracted data: {data}")

    # Close the session
    stagehand.close()

if __name__ == "__main__":
    main()
```

### Async Client

```python
import os
import asyncio
from stagehand import Stagehand, StagehandConfig
from dotenv import load_dotenv

load_dotenv()

async def main():
    # Configure Stagehand
    config = StagehandConfig(
        env="BROWSERBASE",
        api_key=os.getenv("BROWSERBASE_API_KEY"),
        project_id=os.getenv("BROWSERBASE_PROJECT_ID"),
        model_name="gpt-4o",
        model_client_options={"apiKey": os.getenv("MODEL_API_KEY")}
    )

    # Initialize Stagehand
    stagehand = Stagehand(config=config, server_url=os.getenv("STAGEHAND_SERVER_URL"))
    await stagehand.init()
    print(f"Session created: {stagehand.session_id}")
    
    # Get page reference
    page = stagehand.page

    # Navigate to a page
    await page.goto("https://google.com/")

    # Use Stagehand AI primitives
    await page.act("search for openai")

    # Combine with Playwright
    await page.keyboard.press("Enter")

    # Observe elements on the page
    observed = await page.observe("find the news button")
    if observed:
        await page.act(observed[0])  # Act on the first observed element

    # Extract data from the page
    data = await page.extract("extract the first result from the search")
    print(f"Extracted data: {data}")

    # Close the session
    await stagehand.close()

if __name__ == "__main__":
    asyncio.run(main())
```

## Agent Example

```python
import os
from stagehand.sync import Stagehand, StagehandConfig
from stagehand.schemas import AgentConfig, AgentExecuteOptions, AgentProvider
from dotenv import load_dotenv

load_dotenv()

def main():
    # Configure Stagehand
    config = StagehandConfig(
        env="BROWSERBASE",
        api_key=os.getenv("BROWSERBASE_API_KEY"),
        project_id=os.getenv("BROWSERBASE_PROJECT_ID"),
        model_name="gpt-4o",
        model_client_options={"apiKey": os.getenv("MODEL_API_KEY")}
    )

    # Initialize Stagehand
    stagehand = Stagehand(config=config, server_url=os.getenv("STAGEHAND_SERVER_URL"))
    stagehand.init()
    print(f"Session created: {stagehand.session_id}")
    
    # Navigate to Google
    stagehand.page.goto("https://google.com/")
    
    # Configure the agent
    agent_config = AgentConfig(
        provider=AgentProvider.OPENAI,
        model="computer-use-preview",
        instructions="You are a helpful web navigation assistant. You are currently on google.com."
        options={"apiKey": os.getenv("MODEL_API_KEY")}
    )
    
    # Define execution options
    execute_options = AgentExecuteOptions(
        instruction="Search for 'latest AI news' and extract the titles of the first 3 results",
        max_steps=10,
        auto_screenshot=True
    )
    
    # Execute the agent task
    agent_result = stagehand.agent.execute(agent_config, execute_options)
    
    print(f"Agent execution result: {agent_result}")
    
    # Close the session
    stagehand.close()

if __name__ == "__main__":
    main()
```

## Pydantic Schemas

- **ActOptions**  

  The `ActOptions` model takes an `action` field that tells the AI what to do on the page, plus optional fields such as `useVision` and `variables`:
  ```python
  from stagehand.schemas import ActOptions
  
  # Example:
  await page.act(ActOptions(action="click on the 'Quickstart' button"))
  ```

- **ObserveOptions**  

  The `ObserveOptions` model lets you find elements on the page using natural language. The `onlyVisible` option helps limit the results:
  ```python
  from stagehand.schemas import ObserveOptions
  
  # Example:
  await page.observe(ObserveOptions(instruction="find the button labeled 'News'", onlyVisible=True))
  ```

- **ExtractOptions**  

  The `ExtractOptions` model extracts structured data from the page. Pass your instructions and a schema defining your expected data format. **Note:** If you are using a Pydantic model for the schema, call its `.model_json_schema()` method to ensure JSON serializability.
  ```python
  from stagehand.schemas import ExtractOptions
  from pydantic import BaseModel
  
  class DescriptionSchema(BaseModel):
      description: str
  
  # Example:
  data = await page.extract(
      ExtractOptions(
          instruction="extract the description of the page",
          schemaDefinition=DescriptionSchema.model_json_schema()
      )
  )
  description = data.get("description") if isinstance(data, dict) else data.description
  ```

## Why?
**Stagehand adds determinism to otherwise unpredictable agents.**

While there's no limit to what you could instruct Stagehand to do, our primitives allow you to control how much you want to leave to an AI. It works best when your code is a sequence of atomic actions. Instead of writing a single script for a single website, Stagehand allows you to write durable, self-healing, and repeatable web automation workflows that actually work.

> [!NOTE] 
> `Stagehand` is currently available as an early release, and we're actively seeking feedback from the community. Please join our [Slack community](https://join.slack.com/t/stagehand-dev/shared_invite/zt-2tdncfgkk-fF8y5U0uJzR2y2_M9c9OJA) to stay updated on the latest developments and provide feedback.


## Configuration

Stagehand can be configured via environment variables or through a `StagehandConfig` object. Available configuration options include:

- `stagehand_server_url`: URL of the Stagehand API server.
- `browserbase_api_key`: Your Browserbase API key (`BROWSERBASE_API_KEY`).
- `browserbase_project_id`: Your Browserbase project ID (`BROWSERBASE_PROJECT_ID`).
- `model_api_key`: Your model API key (e.g. OpenAI, Anthropic, etc.) (`MODEL_API_KEY`).
- `verbose`: Verbosity level (default: 1).
  - Level 0: Error logs
  - Level 1: Basic info logs (minimal, maps to INFO level)
  - Level 2: Medium logs including warnings (maps to WARNING level)
  - Level 3: Detailed debug information (maps to DEBUG level)
- `model_name`: Optional model name for the AI (e.g. "gpt-4o").
- `dom_settle_timeout_ms`: Additional time (in ms) to have the DOM settle.
- `debug_dom`: Enable debug mode for DOM operations.
- `stream_response`: Whether to stream responses from the server (default: True).
- `timeout_settings`: Custom timeout settings for HTTP requests.

Example using a unified configuration:

```python
from stagehand.config import StagehandConfig
import os

config = StagehandConfig(
    env="BROWSERBASE" if os.getenv("BROWSERBASE_API_KEY") and os.getenv("BROWSERBASE_PROJECT_ID") else "LOCAL",
    api_key=os.getenv("BROWSERBASE_API_KEY"),
    project_id=os.getenv("BROWSERBASE_PROJECT_ID"),
    debug_dom=True,
    headless=False,
    dom_settle_timeout_ms=3000,
    model_name="gpt-4o-mini",
    model_client_options={"apiKey": os.getenv("MODEL_API_KEY")},
    verbose=3  # Set verbosity level: 1=minimal, 2=medium, 3=detailed logs
)
```

## License

MIT License (c) 2025 Browserbase, Inc.
