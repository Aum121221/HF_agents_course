##in
An Agent is a system that leverages an AI model to interact with its environment in order to achieve a user-defined objective. It combines reasoning, planning, and the execution of actions (often via external tools) to fulfill tasks.

![Image](Huggingface%20ai/images/Pasted%20image%2020260418194004.png)

. The AI model **handles reasoning and planning**. It decides **which Actions to take based on the situation**.

An LLM is a type of AI model that excels at **understanding and generating human language**. They are trained on vast amounts of text data, allowing them to learn patterns, structure, and even nuance in language. These models typically consist of many millions of parameters.

Alright, let’s go technical with a **beam search decoding example in Python using Hugging Face’s Transformers**. This will show you exactly how the parameters you listed affect output.


Most LLMs nowadays are **built on the Transformer architecture**—a deep learning architecture based on the “Attention” algorithm, that has gained significant interest since the release of BERT from Google in 2018.

![Transformer](https://huggingface.co/datasets/agents-course/course-images/resolve/main/en/unit1/transformer.jpg)
The original Transformer architecture looked like this, with an encoder on the left and a decoder on the right.

There are 3 types of transformers:

1. **Encoders**  
    An encoder-based Transformer takes text (or other data) as input and outputs a dense representation (or embedding) of that text.
    
    - **Example**: BERT from Google
    - **Use Cases**: Text classification, semantic search, Named Entity Recognition
    - **Typical Size**: Millions of parameters
2. **Decoders**  
    A decoder-based Transformer focuses **on generating new tokens to complete a sequence, one token at a time**.
    
    - **Example**: Llama from Meta
    - **Use Cases**: Text generation, chatbots, code generation
    - **Typical Size**: Billions (in the US sense, i.e., 10^9) of parameters
3. **Seq2Seq (Encoder–Decoder)**  
    A sequence-to-sequence Transformer _combines_ an encoder and a decoder. The encoder first processes the input sequence into a context representation, then the decoder generates an output sequence.
    
    - **Example**: T5, BART
    - **Use Cases**: Translation, Summarization, Paraphrasing
    - **Typical Size**: Millions of parameters

## Beam Search Visualiser
#### Parameters:

- **Sentence to decode from** (`inputs`): the input sequence to your decoder.
- **Number of steps** (`max_new_tokens`): the number of tokens to generate.
- **Number of beams** (`num_beams`): the number of beams to use.
- **Length penalty** (`length_penalty`): the length penalty to apply to outputs. `length_penalty` > 0.0 promotes longer sequences, while `length_penalty` < 0.0 encourages shorter sequences. This parameter will not impact the beam search paths, but only influence the choice of sequences in the end towards longer or shorter sequences.
- **Number of return sequences** (`num_return_sequences`): the number of sequences to be returned at the end of generation. Should be `<= num_beams`.





---

## ⚙️ Example: Beam Search with GPT‑2

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer

# Load pretrained GPT-2
model_name = "gpt2"
tokenizer = GPT2Tokenizer.from_pretrained(model_name)
model = GPT2LMHeadModel.from_pretrained(model_name)

# Input sentence
input_text = "The cat sat on the"
input_ids = tokenizer.encode(input_text, return_tensors="pt")

# Beam search decoding
outputs = model.generate(
    input_ids,
    max_new_tokens=10,        # number of steps (tokens to generate)
    num_beams=3,              # number of beams (parallel paths)
    length_penalty=1.0,       # >1 favors longer outputs, <1 favors shorter
    num_return_sequences=3,   # how many sequences to return (≤ num_beams)
    early_stopping=True
)

# Decode results
for i, output in enumerate(outputs):
    print(f"Sequence {i+1}: {tokenizer.decode(output, skip_special_tokens=True)}")
```

---

## 📊 Possible Output (illustrative)

With `num_beams=3` and `num_return_sequences=3`, you might see:

1. `"The cat sat on the mat and looked around."`
2. `"The cat sat on the floor near the fireplace."`
3. `"The cat sat on the sofa watching the birds."`

---

## 🔍 Parameter Effects

- **`max_new_tokens=10`** → Limits generation to 10 new words.
- **`num_beams=3`** → Explores 3 parallel continuations instead of just the single “greedy” one.
- **`length_penalty=1.0`** → Neutral; >1 would push toward longer sentences, <1 toward shorter.
- **`num_return_sequences=3`** → Returns all 3 beams’ best completions.

---

## 🧠 Connecting to Transformer Types

- **Decoder models (GPT‑2, Llama)** → Use beam search for text generation.
- **Encoder models (BERT)** → Don’t generate text, so beam search isn’t relevant.
- **Seq2Seq models (T5, BART)** → Use beam search for translation/summarization, e.g. multiple candidate summaries.

---

The table below illustrates the diversity of special tokens.

|**Model**|**Provider**|**EOS Token**|**Functionality**|
|---|---|---|---|
|**GPT4**|OpenAI|`<\|endoftext\|>`|End of message text|
|**Llama 3**|Meta (Facebook AI Research)|`<\|eot_id\|>`|End of sequence|
|**Deepseek-R1**|DeepSeek|`<\|end_of_sentence\|>`|End of message text|
|**SmolLM2**|Hugging Face|`<\|im_end\|>`|End of instruction or message|
|**Gemma**|Google|`<end_of_turn>`|End of conversation turn|


### System Messages

System messages (also called System Prompts) define **how the model should behave**. They serve as **persistent instructions**, guiding every subsequent interaction.

### Conversations: User and Assistant Messages

A conversation consists of alternating messages between a Human (user) and an LLM (assistant).

## Chat-Templates

 chat templates are essential for **structuring conversations between language models and users**. They guide how message exchanges are formatted into a single prompt.

### [](https://huggingface.co/learn/agents-course/unit1/messages-and-special-tokens#base-models-vs-instruct-models)Base Models vs. Instruct Models

Another point we need to understand is the difference between a Base Model vs. an Instruct Model:

- _A Base Model_ is trained on raw text data to predict the next token.
    
- An _Instruct Model_ is fine-tuned specifically to follow instructions and engage in conversations. For example, `SmolLM2-135M` is a base model, while `SmolLM2-135M-Instruct` is its instruction-tuned variant.
    

To make a Base Model behave like an instruct model, we need to **format our prompts in a consistent way that the model can understand**. This is where chat templates come in.



### Understanding Chat Templates

Because each instruct model uses different conversation formats and special tokens, chat templates are implemented to ensure that we correctly format the prompt the way each model expects.


## What are AI Tools?

A **Tool is a function given to the LLM**. This function should fulfill a **clear objective**.

Here are some commonly used tools in AI agents:

|Tool|Description|
|---|---|
|Web Search|Allows the agent to fetch up-to-date information from the internet.|
|Image Generation|Creates images based on text descriptions.|
|Retrieval|Retrieves information from an external source.|
|API Interface|Interacts with an external API (GitHub, YouTube, Spotify, etc.).|

Those are only examples, as you can in fact create a tool for any use case!

A good tool should be something that **complements the power of an LLM**.

The Tool-calling steps are typically not shown to the user: the Agent appends them as a new message before passing the updated conversation to the LLM again. The LLM then processes this additional context and generates a natural-sounding response for the user. From the user’s perspective, it appears as if the LLM directly interacted with the tool, but in reality, it was the Agent that handled the entire execution process in the background.

## How do we give tools to an LLM?

The complete answer may seem overwhelming, but we essentially use the system prompt to provide textual descriptions of available tools to the model:

![System prompt for tools](https://huggingface.co/datasets/agents-course/course-images/resolve/main/en/unit1/Agent_system_prompt.png)

For this to work, we have to be very precise and accurate about:

1. **What the tool does**
2. **What exact inputs it expects**


we will implement a simplified **calculator** tool that will just multiply two integers. This could be our Python implementation:

Copied

def calculator(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b

So our tool is called `calculator`, it **multiplies two integers**, and it requires the following inputs:

- **`a`** (_int_): An integer.
- **`b`** (_int_): An integer.

The output of the tool is another integer number that we can describe like this:

- (_int_): The product of `a` and `b`.

All of these details are important. Let’s put them together in a text string that describes our tool for the LLM to understand:


Tool Name: calculator, Description: Multiply two integers., Arguments: a: int, b: int, Outputs: int


We define a **`Tool`** class that includes:

- **`name`** (_str_): The name of the tool.
- **`description`** (_str_): A brief description of what the tool does.
- **`function`** (_callable_): The function the tool executes.
- **`arguments`** (_list_): The expected input parameters.
- **`outputs`** (_str_ or _list_): The expected outputs of the tool.
- **`__call__()`**: Calls the function when the tool instance is invoked.
- **`to_string()`**: Converts the tool’s attributes into a textual representation.

### Model Context Protocol (MCP): a unified tool interface

Model Context Protocol (MCP) is an **open protocol** that standardizes how applications **provide tools to LLMs**. MCP provides:

- A growing list of pre-built integrations that your LLM can directly plug into
- The flexibility to switch between LLM providers and vendors
- Best practices for securing your data within your infrastructure

This means that **any framework implementing MCP can leverage tools defined within the protocol**, eliminating the need to reimplement the same tool interface for each framework.


## The Core Components

Agents’ work is a continuous cycle of: **thinking (Thought) → acting (Act) and observing (Observe)**.

Let’s break down these actions together:

1. **Thought**: The LLM part of the Agent decides what the next step should be.
2. **Action:** The agent takes an action by calling the tools with the associated arguments.
3. **Observation:** The model reflects on the response from the tool.

## [](https://huggingface.co/learn/agents-course/unit1/agent-steps-and-structure#the-thought-action-observation-cycle)The Thought-Action-Observation Cycle

The three components work together in a continuous loop. To use an analogy from programming, the agent uses a **while loop**: the loop continues until the objective of the agent has been fulfilled.


- **Agents iterate through a loop until the objective is fulfilled:**

**Alfred’s process is cyclical**. It starts with a thought, then acts by calling a tool, and finally observes the outcome. If the observation had indicated an error or incomplete data,and at last it reflects on the data received from tool calling and generates the final answer for user, Alfred could have re-entered the cycle to correct its approach.

## thoughts
Thoughts represent the **Agent’s internal reasoning and planning processes** to solve the task.

## Examples of Common Thought Types

|Type of Thought|Example|
|---|---|
|Planning|“I need to break this task into three steps: 1) gather data, 2) analyze trends, 3) generate report”|
|Analysis|“Based on the error message, the issue appears to be with the database connection parameters”|
|Decision Making|“Given the user’s budget constraints, I should recommend the mid-tier option”|
|Problem Solving|“To optimize this code, I should first profile it to identify bottlenecks”|
|Memory Integration|“The user mentioned their preference for Python earlier, so I’ll provide examples in Python”|
|Self-Reflection|“My last approach didn’t work well, I should try a different strategy”|
|Goal Setting|“To complete this task, I need to first establish the acceptance criteria”|
|Prioritization|“The security vulnerability should be addressed before adding new features”|


## Chain-of-Thought (CoT)

**Chain-of-Thought (CoT)** is a prompting technique that guides a model to **think through a problem step-by-step before producing a final answer**


## ReAct: Reasoning + Acting

A key method is the **ReAct approach**, which combines “Reasoning” (Think) with “Acting” (Act).

ReAct is a prompting technique that encourages the model to think step-by-step and interleave actions (like using tools) between reasoning steps.

This enables the agent to solve complex multi-step tasks by alternating between:

- Thought: internal reasoning
- Action: tool usage
- Observation: receiving tool output

### [](https://huggingface.co/learn/agents-course/unit1/thoughts#-example-react)🔄 Example (ReAct)

Copied

Thought: I need to find the latest weather in Paris.
Action: Search["weather in Paris"]
Observation: It's 18°C and cloudy.
Thought: Now that I know the weather...
Action: Finish["It's 18°C and cloudy in Paris."]

## 🔁 Comparison: ReAct vs. CoT

|Feature|Chain-of-Thought (CoT)|ReAct|
|---|---|---|
|Step-by-step logic|✅ Yes|✅ Yes|
|External tools|❌ No|✅ Yes (Actions + Observations)|
|Best suited for|Logic, math, internal tasks|Info-seeking, dynamic multi-step tasks|


# Actions: Enabling the Agent to Engage with Its Environment


Actions are the concrete steps an **AI agent takes to interact with its environment**.

## [](https://huggingface.co/learn/agents-course/unit1/actions#types-of-agent-actions)Types of Agent Actions

There are multiple types of Agents that take actions differently:

|Type of Agent|Description|
|---|---|
|JSON Agent|The Action to take is specified in JSON format.|
|Code Agent|The Agent writes a code block that is interpreted externally.|
|Function-calling Agent|It is a subcategory of the JSON Agent which has been fine-tuned to generate a new message for each action.|

Actions themselves can serve many purposes:

|Type of Action|Description|
|---|---|
|Information Gathering|Performing web searches, querying databases, or retrieving documents.|
|Tool Usage|Making API calls, running calculations, and executing code.|
|Environment Interaction|Manipulating digital interfaces or controlling physical devices.|
|Communication|Engaging with users via chat or collaborating with other agents.|

The LLM only handles text and uses it to describe the action it wants to take and the parameters to supply to the tool. For an agent to work properly, the LLM must STOP generating new tokens after emitting all the tokens to define a complete Action. This passes control from the LLM back to the agent and ensures the result is parseable - whether the intended format is JSON, code, or function-calling.

## [](https://huggingface.co/learn/agents-course/unit1/actions#the-stop-and-parse-approach)The Stop and Parse Approach

One key method for implementing actions is the **stop and parse approach**. This method ensures that the agent’s output is structured and predictable:

1. **Generation in a Structured Format**:

The agent outputs its intended action in a clear, predetermined format (JSON or code).

2. **Halting Further Generation**:

Once the text defining the action has been emitted, **the LLM stops generating additional tokens**. This prevents extra or erroneous output.

3. **Parsing the Output**:

An external parser reads the formatted action, determines which Tool to call, and extracts the required parameters.

For example, an agent needing to check the weather might output:


Thought: I need to check the current weather for New York.
Action :
{
  "action": "get_weather",
  "action_input": {"location": "New York"}
}

The framework can then easily parse the name of the function to call and the arguments to apply.


Note: Function-calling agents operate similarly by structuring each action so that a designated function is invoked with the correct arguments. 

## [](https://huggingface.co/learn/agents-course/unit1/actions#code-agents)Code Agents

An alternative approach is using _Code Agents_. The idea is: **instead of outputting a simple JSON object**, a Code Agent generates an **executable code block—typically in a high-level language like Python**.

![Code Agents](https://huggingface.co/datasets/agents-course/course-images/resolve/main/en/unit1/code-vs-json-actions.png)

This approach offers several advantages:

- **Expressiveness:** Code can naturally represent complex logic, including loops, conditionals, and nested functions, providing greater flexibility than JSON.
- **Modularity and Reusability:** Generated code can include functions and modules that are reusable across different actions or tasks.
- **Enhanced Debuggability:** With a well-defined programming syntax, code errors are often easier to detect and correct.
- **Direct Integration:** Code Agents can integrate directly with external libraries and APIs, enabling more complex operations such as data processing or real-time decision making.

You must keep in mind that executing LLM-generated code may pose security risks, from prompt injection to the execution of harmful code. That’s why it’s recommended to use AI agent frameworks like `smolagents` that integrate default safeguards. If you want to know more about the risks and how to mitigate them, [please have a look at this dedicated section](https://huggingface.co/docs/smolagents/tutorials/secure_code_execution).

For example, a Code Agent tasked with fetching the weather might generate the following Python snippet:

#Code Agent Example: Retrieve Weather Information
def get_weather(city):
    import requests
    api_url = f"https://api.weather.com/v1/location/{city}?apiKey=YOUR_API_KEY"
    response = requests.get(api_url)
    if response.status_code == 200:
        data = response.json()
        return data.get("weather", "No weather information available")
    else:
        return "Error: Unable to fetch weather data."

#Execute the function and prepare the final answer
result = get_weather("New York")
final_answer = f"The current weather in New York is: {result}"
print(final_answer)

In this example, the Code Agent:

- Retrieves weather data **via an API call**,
- Processes the response,
- And uses the print() function to output a final answer.

This method **also follows the stop and parse approach** by clearly delimiting the code block and signaling when execution is complete (here, by printing the final_answer).


# Observe: Integrating Feedback to Reflect and Adapt

Observations are **how an Agent perceives the consequences of its actions**


# Observe: Integrating Feedback to Reflect and Adapt

Observations are **how an Agent perceives the consequences of its actions**.

They provide crucial information that fuels the Agent’s thought process and guides future actions.

They are **signals from the environment**—whether it’s data from an API, error messages, or system logs—that guide the next cycle of thought.

In the observation phase, the agent:

- **Collects Feedback:** Receives data or confirmation that its action was successful (or not). This can be seen like Tool “logs” that provide textual feedback of the Action execution.
- **Appends Results:** Integrates the new information into its existing context, effectively updating its memory.
- **Adapts its Strategy:** Uses this updated context to refine subsequent thoughts and actions.

## How Are the Results Appended?

After performing an action, the framework follows these steps in order:

1. **Parse the action** to identify the function(s) to call and the argument(s) to use.
2. **Execute the action.**
3. **Append the result** as an **Observation**

