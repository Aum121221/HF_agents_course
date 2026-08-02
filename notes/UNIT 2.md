# SMOLAGENTS 
`smolagents` is a simple yet powerful framework for building AI agents. It provides LLMs with the _agency_ to interact with the real world, such as searching or generating images.
`smolagents` is an open-source Python library designed to make it extremely easy to build and run agents using just a few lines of code.

# AGENT PARAMETERS

| Parameter             | Type        | Purpose                                                            |
| --------------------- | ----------- | ------------------------------------------------------------------ |
| `tools`               | list        | Defines which tools the agent can call (search, image gen, ektc.). |
| `prompt_templates`    | object      | Customizes how prompts are structured for the agent.               |
| `instructions`        | str         | High-level guidance or rules for the agent.                        |
| `max_steps`           | int         | Limits how many reasoning steps the agent can take.                |
| `add_base_tools`      | bool        | Whether to include default tools automatically.                    |
| `verbosity_level`     | LogLevel    | Controls logging detail (INFO, DEBUG, etc.).                       |
| `managed_agents`      | list        | Allows orchestration of multiple agents.                           |
| `step_callbacks`      | callable(s) | Hooks to run after each reasoning step.                            |
| `planning_interval`   | int         | Frequency of planning updates.                                     |
| `name`                | str         | Human-readable name for the agent.                                 |
| `description`         | str         | Short description of the agent’s role.                             |
| `provide_run_summary` | bool        | Whether to output a summary after execution.                       |
| `final_answer_checks` | callable(s) | Validations before returning the final answer.                     |
| `return_full_result`  | bool        | Whether to return intermediate reasoning traces.                   |
| `logger`              | AgentLogger | Custom logging integration.                                        |


---

# AGENT METHODS
### 🧩 `extract_action` ()

When a language model (LLM) generates text, it often mixes two parts:

- **Thought** → the reasoning (“I need to calculate”).
- **Action** → the actual command (“calculator(2+2)”).

To separate these cleanly, we use a **marker word** (called the `split_token`).  
That marker tells the agent: _“Everything after this word is the action.”_

---

⚙️ Parameters

- **`model_output (str)`** → The raw text the LLM produced.
- **`split_token (str)`** → The keyword that marks where the action starts.
    - Example: `"Action:"` or `"Tool:"` depending on your system prompt.
    - It must match exactly what you used in your prompt examples.

---

 #### 🔧 Example

```python
from smolagents import MultiStepAgent

model_output = "Thought: I need to calculate. Action: calculator(2+2)"

# Use "Action:" as the split_token
action = MultiStepAgent.extract_action(model_output, split_token="Action:")

print(action)
# Output: calculator(2+2)
```

Here:

- The LLM wrote both a thought and an action.
- `extract_action` looked for `"Action:"` and returned only the part after it.

---

 🔄 Workflow Table

|Step|Input|split_token|Output|
|---|---|---|---|
|1|`"Thought: I need to calculate. Action: calculator(2+2)"`|`"Action:"`|`"calculator(2+2)"`|
|2|`"Thought: Search info. Tool: search_web('AI agents')"`|`"Tool:"`|`"search_web('AI agents')"`|

---

👉 In short: **`split_token` is the keyword that tells the agent where the action begins in the LLM’s text.** Without it, the agent wouldn’t know how to separate reasoning from the actual command.

---

### 🧩  `from_dict` ()

- It’s a **shortcut** for creating an agent from a saved configuration.
- Instead of typing all the parameters again (`tools`, `name`, `max_steps`, etc.), you put them in a **dictionary** (like a settings file).
- `from_dict` reads that dictionary and builds the agent for you.
- If you pass extra keyword arguments (`**kwargs`), they **override** what’s in the dictionary.

---

 #### ⚙️ Example

```python
agent_config = {
    "name": "MathAgent",
    "description": "Solves math problems",
    "max_steps": 5,
    "tools": ["calculator"]
}

# Create agent from the dictionary
agent = MultiStepAgent.from_dict(agent_config)

# Override max_steps while loading
agent = MultiStepAgent.from_dict(agent_config, max_steps=10)
```

Here:

- The first call builds the agent exactly as described in `agent_config`.
- The second call changes `max_steps` to 10, even though the dictionary said 5.

---

 🔄 Why It’s Useful

- **Reusability** → Save agent configs and reload them later.
- **Portability** → Share configs with teammates.
- **Flexibility** → Adjust settings quickly without editing the original dict.

---

👉 In short: **`from_dict` is the “load from settings” method.** You give it a dictionary of agent details, and it gives you back a working agent.



---

### 🧩  from_folder ()

- **Purpose** → Loads an agent that was previously saved in a local folder.
- Instead of rebuilding the agent from scratch or from a dictionary, you can just point to the folder where its configuration and files are stored.
- It returns a working `MultiStepAgent` instance.

---

⚙️ Parameters

- **`folder (str | Path)`** → The path to the folder where the agent is saved.
- **`**kwargs`** → Extra arguments you can pass in. These will override or add to the agent’s original settings when it’s loaded.

---

#### 🔧 Example

```python
from smolagents import MultiStepAgent

# Load agent from a local folder
agent = MultiStepAgent.from_folder("saved_agents/math_agent")

# Override something while loading
agent = MultiStepAgent.from_folder("saved_agents/math_agent", max_steps=15)
```

Here:

- The first call rebuilds the agent exactly as it was saved in `"saved_agents/math_agent"`.
- The second call loads it but changes `max_steps` to 15.

---

🔄 Why It’s Useful

- **Persistence** → You can save agents once and reload them later without redefining everything.
- **Portability** → Share the folder with teammates, and they can load the same agent.
- **Flexibility** → Adjust parameters on the fly with `**kwargs`.

---

👉 In short: **`from_folder` is the “load from local files” method.** You give it the folder path where the agent was saved, and it reconstructs the agent for you.



---

### 🧩  from_hub ()

- **Purpose** → Loads an agent that has been uploaded to the Hugging Face Hub.
- Instead of creating the agent locally or from a dictionary, you can fetch it directly from a Hub repository.
- This makes it easy to share and reuse agents across different environments.

---

 ⚙️ Parameters

- **`repo_id (str)`** → The name of the repository on Hugging Face Hub where the agent is stored.
    - Example: `"username/my-agent-repo"`.
- **`token (str | None)`** → Your Hugging Face authentication token.
    - If not provided, it uses the token from `huggingface-cli login`.
- **`trust_remote_code (bool)`** → Defaults to `False`.
    - If set to `True`, you acknowledge the risk and allow execution of remote code from the Hub repo.
    - If left `False`, loading will fail if the repo requires custom code.
- **`**kwargs`** → Extra arguments.
    - Hub-related ones (like `cache_dir`, `revision`, `subfolder`) are used for downloading.
    - Others are passed to the agent’s initialization.

---

#### 🔧 Example

```python
from smolagents import MultiStepAgent

# Load agent from Hugging Face Hub
agent = MultiStepAgent.from_hub(
    repo_id="Aum121221/math-agent",
    token="your_hf_token_here",
    trust_remote_code=True,
    max_steps=10
)
```

Here:

- The agent is fetched from the Hub repo `"Aum121221/math-agent"`.
- Authentication is handled with your Hugging Face token.
- `trust_remote_code=True` allows custom code from the repo.
- `max_steps=10` overrides the default setting.

---

🔄 Why It’s Useful

- **Collaboration** → Share agents publicly or privately on Hugging Face Hub.
- **Reusability** → Load agents anywhere without needing local files.
- **Flexibility** → Override parameters while loading.

---

👉 In short: **`from_hub` is the “load from Hugging Face Hub” method.** You give it the repo name, token, and trust flag, and it reconstructs the agent from the Hub.


---

### 🧩 `initialize_system_prompt()`

- **Purpose** → This is a placeholder method.
- It’s meant to be **implemented in child classes** (like `CodeAgent` or `ToolCallingAgent`).
- Each agent type can define its own **system prompt** (the initial instructions that guide the LLM).
- By default, the base class doesn’t implement it.

---

### 🧩 `interrupt()`

- **Purpose** → Stops the agent’s execution mid‑run.
- Useful if you want to **cancel a task** before it finishes.
- Example: If the agent is looping through steps and you want to halt it, call `interrupt()`.

---

### 🧩 provide_final_answer()
   **`provide_final_answer(task: str, images: list[PIL.Image.Image] | None) → str`**
   
- **Purpose** → Produces the **final answer** to the task after the agent has finished its reasoning and tool calls.
- **Parameters**:
    - `task (str)` → The task description.
    - `images (optional)` → A list of image objects if the task involves visual input.
- **Returns** → A string containing the final answer.
- It uses the **logs of the agent’s interactions** (thoughts, actions, observations) to summarize and provide the final result.

---

 🔄 Workflow Table

| Method                       | What it Does                              | When to Use                       |
| ---------------------------- | ----------------------------------------- | --------------------------------- |
| `initialize_system_prompt()` | Defines the agent’s starting instructions | In child classes (custom prompts) |
| `interrupt()`                | Stops agent execution                     | If you need to cancel a run       |
| `provide_final_answer()`     | Returns the final answer based on logs    | At the end of a task              |
|                              |                                           |                                   |

---

👉 In short:

- **`initialize_system_prompt`** → sets up the agent’s “brain” (system instructions).
- **`interrupt`** → emergency stop.
- **`provide_final_answer`** → wraps up the task with a clean result.



---

### 🧩  push_to_hub ()

- **Purpose** → Uploads your agent to the Hugging Face Hub so it can be shared, reused, or version‑controlled.
- Think of it as the opposite of `from_hub`: instead of loading, you’re publishing.

