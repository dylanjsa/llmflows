
<p align="center">
  <img src="https://github.com/stoyan-stoyanov/llmflows/blob/main/docs/logo_with_subheading.png"/>
</p>


[![Twitter](https://img.shields.io/twitter/follow/LLMFlows?style=social)](https://twitter.com/LLMFlows)
![Pylint workflow](https://github.com/stoyan-stoyanov/llmflows/actions/workflows/pylint.yml/badge.svg)
![License](https://img.shields.io/github/license/stoyan-stoyanov/llmflows)
![PyPi](https://img.shields.io/pypi/v/llmflows)
![Stars](https://img.shields.io/github/stars/stoyan-stoyanov/llmflows?style=social)
![Release date](https://img.shields.io/github/release-date/stoyan-stoyanov/llmflows?style=social)

Documentation: <a href="https://llmflows.readthedocs.io/" target="_blank">https://llmflows.readthedocs.io</a></br>
PyPI: <a href="https://pypi.org/project/llmflows/" target="_blank">https://pypi.org/project/llmflows</a></br>
Twitter: <a href="https://twitter.com/LLMFlows/" target="_blank">https://twitter.com/LLMFlows</a></br>
Substack: <a href="https://llmflows.substack.com/" target="_blank">https://llmflows.substack.com</a></br>

## 🤖 About LLM Flows
LLMFlows is a framework for building simple, explicit, and transparent LLM(Large Language Model) applications such as chatbots, question-answering systems, and agents.

At its core, LLMFlows provides a minimalistic set of abstractions that allow you to utilize LLMs and vector stores and build well-structured and explicit apps that don't have hidden prompts or behind LLM calls. LLM Flows ensures complete transparency for each component, making monitoring, maintenance, and debugging easy. 




## 📦 Installation
```
pip install llmflows
```

## 🧭 Philosophy

### **Simple**
Our goal is to build a simple, well-documented framework with minimal abstractions that 
allow users to build flexible LLM-powered apps without compromising on capabilities.

### **Explicit**
We want to create an explicit API enabling users to write clean and readable code while 
easily creating complex flows of LLMs interacting with each other. LLMFlows' classes 
give users full control and do not have any hidden prompts or LLM calls. 

### **Transparent**
We aim to help users have full transparency on their LLM-powered apps by providing 
traceable flows and complete information for each app component, making it easy to 
monitor, maintain, and debug.

## 🧪 Getting Started
### LLMs
LLMs are one of the main abstractions in LLMFlows. LLM classes are wrappers around LLM 
APIs such as OpenAI's APIs. They provide methods for configuring and calling these APIs, 
retrying failed calls, and formatting the responses.

```python
from llmflows.llms import OpenAI

llm = OpenAI(api_key="<your-openai-api-key>")

result, call_data, model_config = llm.generate(
   prompt="Generate a cool title for an 80s rock song"
)
```


### PromptTemplates
The `PromptTemplate` class allows us to create strings with variables that we can fill 
in dynamically later on. Once a prompt template object is created an actual prompt can 
be generated by providing the required variables.

```python
from llmflows.llms import OpenAI
from llmflows.prompts import PromptTemplate


prompt_template = PromptTemplate(
    prompt="Generate a title for a 90s hip-hop song about {topic}."
)
llm_prompt = prompt_template.get_prompt(topic="friendship")

print(llm_prompt)

llm = OpenAI(api_key="<your-openai-api-key>")
song_title = llm.generate(llm_prompt)

print(song_title)
```

### Chat LLMs
Unlike regular LLMs that only require a prompt to generate text, chat LLMs require a 
conversation history. The conversation history is represented 
as a list of messages between a user and an assistant. This conversation history is 
sent to the model, and a new message is generated based on it.

LLMFlows provides a `MessageHistory` class to manage the required conversation history 
for chat LLMs.

You can build a simple chatbot by using the `OpenAIChat` and `MessageHistory` classes:

```python
from llmflows.llms import OpenAIChat, MessageHistory

llm = OpenAIChat(api_key="<your-openai-api-key>")
message_history = MessageHistory()

while True:
    user_message = input("You:")
    message_history.add_user_message(user_message)

    llm_response, call_data, model_config = llm.generate(message_history)
    message_history.add_ai_message(llm_response)

    print(f"LLM: {llm_response}")
```

### LLM Flows
Often times, real-world applications can be more complex and can have dependencies 
between prompts and LLM calls. For example:

![Complex flow](https://github.com/stoyan-stoyanov/llmflows/blob/main/docs/complex_flow.png)

When you want to build apps with complex dependencies you can use the `Flow` and 
`Flowstep` classes. LLMFlows will figure out the dependencies and make sure each 
flowstep runs only when all its dependencies are met:

```python
from llmflows.flows import Flow, FlowStep
from llmflows.llms import OpenAI
from llmflows.prompts import PromptTemplate

openai_llm = OpenAI(api_key="<your-openai-api-key>")

# Create prompt templates
title_template = PromptTemplate("What is a good title of a movie about {topic}?")
song_template = PromptTemplate(
    "What is a good song title of a soundtrack for a movie called {movie_title}?"
)
characters_template = PromptTemplate(
    "What are two main characters for a movie called {movie_title}?"
)
lyrics_template = PromptTemplate(
    "Write lyrics of a movie song called {song_title}. The main characters are "
    "{main_characters}"
)

# Create flowsteps
movie_title_flowstep = FlowStep(
    name="Movie Title Flowstep",
    llm=openai_llm,
    prompt_template=title_template,
    output_key="movie_title",
)

song_title_flowstep = FlowStep(
    name="Song Title Flowstep",
    llm=openai_llm,
    prompt_template=song_template,
    output_key="song_title",
)

characters_flowstep = FlowStep(
    name="Characters Flowstep",
    llm=openai_llm,
    prompt_template=characters_template,
    output_key="main_characters",
)

song_lyrics_flowstep = FlowStep(
    name="Song Lyrics Flowstep",
    llm=openai_llm,
    prompt_template=lyrics_template,
    output_key="song_lyrics",
)

# Connect flowsteps
movie_title_flowstep.connect(song_title_flowstep, characters_flowstep, song_lyrics_flowstep)
song_title_flowstep.connect(song_lyrics_flowstep)
characters_flowstep.connect(song_lyrics_flowstep)

# Create and run Flow
soundtrack_flow = Flow(movie_title_flowstep)
results = soundtrack_flow.start(topic="friendship", verbose=True)
```

### Async Flows
Sometimes multiple flow steps can run in parallel if all their dependencies are met. 
For cases like this, LLMFlows provides async classes to improve the runtime of any 
complex flow by running flow steps that already have all their required inputs in 
parallel.

```python

...

movie_title_flowstep = AsyncFlowStep(
    name="Movie Title Flowstep",
    llm=openai_llm,
    prompt_template=title_template,
    output_key="movie_title",
)

song_title_flowstep = FlowStep(
    name="Song Title Flowstep",
    llm=openai_llm,
    prompt_template=song_template,
    output_key="song_title",
)

characters_flowstep = AsyncFlowStep(
    name="Characters Flowstep",
    llm=openai_llm,
    prompt_template=characters_template,
    output_key="main_characters",
)

song_lyrics_flowstep = AsyncFlowStep(
    name="Song Lyrics Flowstep",
    llm=openai_llm,
    prompt_template=lyrics_template,
    output_key="song_lyrics",
)

...

```

For more examples, such as integrating vector databases, creating question-answering 
apps, agents, and web applications visit our 
[documentation](https://llmflows.readthedocs.io/en/latest/) or check out the 
[examples folder](https://github.com/stoyan-stoyanov/llmflows/tree/main/examples) of the repository.

## 🛠️ Features

### **LLMs**
- Utilize LLMs such as OpenAI's ChatGPT to generate natural language text.
- Configure LLM classes easily, choosing specific models, parameters, and settings.
- Benefit from automatic retries when model calls fail, ensuring reliable LLM 
  interactions.

### **Prompt Templates**
- Create dynamic prompts using Prompt Templates, providing flexible and customizable 
  text generation.
- Define variables within prompts to generate prompt strings tailored to specific 
  inputs.

### **Flows and FlowSteps**
- Structure LLM applications using Flows and FlowSteps, providing a clear and organized framework for executing LLM interactions.
- Connect flow steps to pass outputs as inputs, facilitating seamless data flow and
    maintaining a transparent LLM pipeline.
- Leverage Async Flows to run LLMs in parallel when all their inputs are available, 
  optimizing performance and efficiency.
- Incorporate custom string manipulation functions directly into flows, allowing 
  specialized text transformations without relying solely on LLM calls.

### **VectorStore Integrations**
- Integrate with vector databases like Pinecone using the VectorStoreFlowStep, 
  empowering efficient and scalable storage and retrieval of vector embeddings.
- Leverage vector databases for seamless storage and querying of vectors, enabling straightforward integration with LLM-powered applications.

### **Callbacks**
- Execute callback functions at different stages within flow steps, enabling enhanced customization, logging, tracing, or other specific integrations.
- Utilize callbacks to comprehensively control and monitor LLM-powered apps, ensuring 
  clear visibility into the execution process.

### **Explicit API and Full Transparency**
With LLMFlows you have the full control to create explicit applications without any hidden prompts or predefined behaviors.

In addition LLMFlows allows you to answer questions such as:

- When was a particular flowstep run?
- How much time did it take?
- What were the input variables?
- What was the prompt template?
- What did the prompt look like?
- What was the exact configuration of the model?
- How many times did we retry the request?
- What was the raw data the API returned?
- How many tokens were used?
- What was the final result?

## 📚 User Guide
[Introduction](https://llmflows.readthedocs.io/en/latest/user_guide/Introduction/)<br/>
[LLM Flows](https://llmflows.readthedocs.io/en/latest/user_guide/LLM%20Flows/)<br/>
[Async Flows](https://llmflows.readthedocs.io/en/latest/user_guide/Async%20Flows/)<br/>
[Vector Stores](https://llmflows.readthedocs.io/en/latest/user_guide/Vector%20Stores/)<br/>
[Question Answering](https://llmflows.readthedocs.io/en/latest/user_guide/Question%20Answering/)<br/>
[Vector Stores in Flows](https://llmflows.readthedocs.io/en/latest/user_guide/Vector%20Stores%20in%20Flows/)<br/>
[Functional FlowSteps](https://llmflows.readthedocs.io/en/latest/user_guide/Functional%20FlowSteps/)<br/>
[Callbacks](https://llmflows.readthedocs.io/en/latest/user_guide/Callbacks/)<br/>
[LLM Flows with FastAPI](https://llmflows.readthedocs.io/en/latest/user_guide/LLMFlows%20with%20FastAPI/)<br/>
[Agents](https://llmflows.readthedocs.io/en/latest/user_guide/Agents/)<br/>

## ▶️ Examples
You can find all examples used in the User Guide in the [examples folder](https://github.com/stoyan-stoyanov/llmflows/tree/main/examples) of the repository.

## 📃 License
LLMFlows is covered by the MIT license. For more information, check [`LICENCE`](https://github.com/stoyan-stoyanov/llmflows/blob/main/LICENSE).

## ❤️ How you can help?
Thank you for spending time going over our README! 

If you like the project please consider giving it a star, sharing it with friends or on social media.

If you've tried LLMFlows and have some issues, feedback or ideas feel free to open an issue or reach out!

If you find LLMFlows exciting and you are considering contributing, please check [`CONTRIBUTING.md`](https://github.com/stoyan-stoyanov/llmflows/blob/main/CONTRIBUTING.md).

## ✉️ Contact
If you want to reach out please don't hesitate to connect on the following social media:

[Threads](https://www.threads.net/@sptstoyanov)<br/>
[LinkedIn](https://www.linkedin.com/in/spstoyanov/)<br/>
[Twitter](https://twitter.com/stoyanpstoyanov)<br/>

I would love to hear from you!