---

 ⚙️ Parameters

- **`repo_id (str)`** → The name of the repository you want to push to.
    - Example: `"username/my-agent-repo"` or `"orgname/agent-repo"`.
- **`commit_message (str)`** → Message for the commit. Defaults to `"Upload agent"`.
- **`private (bool | None)`** → Whether to make the repo private.
    - If `None`, it follows the organization’s default (usually public).
- **`token (bool | str | None)`** → Hugging Face authentication token.
    - If not set, it uses the token from `huggingface-cli login`.
- **`create_pr (bool)`** → If `True`, creates a Pull Request instead of committing directly. Defaults to `False`.

---

 #### 🔧 Example

```python
from smolagents import MultiStepAgent

# Assume you already have an agent object
agent.push_to_hub(
    repo_id="Aum121221/math-agent",
    commit_message="Initial upload of MathAgent",
    private=True,
    token="your_hf_token_here",
    create_pr=False
)
```

This will:

- Upload the agent to the repo `"Aum121221/math-agent"`.
- Use the commit message `"Initial upload of MathAgent"`.
- Make the repo private.
- Authenticate with your Hugging Face token.
- Commit directly (no PR).

---

🔄 Why It’s Useful

- **Collaboration** → Share agents with others via the Hub.
- **Version Control** → Track changes with commit messages.
- **Distribution** → Make agents public or private depending on your needs.
- **Integration** → Others can load your agent with `from_hub`.

---

👉 In short: **`push_to_hub` is the “publish agent online” method.** It takes your local agent and uploads it to Hugging Face Hub, making it accessible to others.



---

### 🧩  replay ()

- **Purpose** → Prints a **pretty replay** of the agent’s steps during execution.
- It shows the sequence of **Thought → Action → Observation → Answer** so you can see how the agent reasoned.
- This is mainly for **debugging and analysis**.

---

⚙️ Parameters

- **`detailed (bool)`** → Optional.
    - If `False` (default), it shows a clean replay of steps.
    - If `True`, it also displays the **memory state at each step**.
    - ⚠️ Warning: This makes the logs much longer (exponentially), so use only when debugging.

---

#### 🔧 Example

```python
# Simple replay
agent.replay()

# Detailed replay with memory at each step
agent.replay(detailed=True)
```

---

🔄 Workflow Table

|Step|What Happens|
|---|---|
|1|Agent runs a task (ReAct cycle).|
|2|Logs are stored (thoughts, actions, observations).|
|3|`replay()` prints those logs in a readable format.|
|4|If `detailed=True`, memory snapshots are also shown.|

---



---

### 🧩  run ()

- **Purpose** → Executes the agent on a given task using the ReAct cycle (Thought → Action → Observation → Answer).
- It’s the main entry point to actually _use_ the agent once it’s set up.

#### 🔧 Example

```python
from smolagents import CodeAgent

agent = CodeAgent(tools=[])

# Simple run
result = agent.run("What is the result of 2 power 3.7384?")
print(result)

# Streaming run (step by step)
for step in agent.run("Solve 2+2", stream=True):
    print(step)
```

---


---

👉 In short: **`run` is the method that actually makes the agent solve a task.** You can choose between a simple one‑shot answer or a detailed step‑by‑step replay depending on your parameters.


---

### 🧩 save()

- **Purpose** → Saves your agent into a local folder so you can reload it later or publish it to the Hugging Face Hub.
- It doesn’t just copy your agent’s code — it also auto‑generates supporting files so the agent is portable and reproducible.

---

 ⚙️ Parameters

- **`output_dir (str | Path)`** → The folder where you want to save your agent.
- **`relative_path (str | None)`** → Optional. Lets you specify a relative path inside the output folder.

---

 📂 What Gets Saved

When you call `save`, the following files/folders are created inside `output_dir`:

- **`tools/{tool_name}.py`** → Python files for each tool the agent uses.
- **`managed_agents/`** → Logic for any managed agents.
- **`agent.json`** → Dictionary representation of the agent (its config).
- **`prompt.yaml`** → Prompt templates used by the agent.
- **`app.py`** → A simple UI for the agent (used if you export it to a Hugging Face Space).
- **`requirements.txt`** → Auto‑detected Python dependencies.

---

#### 🔧 Example

```python
from smolagents import MultiStepAgent

# Assume you already have an agent object
agent.save(output_dir="saved_agents/math_agent")
```

This will create a folder `saved_agents/math_agent` with all the files listed above.

---

 🔄 Why It’s Useful

- **Persistence** → Save your agent once and reload it later with `from_folder`.
- **Portability** → Share the folder with teammates.
- **Publishing** → Use `push_to_hub` to upload the saved agent to Hugging Face Hub.
- **Reproducibility** → All code, prompts, and requirements are stored together.

---

👉 In short: **`save` is the “export agent locally” method.** It packages your agent into a folder with code, config, prompts, and dependencies so you can reload or share it easily.



---

### 🧩`step()`
**`step(memory_step: ActionStep)`**

- **Purpose** → Executes **one cycle** of the ReAct framework:
    - The agent **thinks** (reasoning),
    - **acts** (calls a tool),
    - **observes** (gets the result).
- **Return value** →
    - `None` if the step is not final (agent continues).
    - A **final answer string** if the task is completed.

---

### 🧩 `to_dict()` 

- **Purpose** → Converts the agent into a **dictionary representation**.
- This is the same format used by `from_dict`.
- Useful for saving, exporting, or serializing the agent’s configuration.

---





---

### 🧩 `stream_to_gradio()`

- **Purpose** → Runs an agent on a given task and streams its messages into a **Gradio ChatInterface**.
- **Parameters**:
    - `agent` → The agent you want to run.
    - `task (str)` → The task to perform.
    - `task_images (list | None)` → Optional images for multimodal tasks.
    - `reset_agent_memory (bool)` → If `True`, clears memory before starting.
    - `additional_args (dict | None)` → Extra inputs (e.g., dataframes).

#### Example

```python
from smolagents import CodeAgent, stream_to_gradio

agent = CodeAgent(tools=[], model=my_model)

# Stream agent responses into Gradio
stream_to_gradio(agent, task="Solve 2+2", reset_agent_memory=True)
```

---

## smolagents — Model methods

---

### `🧩generate()`

The core method of every model. It takes a list of messages and returns the model's response as a `ChatMessage` object. All agents call this method internally whenever they need a response from the model.

**`messages`** — A list of message dictionaries, each with a `role` (`"user"` or `"system"`) and a `content` field. Can also accept a list of `ChatMessage` objects directly.

**`stop_sequences`** — An optional list of strings that immediately halt generation when encountered in the output.

**`response_format`** — An optional dictionary to control the structure of the model's output, for example enforcing JSON responses.

**`tools_to_call_from`** — An optional list of `Tool` objects made available to the model during generation. The model can choose to call any of them in its response.

**`**kwargs`** — Any additional arguments forwarded to the underlying model API for that specific call.

---

### `🧩parse_tool_calls()`

Some model APIs return tool calls as raw text rather than structured objects. This method parses the raw response and extracts tool call information into a usable format.

---

### `🧩to_dict()`

Serializes the model configuration into a JSON-compatible dictionary. Useful for saving, logging, or reconstructing a model from its configuration.

---

```python
from smolagents import HfApiModel, CodeAgent
from smolagents.tools import DuckDuckGoSearchTool

model = HfApiModel(model_id="meta-llama/Llama-3.3-70B-Instruct")

# Calling generate() directly
response = model.generate(
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user",   "content": "What is the capital of France?"}
    ],
    stop_sequences=["END"],
    response_format={"type": "json_object"},
    tools_to_call_from=[DuckDuckGoSearchTool()],
)
print(response)           # ChatMessage object

# Parsing tool calls from a raw response
model.parse_tool_calls(response)

# Serializing the model config
print(model.to_dict())    # JSON-compatible dictionary
```

---

### 🧩 `visualize()`

- **Purpose** → Creates a **rich tree visualization** of the agent’s structure.
- Shows how tools, prompts, and managed agents are connected.
- Helpful for debugging or explaining the agent’s design.

---

### 🧩 `write_memory_to_messages()`

**`write_memory_to_messages(summary_mode: bool = False)`**
- **Purpose** → Reads the agent’s past memory (LLM outputs, actions, observations, errors) and turns them into a **series of messages**.
- These messages can be fed back into the LLM to continue reasoning.
- Adds helpful keywords like `PLAN`, `error`, etc. to guide the model.
- **Parameter**:
    - `summary_mode=True` → compresses memory into summaries.
    - `False` → includes full details.

---

 🔄 Workflow Table

| Method                       | What It Does                                  | When to Use                       |
| ---------------------------- | --------------------------------------------- | --------------------------------- |
| `step()`                     | Runs one Thought → Action → Observation cycle | During agent execution            |
| `to_dict()`                  | Converts agent to dictionary                  | For saving/exporting              |
| `visualize()`                | Shows tree view of agent structure            | For debugging/explaining          |
| `write_memory_to_messages()` | Turns memory into LLM input messages          | For continuing tasks or debugging |

---

👉 In short:

- **`step`** → one execution cycle.
- **`to_dict`** → save/export agent config.
- **`visualize`** → see agent’s structure.
- **`write_memory_to_messages`** → replay memory as messages for the LLM.
### EXAMPLE
```
from smolagents import CodeAgent

agent = CodeAgent(tools=[calculator_tool])

# Step execution
step_result = agent.step(memory_step="Solve 2+2")
print(step_result)  # None if ongoing, or "4" if final

# Convert to dictionary
print(agent.to_dict())

# Visualize structure
agent.visualize()

# Replay memory as messages
messages = agent.write_memory_to_messages()
print(messages)

```



---

# SMOLAGENTS IMPORTABLES

## AGENT CLASS

HF agents inherit from [MultiStepAgent](https://huggingface.co/docs/smolagents/main/en/reference/agents#smolagents.MultiStepAgent), which means they can act in multiple steps, each step consisting of one thought, then one tool call and execution. Read more in [this conceptual guide](https://huggingface.co/docs/smolagents/main/en/conceptual_guides/react).

We provide two types of agents, based on the main `Agent` class.

- [CodeAgent](https://huggingface.co/docs/smolagents/main/en/reference/agents#smolagents.CodeAgent) writes its tool calls in Python code (this is the default).
- [ToolCallingAgent](https://huggingface.co/docs/smolagents/main/en/reference/agents#smolagents.ToolCallingAgent) writes its tool calls in JSON.

Both require arguments `model` and list of tools `tools` at initialization.


#### 🧩 `CodeAgent` Overview

- **Purpose** → An agent that formulates tool calls in **code format** (Python snippets), then parses and executes them.
- **Core Parameters**:
    - `tools` → List of tools the agent can use (e.g., calculator, search).
    - `model` → The model that generates actions.
    - `executor_type` → Where code runs (`local`, `docker`, `wasm`, etc.).
    - `prompt_templates` → Custom prompts guiding reasoning.
    - `stream_outputs` → Whether to stream results step by step.

---

🔧 Key Methods

- **`cleanup()`** → Frees resources (e.g., remote executor).
- **`from_dict(agent_dict, **kwargs)`** → Rebuilds a `CodeAgent` from a dictionary config.
- **`run(task, ...)`** → Executes a full task (Thought → Action → Observation → Answer).
- **`step(memory_step)`** → Runs a single cycle of reasoning.
- **`to_dict()`** → Converts agent to a dictionary (for saving/exporting).
- **`visualize()`** → Shows a tree view of the agent’s structure.
- **`write_memory_to_messages()`** → Turns past logs into structured messages for the LLM.
- **`save(output_dir)`** → Exports the agent locally with tools, prompts, and requirements.
- **`push_to_hub(repo_id, ...)`** → Publishes the agent to Hugging Face Hub.
- **`from_hub(repo_id, ...)`** → Loads an agent from Hugging Face Hub.
- **`from_folder(path)`** → Loads an agent from a local folder.

---




👉 In short: **`CodeAgent` is the “code‑driven” agent class**. You can build it, run tasks, save locally, reload, publish to Hub, and share — all while keeping its reasoning cycle transparent and reproducible.



---

#### 🧩 `ToolCallingAgent` Overview

- **Purpose** → An agent that uses **JSON‑like tool calls** instead of code snippets.
- It leverages the LLM’s built‑in **tool calling capabilities** (`model.get_tool_call`).
- This makes tool execution more structured and less error‑prone compared to `CodeAgent`.

---

⚙️ Parameters

- **`tools (list[Tool])`** → Tools the agent can use.
- **`model (Model)`** → The LLM generating tool calls.
- **`prompt_templates`** → Optional custom prompts.
- **`planning_interval`** → How often the agent does a planning step.
- **`stream_outputs (bool)`** → Whether to stream results step by step.
- **`max_tool_threads (int)`** → Maximum threads for parallel tool calls (controls concurrency).
- **`**kwargs`** → Extra arguments.

---

🔧 Key Methods

- **`execute_tool_call(tool_name, arguments)`**
    
    - Runs a specific tool with given arguments.
    - Arguments can be a dict or string.
    - If arguments reference state variables, they’re replaced with actual values.
- **`process_tool_calls(chat_message, memory_step)`**
    
    - Reads tool calls from the model’s output (`chat_message`).
    - Executes them and updates memory (`ActionStep`).
    - Yields either a `ToolCall` (the request) or `ToolOutput` (the result).

---

#### 🔄  Example

```python
from smolagents import ToolCallingAgent

# Create agent with tools
agent = ToolCallingAgent(tools=[search_tool, calculator_tool], model=my_model)

# Run a task
result = agent.run("Search the web for AI agents and add 2+2")
print(result)
```

**Behind the scenes:**

1. The LLM outputs JSON‑like tool calls:
    
    ```json
    { "tool": "search_tool", "arguments": {"query": "AI agents"} }
    { "tool": "calculator_tool", "arguments": {"expression": "2+2"} }
    ```
    
2. `process_tool_calls` parses these calls.
3. `execute_tool_call` runs the tools.
4. Memory logs are updated with Thought → Action → Observation.
5. Final answer is returned.

---

🔄 Comparison: `CodeAgent` vs `ToolCallingAgent`

|Feature|CodeAgent|ToolCallingAgent|
|---|---|---|
|Tool calls|Python code snippets|JSON‑like structured calls|
|Execution|Parsed + executed code|Direct tool execution via `get_tool_call`|
|Flexibility|More free‑form, but error‑prone|Safer, structured, less parsing errors|
|Best for|Complex code generation tasks|Direct tool orchestration|

---

👉 In short:

- **`ToolCallingAgent`** is the structured, JSON‑driven agent.
- **`CodeAgent`** is the code‑driven agent.
- Both follow the ReAct cycle, but differ in how they represent and execute tool calls.

Here’s how the **Gradio integration** works in Smolagents:

---

## MEMORY CLASS

### 🧩 `AgentMemory` Overview

- **Purpose** → Stores everything the agent does across multiple steps.
- It keeps track of:
    - The **system prompt** (the agent’s role/instructions).
    - All **steps** taken (tasks, actions, planning).
- This allows the agent to **replay**, **reset**, or **summarize** its reasoning history.

---

⚙️ Attributes

- **`system_prompt (SystemPromptStep)`** → The initial instruction guiding the agent.
- **`steps (list[TaskStep | ActionStep | PlanningStep])`** → A chronological log of what the agent did.

---

🔧 Key Methods

- **`get_full_steps()`** → Returns a detailed log of all steps, including model input messages.
- **`get_succinct_steps()`** → Returns a shorter version (no raw model inputs).
- **`replay(logger, detailed=False)`** → Prints a “movie” of the agent’s steps.
    - If `detailed=True`, also shows memory at each step (useful for debugging).
- **`reset()`** → Clears all steps but keeps the system prompt.
- **`return_full_code()`** → Concatenates all code actions into one script (handy for debugging or exporting).

---

#### 🔄  Example

```python
from smolagents import AgentMemory

# Initialize memory with a system prompt
memory = AgentMemory(system_prompt="You are a math solver.")

# Agent runs steps (internally logs them)
# ...

# Replay steps
memory.replay(logger=my_logger, detailed=True)

# Get succinct history
print(memory.get_succinct_steps())

# Reset memory
memory.reset()

# Extract all code actions
print(memory.return_full_code())
```

---

📂 Workflow Table

|Method|Purpose|
|---|---|
|`get_full_steps()`|Full detailed log of steps|
|`get_succinct_steps()`|Short summary of steps|
|`replay()`|Pretty replay of reasoning|
|`reset()`|Clear memory, keep system prompt|
|`return_full_code()`|Export all code actions|

---

👉 In short: **AgentMemory is the agent’s “black box recorder.”** It logs every thought, action, and observation so you can replay, debug, reset, or extract code later.

## UTILITY CLASS
### 🧩 `GradioUI`

- **Purpose** → Provides a **web interface** for interacting with a `MultiStepAgent`.
- Uses `gradio.ChatInterface` for a native chatbot experience.
- Supports **file uploads** and **memory reset**.

**Parameters:**

- `agent (MultiStepAgent)` → The agent to interact with.
- `file_upload_folder (str | None)` → Where uploaded files are stored.
- `reset_agent_memory (bool)` → If `True`, clears memory at each interaction.

#### Example

```python
from smolagents import CodeAgent, GradioUI, InferenceClientModel

model = InferenceClientModel(model_id="meta-llama/Meta-Llama-3.1-8B-Instruct")
agent = CodeAgent(tools=[], model=model)

gradio_ui = GradioUI(agent, file_upload_folder="uploads", reset_agent_memory=True)
gradio_ui.launch()
```

---

**🔧 Supporting Methods**


- **`launch(share=True, **kwargs)`** → Starts the Gradio app.
- **`upload_file(file, file_uploads_log, allowed_file_types)`** → Handles file uploads with validation.

---

🔄 Workflow Table

| Stage | Method               | Purpose                            |
| ----- | -------------------- | ---------------------------------- |
| 1     | `stream_to_gradio()` | Stream agent responses into Gradio |
| 2     | `launch()`           | Start the Gradio app               |
| 3     | `upload_file()`      | Handle file uploads securely       |
```
from smolagents import GradioUI

# Example usage inside GradioUI
uploaded_files = []
file = "notes.pdf"

# Validate and log upload
GradioUI.upload_file(
    file=file,
    file_uploads_log=uploaded_files,
    allowed_file_types=[".pdf", ".docx"]
)

print(uploaded_files)  # ["notes.pdf"]

```
---


---




## PROMPT CLASS
### 🧩 `PromptTemplates`

- **Purpose** → Holds all the prompt templates an agent uses.
- **Parameters**:
    - `system_prompt (str)` → The initial system instruction (like the agent’s “role”).
    - `planning (PlanningPromptTemplate)` → Prompts for planning steps.
    - `managed_agent (ManagedAgentPromptTemplate)` → Prompts for managed agents.
    - `final_answer (FinalAnswerPromptTemplate)` → Prompts for wrapping up with the final answer.

---

### 🧩 `PlanningPromptTemplate`

- **Purpose** → Guides the agent when it needs to **plan** its next steps.
- **Parameters**:
    - `plan (str)` → Initial planning prompt.
    - `update_plan_pre_messages (str)` → Prompt before updating the plan.
    - `update_plan_post_messages (str)` → Prompt after updating the plan.

---

### 🧩 `ManagedAgentPromptTemplate`

- **Purpose** → Guides interactions with **managed agents** (sub‑agents).
- **Parameters**:
    - `task (str)` → Prompt for assigning a task.
    - `report (str)` → Prompt for reporting results.

---

### 🧩 `FinalAnswerPromptTemplate`

- **Purpose** → Guides the agent when producing the **final answer**.
- **Parameters**:
    - `pre_messages (str)` → Prompt before final answer.
    - `post_messages (str)` → Prompt after final answer.

---

🔄 Workflow Table

| Stage         | Template                     | Purpose                           |
| ------------- | ---------------------------- | --------------------------------- |
| System        | `system_prompt`              | Defines agent’s role and rules    |
| Planning      | `PlanningPromptTemplate`     | Helps agent plan and update steps |
| Managed Agent | `ManagedAgentPromptTemplate` | Coordinates sub‑agents            |
| Final Answer  | `FinalAnswerPromptTemplate`  | Wraps up with a polished answer   |

---

#### ⚙️ Example



```python
from smolagents import PromptTemplates, PlanningPromptTemplate, ManagedAgentPromptTemplate, FinalAnswerPromptTemplate

prompts = PromptTemplates(
    system_prompt="You are a math problem solver.",
    planning=PlanningPromptTemplate(
        plan="First, outline the steps to solve the problem.",
        update_plan_pre_messages="Check if the plan needs updating.",
        update_plan_post_messages="Confirm updated plan."
    ),
    managed_agent=ManagedAgentPromptTemplate(
        task="Delegate sub-task to managed agent.",
        report="Summarize managed agent’s findings."
    ),
    final_answer=FinalAnswerPromptTemplate(
        pre_messages="Prepare final answer.",
        post_messages="Deliver final answer clearly."
    )
)
```

---

👉 In short: **PromptTemplates are the scaffolding for agent reasoning.** They define how the agent talks to itself (planning), to sub‑agents (managed agent), and to the user (final answer).

Here’s how **memory** works in Smolagents, using the `AgentMemory` class:

---

## `MODEL CLASS`

Models are the **core engines** of AI systems. They allow agents, apps, and workflows to THINK, OBSERVE, and ACT.

Got it! Here's the revised version:

---

### 🧩 — Model class

The `Model` class is the **base class** for all model implementations in smolagents. It defines a standard interface that every model must follow to work with agents. You never instantiate it directly.

- **Inference** = the _act of running the model_ to produce results.
    
- **API** = the _interface_ you use to request that inference from a hosted model.
---

**`model_id`** — A string identifier for the specific model to use, e.g. `"meta-llama/Llama-3.3-70B-Instruct"`.

**`flatten_messages_as_text`** — A boolean that controls how messages are sent to the model. When `True`, complex message objects are converted to plain text before being sent. Defaults to `False`.

**`tool_name_key`** — The key used to extract the tool name from the model's response. Only change this if your model returns tool calls under a different key. Defaults to `"name"`.

**`tool_arguments_key`** — The key used to extract tool arguments from the model's response. Only change this if your model uses a different key. Defaults to `"arguments"`.

**`**kwargs`** — Additional keyword arguments such as `temperature`, `max_tokens`, and `top_p` that are forwarded directly to the underlying model's API call on every completion request.

---

```python
from smolagents import CodeAgent, HfApiModel

model = HfApiModel(
    model_id="meta-llama/Llama-3.3-70B-Instruct",
    flatten_messages_as_text=False,
    tool_name_key="name",
    tool_arguments_key="arguments",
    temperature=0.7,
    max_tokens=1000,
    top_p=0.9,
)

agent = CodeAgent(tools=[], model=model)
agent.run("What is 2 + 2?")
```

---

### 🧩 — `ApiModel` class

---

`ApiModel` is a subclass of `Model` and the base class for all API-based model implementations. It extends the core `Model` interface with functionality specific to external API interactions — such as rate limiting, retries, and client management. You don't use it directly either; concrete subclasses like `HfApiModel` and `OpenAIServerModel` inherit from it.

---

**`model_id`** — A required string identifier for the model to be used with the API, e.g. `"meta-llama/Llama-3.3-70B-Instruct"`.

**`custom_role_conversions`** — An optional dictionary that maps internal role names to API-specific ones. Useful when a third-party API uses different role naming conventions than smolagents expects, e.g. `{"system": "developer"}`.

**`client`** — An optional pre-configured API client instance. If not provided, `create_client()` is called automatically to build a default one.

**`requests_per_minute`** — An optional float that enforces a rate limit on outgoing requests. Smolagents will throttle calls to stay within this limit.

**`retry`** — A boolean that controls whether the model automatically retries on rate limit errors, up to a maximum number of attempts defined by `RETRY_MAX_ATTEMPTS`. Defaults to `True`.

**`**kwargs`** — Additional keyword arguments forwarded to the underlying model completion call, such as `temperature` or `max_tokens`.

---

create_client()`

Called automatically during initialization if no `client` is provided. Creates and configures the API client for the specific service the subclass targets. You would override this when writing a custom API-based model subclass.

---

```python
from smolagents import HfApiModel

# HfApiModel inherits from ApiModel
model = HfApiModel(
    model_id="meta-llama/Llama-3.3-70B-Instruct",
    custom_role_conversions={"system": "developer"},  # remap role names if needed
    client=None,               # auto-creates a default client via create_client()
    requests_per_minute=30.0,  # throttle to 30 requests per minute
    retry=True,                # auto-retry on rate limit errors
    temperature=0.7,
    max_tokens=1000,
)

agent.run("Summarize the latest AI research papers.")
```

---

### 🧩 — `TransformersModel`

---

`TransformersModel` is a subclass of `Model` that runs models **locally on your machine** using the Hugging Face `transformers` library. Instead of calling an external API, it builds a local pipeline for the given `model_id`. Requires `transformers` and `torch` to be installed.

```bash
pip install 'smolagents[transformers]'
```

---

**`model_id`** — The Hugging Face model identifier or a local path to the model. For example `"HuggingFaceTB/SmolLM-135M-Instruct"`.

**`device_map`** — Controls which hardware the model is loaded onto. Common values are `"cpu"`, `"cuda"`, or `"auto"` to let the library decide automatically.

**`torch_dtype`** — The data type used to load model weights, e.g. `"float16"` or `"bfloat16"`. Lower precision reduces memory usage.

**`trust_remote_code`** — Some models on the Hub include custom code that must be executed during loading. Set to `True` only for those models. Defaults to `False`.

**`model_kwargs`** — A dictionary of additional arguments passed directly to `AutoModel.from_pretrained()`, such as `revision` or `config`.

**`max_new_tokens`** — The maximum number of new tokens the model can generate in a single call, not counting the input prompt. Defaults to `4096`.

**`max_tokens`** — An alias for `max_new_tokens`. If both are provided, `max_tokens` takes precedence.

**`apply_chat_template_kwargs`** — A dictionary of extra arguments passed to the tokenizer's `apply_chat_template()` method for fine-grained control over prompt formatting.

**`**kwargs`** — Any additional arguments forwarded to the underlying `model.generate()` call, such as `temperature` or `top_p`.

---

```python
from smolagents import TransformersModel, CodeAgent

model = TransformersModel(
    model_id="Qwen/Qwen3-Next-80B-A3B-Thinking",
    device_map="auto",
    torch_dtype="bfloat16",
    trust_remote_code=False,
    model_kwargs={"revision": "main"},
    max_new_tokens=5000,
    temperature=0.7,
    top_p=0.9,
)

agent = CodeAgent(tools=[], model=model)
agent.run("Explain quantum mechanics in simple terms.")
```

---

### 🧩 — `InferenceClientModel`

---

`InferenceClientModel` wraps `huggingface_hub`'s `InferenceClient` to run models through external **Inference Providers** rather than locally. It supports Cerebras, Cohere, Fal, Fireworks, HF-Inference, Hyperbolic, Nebius, Novita, Replicate, SambaNova, Together, and more.

---

**`model_id`** — The Hugging Face model identifier to use for inference. Can also be a URL pointing to a deployed Inference Endpoint. Defaults to `"Qwen/Qwen3-Next-80B-A3B-Thinking"`.

**`provider`** — The name of the inference provider to route the request through, e.g. `"novita"` or `"hyperbolic"`. Defaults to `"auto"`, which picks the first available provider for the model. Ignored if `base_url` is set.

**`token`** — Your Hugging Face API token for authentication. If not provided, falls back to the `HF_TOKEN` environment variable or the token stored in the HF CLI config. Required for gated models like Llama-3.

**`api_key`** — An alias for `token`, provided to match the `openai.OpenAI` client pattern. Cannot be used together with `token`.

**`timeout`** — Maximum time in seconds to wait for an API response before raising an error. Defaults to `120`.

**`client_kwargs`** — A dictionary of additional arguments passed directly to `huggingface_hub.InferenceClient` during initialization.

**`custom_role_conversions`** — A dictionary to remap message role names for models that do not support certain roles, e.g. `{"system": "user"}`.

**`bill_to`** — An optional organization name to bill API usage to instead of your personal account. The organization must be a member of Enterprise Hub.

**`base_url`** — A custom base URL for self-hosted or dedicated endpoints. Cannot be used together with `model_id`.

**`**kwargs`** — Additional arguments forwarded to the underlying completion call, such as `temperature`, `max_tokens`, or `top_p`.

---

`create_client()`

Called automatically on initialization. Creates and configures the `huggingface_hub.InferenceClient` instance for the selected provider. You do not need to call this manually.

---

```python
from smolagents import InferenceClientModel, CodeAgent

model = InferenceClientModel(
    model_id="Qwen/Qwen3-Next-80B-A3B-Thinking",
    provider="hyperbolic",
    token="your_hf_token_here",   # or set HF_TOKEN env variable
    timeout=120,
    custom_role_conversions={"system": "user"},  # for models that don't support system role
    bill_to="my-org",             # optional: bill to an Enterprise Hub org
    temperature=0.7,
    max_tokens=5000,
    top_p=0.9,
)

agent = CodeAgent(tools=[], model=model)
agent.run("Explain quantum mechanics in simple terms.")
```

---

### 🧩 — `LiteLLMModel`

---

`LiteLLMModel` uses the [LiteLLM](https://www.litellm.ai/) SDK to provide a **single unified interface for 100+ LLMs** across providers like OpenAI, Anthropic, Google, Cohere, Mistral, and more. Instead of managing separate clients for each provider, you simply change the `model_id` string and LiteLLM handles the rest.

---

**`model_id`** — The model identifier in LiteLLM's format, which follows the pattern `"provider/model-name"`, e.g. `"anthropic/claude-3-5-sonnet-latest"` or `"openai/gpt-4o"`.

**`api_base`** — An optional custom base URL for the provider's API. Useful when pointing to a self-hosted or proxy endpoint instead of the provider's default URL.

**`api_key`** — The API key for authenticating with the target provider. If not provided, LiteLLM falls back to the relevant environment variable, e.g. `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`.

**`custom_role_conversions`** — A dictionary to remap message role names for api's that do not support certain roles or have different naming for the same, 
e.g. `{"system" "user" "assistant"}`.

**`flatten_messages_as_text`** — Controls whether structured message objects are flattened into plain text before being sent. Automatically defaults to `True` for `"ollama"`, `"groq"`, and `"cerebras"` models.

**`**kwargs`** — Additional arguments forwarded to every LiteLLM completion call, such as `temperature`, `max_tokens`, `top_p`, or `requests_per_minute`.

---

 `create_client()`

Called automatically on initialization. Sets up the underlying LiteLLM client for the selected provider. You do not need to call this manually.

---

```python
from smolagents import LiteLLMModel, CodeAgent

model = LiteLLMModel(
    model_id="anthropic/claude-3-5-sonnet-latest",
    api_key="your_anthropic_api_key",   # or set ANTHROPIC_API_KEY env variable
    api_base=None,                       # use provider's default URL
    custom_role_conversions={"system": "user"},
    flatten_messages_as_text=False,
    temperature=0.2,
    max_tokens=1000,
    requests_per_minute=60,
)

agent = CodeAgent(tools=[], model=model)
agent.run("Summarize the latest research on quantum computing.")
```

---

### 🧩 — `LiteLLMRouterModel`

---

Think of `LiteLLMRouterModel` as a **traffic controller** sitting in front of multiple LLM providers. Instead of sending every request to one fixed model, it distributes requests across a pool of providers automatically — so if one provider is slow, rate-limited, or down, the router silently shifts traffic to another one.

It builds directly on top of `LiteLLMModel` and adds three extra capabilities:

- **Load balancing** — spreads requests evenly across providers
- **Fallback** — if one provider fails, automatically tries the next
- **Retries with backoff** — retries failed requests with increasing delays before giving up

---

**`model_id`** — A logical group name you choose yourself, like `"my-llama-pool"`. It does not refer to a real model — it is just a label that links your router to the correct entries in `model_list`.

**`model_list`** — The actual pool of providers to route between. Each entry in the list has two parts: `model_name` which must match your `model_id`, and `litellm_params` which contains the real provider model name and its credentials.

**`client_kwargs`** — Configuration for how the router picks between providers. The most important key is `routing_strategy`. Options include `"simple-shuffle"` (picks randomly), `"least-busy"` (picks the least loaded), and `"latency-based-routing"` (picks the fastest).

**`custom_role_conversions`** — Remaps role names like `"system"` to `"user"` for providers that don't support certain roles. Same as in `LiteLLMModel`.

**`flatten_messages_as_text`** — Flattens structured messages to plain text. Auto-enabled for `"ollama"`, `"groq"`, and `"cerebras"`.

**`**kwargs`** — Extra arguments like `temperature` and `max_tokens` forwarded to every completion call.

---

```python
import os
from smolagents import LiteLLMRouterModel, CodeAgent
from smolagents.tools import WebSearchTool

# Define a pool of two providers under one group name "my-llama-pool"
# The router will distribute requests between Groq and Cerebras automatically
model = LiteLLMRouterModel(
    model_id="my-llama-pool",       # your chosen group label
    model_list=[
        {
            "model_name": "my-llama-pool",   # must match model_id
            "litellm_params": {
                "model": "groq/llama-3.3-70b",
                "api_key": os.getenv("GROQ_API_KEY"),
            },
        },
        {
            "model_name": "my-llama-pool",   # same group, different provider
            "litellm_params": {
                "model": "cerebras/llama-3.3-70b",
                "api_key": os.getenv("CEREBRAS_API_KEY"),
            },
        },
    ],
    client_kwargs={
        "routing_strategy": "simple-shuffle",  # randomly pick between the two providers
    },
    temperature=0.5,
    max_tokens=1000,
)

agent = CodeAgent(tools=[WebSearchTool()], model=model)
agent.run("What is the speed of a leopard in km/h?")
```

---

Both providers above run the same `llama-3.3-70b` model — the router just decides which one handles each request. Swap in `"latency-based-routing"` to always pick whichever provider is responding fastest.


###  🧩 — `OpenAIModel`

---


This class lets you call any OpenAIServer compatible model. 

---

**`model_id`** — The model identifier to use on the server (e.g. “gpt-5”).

**`api_base`** — The base URL of the OpenAI-compatible server to send requests to.

**`api_key`** — The API key for authenticating with the server. Falls back to the `OPENAI_API_KEY` environment variable if not provided.

**`organization`** — An optional OpenAI organization ID to associate the request with. Only relevant for OpenAI's own API.

**`project`** — An optional OpenAI project ID to associate the request with. Only relevant for OpenAI's own API.

**`client_kwargs`** — A dictionary of additional arguments passed directly to the `openai.OpenAI` client, such as `max_retries` or `timeout`.

**`custom_role_conversions`** — A dictionary to remap message role names for models that do not support certain roles, e.g. `{"system": "user"}`.

**`flatten_messages_as_text`** — Whether to flatten structured message objects into plain text before sending. Defaults to `False`.

**`**kwargs`** — Additional arguments forwarded to every completion call, such as `temperature`, `max_tokens`, or `top_p`.

---

```python
import os
from smolagents import OpenAIModel, CodeAgent

# Pointing to OpenAI's own server
openai_model = OpenAIModel(
    model_id="gpt-4o",
    api_base="https://api.openai.com/v1",
    api_key=os.environ["OPENAI_API_KEY"],
    organization="my-org-id",       # optional
    project="my-project-id",        # optional
    client_kwargs={"max_retries": 3},
    custom_role_conversions={"system": "user"},
    temperature=0.7,
    max_tokens=1000,
    top_p=0.9,
)

# Pointing to a local vLLM server instead — just change api_base
local_model = OpenAIModel(
    model_id="mistral-7b",
    api_base="http://localhost:8000/v1",
    api_key="not-needed",
    temperature=0.5,
    max_tokens=500,
)

agent = CodeAgent(tools=[], model=openai_model)
agent.run("Explain the difference between TCP and UDP.")
```

---

### 🧩 — `AzureOpenAIModel`

---

`AzureOpenAIModel` allows Smolagents to connect directly to Azure OpenAI deployments through Microsoft's Azure-hosted OpenAI service. It provides the same agent experience as standard OpenAI models while benefiting from Azure-specific features such as enterprise security, regional deployment control, compliance requirements, and integration with existing Azure infrastructure.

Unlike `OpenAIServerModel`, which works with any OpenAI-compatible endpoint, `AzureOpenAIModel` is specifically designed for Azure OpenAI resources and handles Azure-specific authentication and API versioning automatically.

Environment variables can be used instead of passing credentials directly:

- `AZURE_OPENAI_ENDPOINT`
- `AZURE_OPENAI_API_KEY`
- `OPENAI_API_VERSION`

Note that Azure uses `OPENAI_API_VERSION` rather than `AZURE_OPENAI_API_VERSION` because of the underlying OpenAI SDK implementation.

---

Parameters

**`model_id`** — The Azure deployment name to use. This is the deployment you created inside Azure OpenAI Studio, not necessarily the underlying model name. For example, a deployment named `"gpt-4o-mini"` may internally point to a GPT-4o Mini model.

**`azure_endpoint`** — The Azure OpenAI resource endpoint URL, such as:

```text
https://your-resource.openai.azure.com/
```

If omitted, the value is automatically loaded from the `AZURE_OPENAI_ENDPOINT` environment variable.

**`api_key`** — Azure OpenAI API key used for authentication. If not supplied, Smolagents attempts to load it from the `AZURE_OPENAI_API_KEY` environment variable.

**`api_version`** — Azure OpenAI REST API version used for requests. Different Azure features may require specific API versions. If omitted, the value is loaded from the `OPENAI_API_VERSION` environment variable.

**`client_kwargs`** — Additional configuration options passed directly to the underlying Azure OpenAI client. Useful for customizing behavior such as retry settings, timeout values, organization information, project settings, or networking configuration.

**`custom_role_conversions`** — Maps unsupported message roles to alternative roles. Useful when connecting to models that do not fully support OpenAI chat role formats such as `"system"` messages.

**`**kwargs`** — Additional parameters forwarded to every completion request. Common examples include:

- `temperature`
- `max_tokens`
- `top_p`
- `frequency_penalty`
- `presence_penalty`
- `stop`

---

Notes

- Requires an active Azure OpenAI resource.
- The deployment name (`model_id`) must already exist in Azure.
- Supports all Azure-hosted OpenAI models exposed through your deployment.
- Environment variables are recommended for production deployments to avoid hardcoding credentials.
- Azure API versions occasionally introduce new capabilities, so keeping `OPENAI_API_VERSION` current is important.

---

#### Example

```python
import os

from smolagents import AzureOpenAIModel, CodeAgent
from smolagents.tools import WebSearchTool

model = AzureOpenAIModel(
    model_id=os.environ["AZURE_OPENAI_MODEL"],
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version=os.environ["OPENAI_API_VERSION"],
    temperature=0.3,
    max_tokens=2000,
)

agent = CodeAgent(
    tools=[WebSearchTool()],
    model=model,
)

agent.run(
    "Summarize the latest developments in quantum computing."
)
```

---
````md
## smolagents — `AzureOpenAIModel`

---

`AzureOpenAIModel` allows Smolagents to connect directly to Azure OpenAI deployments through Microsoft's Azure-hosted OpenAI service. It provides the same agent experience as standard OpenAI models while benefiting from Azure-specific features such as enterprise security, regional deployment control, compliance requirements, and integration with existing Azure infrastructure.

Unlike `OpenAIServerModel`, which works with any OpenAI-compatible endpoint, `AzureOpenAIModel` is specifically designed for Azure OpenAI resources and handles Azure-specific authentication and API versioning automatically.

Environment variables can be used instead of passing credentials directly:

- `AZURE_OPENAI_ENDPOINT`
- `AZURE_OPENAI_API_KEY`
- `OPENAI_API_VERSION`

Note that Azure uses `OPENAI_API_VERSION` rather than `AZURE_OPENAI_API_VERSION` because of the underlying OpenAI SDK implementation.

---

### Parameters

**`model_id`** — The Azure deployment name to use. This is the deployment you created inside Azure OpenAI Studio, not necessarily the underlying model name. For example, a deployment named `"gpt-4o-mini"` may internally point to a GPT-4o Mini model.

**`azure_endpoint`** — The Azure OpenAI resource endpoint URL, such as:

```text
https://your-resource.openai.azure.com/
```

If omitted, the value is automatically loaded from the `AZURE_OPENAI_ENDPOINT` environment variable.

**`api_key`** — Azure OpenAI API key used for authentication. If not supplied, Smolagents attempts to load it from the `AZURE_OPENAI_API_KEY` environment variable.

**`api_version`** — Azure OpenAI REST API version used for requests. Different Azure features may require specific API versions. If omitted, the value is loaded from the `OPENAI_API_VERSION` environment variable.

**`client_kwargs`** — Additional configuration options passed directly to the underlying Azure OpenAI client. Useful for customizing behavior such as retry settings, timeout values, organization information, project settings, or networking configuration.

**`custom_role_conversions`** — Maps unsupported message roles to alternative roles. Useful when connecting to models that do not fully support OpenAI chat role formats such as `"system"` messages.

**`**kwargs`** — Additional parameters forwarded to every completion request. Common examples include:

- `temperature`
- `max_tokens`
- `top_p`
- `frequency_penalty`
- `presence_penalty`
- `stop`

---

### Notes

- Requires an active Azure OpenAI resource.
- The deployment name (`model_id`) must already exist in Azure.
- Supports all Azure-hosted OpenAI models exposed through your deployment.
- Environment variables are recommended for production deployments to avoid hardcoding credentials.
- Azure API versions occasionally introduce new capabilities, so keeping `OPENAI_API_VERSION` current is important.

---

### Example

```python
import os

from smolagents import AzureOpenAIModel, CodeAgent
from smolagents.tools import WebSearchTool

model = AzureOpenAIModel(
    model_id=os.environ["AZURE_OPENAI_MODEL"],
    azure_endpoint=os.environ["AZURE_OPENAI_ENDPOINT"],
    api_key=os.environ["AZURE_OPENAI_API_KEY"],
    api_version=os.environ["OPENAI_API_VERSION"],
    temperature=0.3,
    max_tokens=2000,
)

agent = CodeAgent(
    tools=[WebSearchTool()],
    model=model,
)

agent.run(
    "Summarize the latest developments in quantum computing."
)
```

---

### When to Use

✅ Use when:

- Your organization already uses Azure infrastructure.
- You require enterprise compliance and governance controls.
- Models must be hosted in specific Azure regions.
- You want Azure-managed authentication and monitoring.
- Company policy requires Azure OpenAI instead of direct OpenAI access.

❌ Avoid when:

- You need provider-agnostic model switching.
- You are running local models.
- You do not have an Azure OpenAI deployment configured.
- You want to route across multiple providers simultaneously (use `LiteLLMRouterModel` instead).
````

---


### 🧩 — `AmazonBedrockModel`

---

`AmazonBedrockModel` connects Smolagents to Amazon Bedrock, AWS's managed platform for foundation models. Through a single interface, it provides access to models from multiple providers including Anthropic Claude, Amazon Nova, Meta Llama, Cohere Command, AI21 Labs, Mistral, and others available within Bedrock.

Instead of integrating separately with each model vendor, Bedrock exposes a unified API. `AmazonBedrockModel` allows Smolagents to use those models while taking advantage of AWS security, IAM permissions, guardrails, monitoring, and regional deployment controls.

The class supports custom AWS clients, automatic client creation, inference configuration, moderation guardrails, and model-specific parameters through the Bedrock Converse API.

---

Parameters

**`model_id`** — The Bedrock model identifier to use. Examples:

```text
us.amazon.nova-pro-v1:0
anthropic.claude-3-haiku-20240307-v1:0
meta.llama3-70b-instruct-v1:0
```

This determines which Bedrock-hosted model receives requests.

**`client`** — An existing boto3 Bedrock Runtime client. Supplying your own client gives full control over authentication, networking, regions, retry strategies, and AWS configuration.

**`client_kwargs`** — Configuration used when Smolagents creates the boto3 client automatically. Common options include:

- `region_name`
- `endpoint_url`
- `config`
- custom AWS networking settings

**`custom_role_conversions`** — Maps chat roles to alternative roles when required by specific Bedrock models. By default, Smolagents converts roles in a way that maximizes compatibility across Bedrock providers.

**`flatten_messages_as_text`** — When enabled, converts structured message formats into plain text before sending requests. Useful for models that perform better with simple text prompts instead of role-based conversations.

**`**kwargs`** — Additional parameters forwarded directly to the Bedrock Converse API. Examples include:

- inference configuration
- guardrail configuration
- generation limits
- provider-specific settings

---

Authentication

Amazon Bedrock supports multiple authentication methods.

 Default AWS Credentials

VARIOUS ACCESS approach OF AWS's credential chain:

- IAM roles
- IAM users
- AWS CLI credentials
- EC2 instance profiles
- ECS task roles
- EKS service accounts

No additional configuration is required if AWS credentials are already available.

API Key Authentication

Newer versions of boto3 support Bedrock API-key authentication through:

```bash
AWS_BEARER_TOKEN_BEDROCK=<token>
```

Requirements:

- boto3 ≥ 1.39.0

For standard IAM authentication:

- boto3 ≥ 1.36.18



Inference Configuration

Additional model-generation settings can be supplied through API parameters.

Example:

```python
additional_api_config = {
    "inferenceConfig": {
        "maxTokens": 3000,
        "temperature": 0.7
    }
}
```

These settings are passed directly to the Bedrock Converse API and may vary depending on the selected model provider.

---

Guardrails

Amazon Bedrock supports built-in safety and governance controls through Guardrails.

Example:

```python
guardrailConfig = {
    "guardrailIdentifier": "guardrail-123",
    "guardrailVersion": "v1"
}
```

Guardrails can help:

- Filter unsafe outputs
- Block sensitive content
- Enforce compliance policies
- Apply organization-specific restrictions

---

Notes

- Supports all Bedrock-compatible foundation models.
- Uses the Bedrock Converse API for unified interactions.
- Works with AWS IAM authentication and API-key authentication.
- Allows provider switching without changing agent code.
- Supports AWS-native monitoring and governance.
- Different providers may support different generation parameters.

---

#### Example

```python
from smolagents import AmazonBedrockModel, CodeAgent
from smolagents.tools import WebSearchTool

model = AmazonBedrockModel(
    model_id="us.amazon.nova-pro-v1:0",
    client_kwargs={
        "region_name": "us-west-2"
    },
    inferenceConfig={
        "maxTokens": 2000
    }
)

agent = CodeAgent(
    tools=[WebSearchTool()],
    model=model,
)

agent.run(
    "Create a market analysis for electric vehicle adoption."
)
```

---

#### Example with Custom boto3 Client

```python
import boto3

from smolagents import AmazonBedrockModel

client = boto3.client(
    "bedrock-runtime",
    region_name="us-west-2"
)

model = AmazonBedrockModel(
    model_id="anthropic.claude-3-haiku-20240307-v1:0",
    client=client,
)
```

---
### 🧩 — `MLXModel`

---

`MLXModel` allows Smolagents to run large language models locally on Apple Silicon devices using Apple's MLX framework. Instead of sending requests to a cloud API, models are downloaded and executed directly on your Mac, providing lower latency, offline operation, improved privacy, and no per-token usage costs.

MLX is Apple's machine learning framework optimized specifically for Apple Silicon chips (M1, M2, M3, M4, etc.). It takes advantage of unified memory architecture and Apple GPUs to efficiently run modern language models.

Unlike cloud-based models such as OpenAI or Bedrock, `MLXModel` performs inference entirely on your local machine.

You must install MLX support before using this class:

```bash
pip install "smolagents[mlx-lm]"
```

---

Parameters

**`model_id`** — The Hugging Face model identifier or local model path to load. This can be any MLX-compatible model hosted on the Hugging Face Hub or stored locally.

Examples:

```text
HuggingFaceTB/SmolLM-135M-Instruct
mlx-community/Qwen2.5-Coder-32B-Instruct-4bit
mlx-community/Llama-3.1-8B-Instruct-4bit
```

**`tool_name_key`** — The field name used by the model's chat template to represent tool names during tool calling. Some models use different naming conventions, so this parameter helps Smolagents correctly parse tool invocation requests.

**`tool_arguments_key`** — The field name used by the model's chat template to represent tool arguments. This ensures proper extraction of tool parameters from generated tool calls.

**`trust_remote_code`** — Whether to allow execution of custom Python code provided by a model repository on Hugging Face. Some advanced models require custom loading logic and will not function unless this is enabled.

For security reasons, this defaults to `False`.

**`load_kwargs`** — Additional arguments forwarded directly to `mlx.lm.load()` when loading the model and tokenizer.

Common examples include:

- quantization settings
- tokenizer configuration
- model loading optimizations
- memory management settings

**`apply_chat_template_kwargs`** — Additional parameters passed to the tokenizer's `apply_chat_template()` method before inference.

Useful for customizing:

- generation prompts
- system message handling
- conversation formatting
- tool-calling templates

**`**kwargs`** — Additional parameters forwarded to the underlying MLX generation engine.

Common options include:

- `max_tokens`
- `temperature`
- `top_p`
- `top_k`
- `repetition_penalty`

---

Requirements

MLX only runs on Apple Silicon hardware.

Supported devices include:

- Apple M1
- Apple M2
- Apple M3
- Apple M4

Intel-based Macs are not supported.

Install required dependencies:

```bash
pip install "smolagents[mlx-lm]"
```

---

Advantages

Running locally with MLX provides several benefits:

- No API costs
- Offline operation
- Improved privacy
- Lower latency
- No internet dependency
- Direct access to local models
- Optimized Apple GPU utilization

---

Notes

- Best suited for Mac users with Apple Silicon.
- Large models may require substantial RAM.
- Quantized models (4-bit, 8-bit) typically offer better performance and lower memory usage.
- Inference speed depends on model size and available system memory.
- Tool-calling support depends on the selected model's chat template.

---

#### Example

```python
from smolagents import MLXModel

model = MLXModel(
    model_id="mlx-community/Qwen2.5-Coder-32B-Instruct-4bit",
    max_tokens=4000,
)

messages = [
    {
        "role": "user",
        "content": "Explain quantum mechanics in simple terms."
    }
]

response = model(
    messages,
    stop_sequences=["END"]
)

print(response)
```

---

### 🧩 — `VLLMModel`

---

`VLLMModel` allows Smolagents to run language models locally using vLLM, a high-performance inference engine designed for fast and scalable LLM serving. It is optimized for GPU acceleration and is widely used in production systems where throughput, latency, and memory efficiency are critical.

Unlike standard Transformers inference, vLLM implements advanced techniques such as PagedAttention and optimized memory management, allowing significantly higher request throughput while serving large models.

`VLLMModel` is particularly useful when deploying local agents, self-hosted AI services, or high-volume inference workloads.

Before using this class, install vLLM support:

```bash
pip install "smolagents[vllm]"
```

---

 Parameters

**`model_id`** — The Hugging Face model identifier or local model path to load.

Examples:

```text
HuggingFaceTB/SmolLM-135M-Instruct
meta-llama/Llama-3.1-8B-Instruct
Qwen/Qwen2.5-Coder-32B-Instruct
mistralai/Mistral-7B-Instruct-v0.3
```

**`model_kwargs`** — Additional arguments forwarded directly to the vLLM `LLM` constructor.

Common examples include:

- `revision`
- `max_model_len`
- `tensor_parallel_size`
- `gpu_memory_utilization`
- `dtype`
- `trust_remote_code`

These settings control model loading behavior and hardware utilization.

**`apply_chat_template_kwargs`** — Additional parameters passed to the tokenizer's `apply_chat_template()` method.

Useful for:

- custom prompt formatting
- tool-calling templates
- system message handling
- conversation preprocessing

**`**kwargs`** — Additional generation parameters forwarded directly to the vLLM generation engine.

Examples:

- `max_tokens`
- `temperature`
- `top_p`
- `top_k`
- `presence_penalty`
- `frequency_penalty`

---
Why vLLM Is Fast

Traditional inference engines repeatedly allocate and manage large memory blocks during generation.

vLLM introduces:

PagedAttention

Efficient KV-cache management that reduces memory fragmentation and increases utilization.

 Continuous Batching

New requests can join active batches without waiting for current generations to finish.

Optimized GPU Scheduling

Improves throughput for concurrent users and production workloads.

Together these techniques allow significantly higher request throughput compared to standard Transformers inference.

---

Requirements

Install vLLM support:

```bash
pip install "smolagents[vllm]"
```

Most deployments use NVIDIA GPUs.

Typical production hardware:

- RTX 4090
- A100
- H100
- L40S
- Multi-GPU clusters

CPU-only execution is generally not recommended for large models.

---

Notes

- Designed primarily for high-performance inference.
- Frequently used in self-hosted AI platforms.
- Supports many Hugging Face models.
- Excellent choice for serving multiple concurrent users.
- Memory requirements depend on model size and context length.
- Particularly effective for long-context and large-scale deployments.

---

#### Example

```python
from smolagents import VLLMModel

model = VLLMModel(
    model_id="meta-llama/Llama-3.1-8B-Instruct",
    model_kwargs={
        "max_model_len": 8192
    },
    max_tokens=2000,
)

response = model(
    [{"role": "user", "content": "Explain reinforcement learning."}],
    stop_sequences=["END"]
)

print(response)
```

---

# WORKING OF CODEAGENTS
## workflow

![[codeagent_docs (1).png]]


workflow table

| Your ReAct Version                                        | Image Step                                 |
| --------------------------------------------------------- | ------------------------------------------ |
| Receive Task                                              | **Step 1**                                 |
| Log cotextualizing (R,C,S)  & Thinking (Action Generated) | **Step 2.1 + 2.2**                         |
| Act (Action Executed)                                     | **Step 2.3**                               |
| Observe & Log (Actions and Results)                       | **Step 2.4**                               |
| Repeat Loop                                               | **Step 2 (while final_answer not called)** |
| Return Answer                                             | **Step 3**                                 |

---
## STEPS
### Image Step 1 = Receive Task

Image:

```text
1. Put the task into a TaskStep
   and add it to logs
```

Your version:

```text
Receive Task:
"What is the capital of France?"
```

Internally:

```text
logs
├── SystemPromptStep
└── TaskStep
     └── What is the capital of France?
```

---

### Image Step 2 = ReAct Loop

Image:

```text
while final_answer tool has not been called
```

Your version:

```text
ReAct Loop #1
ReAct Loop #2
...
```

Same thing.

The agent keeps cycling until it reaches `final_answer()`.

---

### Image Step 2.1 = Log Reading, converting and sending

Image:

```text
agent.write_inner_memory_from_logs()
```

Meaning:

```text
Read everything in logs
Convert to messages
Send to LLM
```

At Loop #1 logs contain:

```text
SystemPromptStep
TaskStep
```

So LLM sees:

```text
Question:
What is the capital of France?
```

This is the setup before thinking.

---

### Image Step 2.2 = Think

Image:

```text
Messages are sent to model
Model returns answer/code
```

Your version:

```text
Think:
"I should search the web."
```

LLM generates:

```python
web_search("capital of France")
```

This is exactly the model output generated in Step 2.2.

---

### Image Step 2.3 = Act

Image:

```text
The code blob is executed
Any tool calls are run
```

Your version:

```text
Act:
web_search("capital of France")
```

Tool executes.

Returns:

```text
Paris is the capital of France
```

This is Step 2.3.

---

### Image Step 2.4 = Observe & Log

Image:

```text
Execution logs are put into
an ActionStep and appended to logs
```

Your version:

```text
Observation:
Paris is the capital of France

Log:
Action:
web_search(...)

Observation:
Paris is the capital of France
```

Internally:

```text
logs
├── SystemPromptStep
├── TaskStep
└── ActionStep
     ├── Action:
     │    web_search(...)
     │
     └── Observation:
          Paris is the capital of France
```

Exactly Image Step 2.4.

---

### Loop #2 Begins

Image goes back to:

```text
2. while final_answer not called
```

Agent again reads logs.

Now logs contain:

```text
Task:
What is the capital of France?

Observation:
Paris is the capital of France
```

---

### Image Step 2.1 Again

```text
Read logs
Build messages
```

LLM now sees:

```text
Question:
What is the capital of France?

Observation:
Paris is the capital of France
```

---

### Image Step 2.2 Again = Think

Your version:

```text
Think:
"I already know the answer."
```

LLM generates:

```python
final_answer("Paris")
```

---

### Image Step 2.3 Again = Act

Execute:

```python
final_answer("Paris")
```

---

### Image Step 2.4 Again = Log

Store:

```text
Action:
final_answer("Paris")
```

inside another ActionStep.

---

### Image Step 3 = Return Answer

Image:

```text
When final_answer tool has been called,
run() returns its argument
```

Your version:

```text
Return:
Paris
```

Exactly the same.

---

## Complete Mapping

```text
IMAGE                            YOUR VERSION
────────────────────────────────────────────────

Step 1
TaskStep created          ←      Receive Task

Step 2.1
Contextualize logs        ←      Prepare for Think

Step 2.2
Model generates code      ←      Think

Step 2.3
Execute code/tool         ←      Act

Step 2.4
Create ActionStep         ←      Observe & Log

Repeat while loop         ←      Repeat ReAct Loop

Step 3
Return final_answer       ←      Return Answer
```

So the entire image can be mentally compressed to:

```text
1. Receive Task

2. ReAct Loop
   2.1 Contextualize Logs
   2.2 Think (LLM)
   2.3 Act (Tool/Code)
   2.4 Observe & Log

3. Repeat until final_answer()

4. Return Answer
```

That is the image translated into ReAct terminology.


# CHAT TEMPLATES
 
## 1. Introduction

Chat-Templates

 chat templates are essential for **structuring conversations between language models and users**. They guide how message exchanges are formatted into a single prompt.
## 2. Core Concepts

### Complete End-to-End Example

 Developer writes

```
messages = [    {        "role": "user",        "content": "What is Python?"    }]
```

↓

Chat Template

```
<|user|>What is Python?<|assistant|>
```

↓

 Tokenizer

```
[ "<|user|>", "What", " is", " Python", "?", "<|assistant|>"]
```

↓

 Token IDs

```
[ 128006, 392, 374, 11203, 30, 128007]
```

↓

Model Generates

```
[ 578, 11203, 374, 459, 4221, 1665]
```

↓

Tokenizer Decodes

```
Python is a programming language.
```




### - apply_chat_template()

````markdown
#### Anchored Explanation

Remember this pipeline:

```text
Logs
 ↓
Messages
 ↓
Chat Template
 ↓
Tokens
 ↓
Model
````

Using your example:

#### 1. Messages (Python dictionaries)

```python
messages = [
  {"role":"system","content":"You are a pirate"},
  {"role":"user","content":"How many helicopters can a human eat?"}
]
```

---

#### 2. `apply_chat_template()`

Converts those dictionaries into the exact text format the model was trained on:

```text
<|system|>
You are a pirate

<|user|>
How many helicopters can a human eat?

<|assistant|>
```

Think:

```text
Messages → Model-specific Prompt
```

---

#### 3. Tokenizer

Converts prompt text into token IDs:

```text
<|system|> ... → [1, 523, 891, ...]
```

Think:

```text
Prompt → Numbers
```

---

#### 4. Model

Receives token IDs and generates new tokens:

```text
<|assistant|>
Matey, humans cannot eat helicopters...
```

---

#### One-line Memory Hook

```text
Messages
 ↓ apply_chat_template
Formatted Prompt
 ↓ tokenize
Token IDs
 ↓
Model Response
```

So **`apply_chat_template()` = "Messages → Prompt Formatter"**.
### continue_final_message


Tells the model to **continue generating from the content of the last assistant message** instead of **starting a new assistant reply**.

The existing text acts as a **prefix (prefill)**, and the model generates what comes next.

It **continues/appends** the text — it does **not modify, replace, or edit** the existing content.

#### Example

```python
chat = [
    {"role": "user", "content": "Give JSON"},
    {"role": "assistant", "content": '{"name": "'}
]

formatted_chat = tokenizer.apply_chat_template(
    chat,
    tokenize=True,
    continue_final_message=True
)
```

The model continues from:

```json
{"name": "
```

and may generate:

```json
John"}
```

Final output:

```json
{"name":"John"}
```

#### Memory Hook

```text
add_generation_prompt=True
→ Start a new assistant message

continue_final_message=True
→ Continue the current assistant message
```

#### Flow

```text
Existing Assistant Message
        ↓
continue_final_message=True
        ↓
Model appends the next tokens
```
###  `add_generation_prompt` 

#### Concept: 
`add_generation_prompt=True` appends the model's **assistant start marker** (e.g., `<|im_start|>assistant`) to the end of the formatted chat, signaling that the assistant's turn begins and generation should start from that point.

```python
prompt = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)
```

#### Example Output

```text
<|im_start|>user
Can I ask a question?<|im_end|>
<|im_start|>assistant
```

#### Memory Hook

```text
add_generation_prompt
        ↓
Assistant Start Marker
        ↓
"Assistant speaks next"
        ↓
Model generates reply
```


### Model Training with Chat Templates

During training, the chat template is applied to every conversation in the dataset so that the training data matches the exact format expected by the model.

Use:

```python
add_generation_prompt=False
```

because the dataset already contains the assistant's actual response, so there is no need to add a token that starts a new assistant message.

#### Example

```python
from transformers import AutoTokenizer
from datasets import Dataset

tokenizer = AutoTokenizer.from_pretrained(
    "HuggingFaceH4/zephyr-7b-beta"
)

chat1 = [
    {"role": "user", "content": "Which is bigger, the moon or the sun?"},
    {"role": "assistant", "content": "The sun."}
]

dataset = Dataset.from_dict({"chat": [chat1]})

dataset = dataset.map(
    lambda x: {
        "formatted_chat": tokenizer.apply_chat_template(
            x["chat"],
            tokenize=False,
            add_generation_prompt=False
        )
    }
)
```

Produces:

```text
<|user|>
Which is bigger, the moon or the sun?</s>

<|assistant|>
The sun.</s>
```

#### Why `add_generation_prompt=False`?

Training data already contains:

```text
User Question
Assistant Answer
```

Adding:

```text
<|assistant|>
```

at the end would incorrectly tell the model that another assistant response should start.

#### thumb rule

```
 Conversation ends with User
→ add_generation_prompt=True

Conversation already contains Assistant Answer
→ add_generation_prompt=False 
```


#### Memory Hook

```text
Inference
→ add_generation_prompt=True
→ Model needs to generate a new answer

Training
→ add_generation_prompt=False
→ Answer already exists in dataset
```

#### Training Pipeline

```text
Chat Dataset
      ↓
apply_chat_template()
      ↓
Formatted Chat Text
      ↓
Tokenization
      ↓
Training (Next-Token Prediction)
```



# Tools

A **Tool is a function given to the LLM**. This function should fulfill a **clear objective**

To interact with a tool, the LLM needs an **interface description** with these key components:

- **Name:** `web_search`
- **Tool description:** Searches the web for specific queries
- **Input:** `query` (string) - The search term to look up
- **Output:** String containing the search results

### Tool Creation Methods

In `smolagents`, tools can be defined in two ways:

1. **Using the `@tool` decorator** for simple function-based tools
2. **Creating a subclass of `Tool`** for more complex functionality

#### [](https://huggingface.co/learn/agents-course/unit2/smolagents/tools#the-tool-decorator)The @tool Decorator

The `@tool` decorator is the **recommended way to define simple tools**. Under the hood, smolagents will parse basic information about the function from Python. So if you name your function clearly and write a good docstring, it will be easier for the LLM to use.

### Defining a Tool as a Python Class

This approach involves creating a subclass of [`Tool`](https://huggingface.co/docs/smolagents/v1.8.1/en/reference/tools#smolagents.Tool). For complex tools, we can implement a class instead of a Python function. The class wraps the function with metadata that helps the LLM understand how to use it effectively. In this class, we define:

- `name`: The tool’s name.
- `description`: A description used to populate the agent’s system prompt.
- `inputs`: A dictionary with keys `type` and `description`, providing information to help the Python interpreter process inputs.
- `output_type`: Specifies the expected output type.
- `forward`: The method containing the inference logic to execute.



# RETREIVAL AUGMENTATION SYSTEM


### RAG
Retrieval Augmented Generation (RAG) systems combine the capabilities of data retrieval and generation models to provide context-aware responses.

### BASIC RETREIVAL

The agent follows this process:

1. **Analyzes the Request:** Alfred’s agent identifies the key elements of the query—luxury superhero-themed party planning, with focus on decor, entertainment, and catering.
2. **Performs Retrieval:** The agent leverages DuckDuckGo to search for the most relevant and up-to-date information, ensuring it aligns with Alfred’s refined preferences for a luxurious event.
3. **Synthesizes Information:** After gathering the results, the agent processes them into a cohesive, actionable plan for Alfred, covering all aspects of the party.
4. **Stores for Future Reference:** The agent stores the retrieved information for easy access when planning future events, optimizing efficiency in subsequent tasks.

### ENHANCED RETREIVAL WITH CUSTOM KNOWLEDGE BASE TOOL

1. **Query Reformulation:** Instead of using the raw user query, the agent can craft optimized search terms that better match the target documents
2. **Query Decomposition:** Instead of using the user query directly, if it contains multiple pieces of information to query, it can be decomposed to multiple queries
3. **Query Expansion:** Somehow similar to Query Reformulation but done multiple times to put the query in multiple wordings to query them all
4. **Reranking:** Using Cross-Encoders to assign more comprehensive and semantic relevance scores between retrieved documents and search query
5. **Multi-Step Retrieval:** The agent can perform multiple searches, using initial results to inform subsequent queries
6. **Source Integration:** Information can be combined from multiple sources like web search and local documentation
7. **Result Validation:** Retrieved content can be analyzed for relevance and accuracy before being included in responses

