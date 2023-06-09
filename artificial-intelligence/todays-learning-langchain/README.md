---
description: LangChain
---

# ⚙ Today's learning--LangChain

{% hint style="info" %}
[https://python.langchain.com/en/latest/getting\_started/getting\_started.html](https://python.langchain.com/en/latest/getting\_started/getting\_started.html)
{% endhint %}

```python

# set the appropriate environment variable
import os
os.environ["SERPAPI_API_KEY"] = "..."
from langchain.agents import load_tools
from langchain.agents import initialize_agent
from langchain.agents import AgentType
from langchain.agents import OpenAI

# First, let's load the language model we're going to use to control the agent.
llm = OpenAI(temperature=0)

# Next, let's load some tools to use. Note that the 'llm-math' tool uses an LLM, so we need to pass that in.
tools = load_tools(["serpapi", "llm-math"],llm=llm)

# Finally, let's initialize the agent with the tools, the language model, and the type of agent we want to use.
agent = initialize_agent(tools,llm,agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True)

# Now let's test it out!
agent.run("What was the high temperature in SF yesterday? What is that number raised to the .023 power?")
```

## Memory: add state to chains and agents

So far, all the chains and agents we've gone through have been stateless. But often, you may want to chain or agent to have some concept of "memory" so that it may remember information about its previous interactions. The clearest and simple example of this is when **designing a chatbot-you want it to remember previous message** so it can use context from that **to have a better conversation.** This would be a type of "**short-term memory**". On <mark style="color:red;">the more complex side</mark>, you could imagine a chain/agent remembering _**key pieces of inofrmation**_ over time - this would be a form of "long-term memory".&#x20;

LangChain provides several specially created chains just for this purpose. This notebook walks through using one of chain (the ConversationChain) with two different types of memory.

By default, <mark style="color:blue;">the ConversationChain</mark> has a <mark style="color:orange;">simple type of memory</mark> that remembers all previous inputs/outputs and adds them to the context that is passed. Let's take a look at using this chain (setting verbose=True so we can see the prompt)

```python
from langchain import OpenAI, ConversationChain

llm = OpenAI(temperature=0)
conversation = ConversationChain(llm=llm, verbose=True)

output = conversation.predict(input="Hi there!")
print(output)
```

## Building a language Model Application: Chat Models

Similarly, you can use chat models instead of LLMs. Chat models are a variation on language models. While chat models use language models under the hood, the interface they expose is a bit different: rather than expose a "text in, txet out" API(Application Programming Interface), they expose an interface where "chat messages" are the inputs and outputs. Chat model APIs are fiarly new, so we are still figuring out the correct abstractions.

## Get Message Completions from a Chat Model

You can get completions by passing one or more message to the chat model. The response will be a message. The types of message currently supported in LangChain are **AIMessage**, **HumanMessage**, **SystemMessage,** and **ChatMessage**-**ChatMessage** takes in an arbitary role parameter. Most of the time, you'll just dealing withdealing with HumanMessage, AIMessage, and SystemMessage.

```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import(
    AIMessage,
    HumanMessage,
    SystemMessage
)
chat = ChatOpenAI(temperature=0)
```

You can get completions by passing in a single message.

```python
chat([HumanMessage(content="Translate this setence from English to French. I love programming.")])
# -> AIMessage(content="J'aime programmer.", additional_kwargs={})
```

You can also pass in multiple messages for OpenAI's gpt-3.5-<mark style="color:blue;">**turbo**</mark> and gpt-4 models.

```python
messages= [
    SystemMessage(content="You are a helpful assitant that translates English to French."),
    HumanMessage(content="I love programming.")
]
chat(message)
# -> AIMessage(content="J'aime programmer.", additional_kwargs={})
```

You can go one step further and generate completions for multiple sets of messages using <mark style="color:red;">**generate**</mark>. This returns an <mark style="color:red;">**LLMResult**</mark> with an additional message parameter:

```python
batch_message = [
    [
        SystemMessage(content="You are a helpful assistant that translates English to French."),
        HumanMessage(content="I love programming.")
    ],
    [
        SystemMessage(content="You are a helpful assistant that translates English to French."),
        HumanMessage(content="I love aitificial intelligence.")
    ],
]
result = chat.generate(batch_messages)
result
# ->LLMResult(generations=[[ChatGeneration(text="J'aime programmer.",
generation_info=None, message=AIMessage(content="J'aime programmer.',
additional_kwargs={}))], [ChatGeneration(text="J'aime l'intelligence artificielle.", generation_info=None, message=AIMessage(content="J'aime l'intelligence artificielle.", additional_kwargs={}))]], llm_output={'token_usage':
{'prompt_tokens': 57, 'completion_token': 20, 'total_token': 77}})
```



## Chat Prompt Template

Similar to LLMs, you can make use of templating by using a <mark style="color:red;">**MessagePromptTemplate**</mark>. You can build a <mark style="color:red;">**ChatPromptTemplate**</mark> from one or more <mark style="color:red;">**MessagePromptTemplate**</mark>s. You can use <mark style="color:red;">**ChatPromptTemplate**</mark>'s <mark style="color:red;">**format\_prompt**</mark>--this return a PromptValue, which you can use convert to a string or <mark style="color:red;">**Message**</mark> object, depending on whether you want to use the formatted value as input to an llm or chat model.

For convenience, there is a <mark style="color:red;">**form\_template**</mark> method exposed on the template. If you were to use this template, this is what it would look like:

```
from langchain.chat_models import ChatOpenAI
from langchain.prompt.chat import(
    ChatPromptTemplate,
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate,
)
chat = ChatOpenAI(temperature=0)

template = "You are a helpful assistant that translate {input_language} to {output_language}."
system_message_prompt = SystemMessagePropmtTemplate.from_template(template)
human_template = "{text}"
human_message_prompt = HumanMessage
```

## Chains with Chat Models

The <mark style="color:red;">**LLMChain**</mark> discussed in the above section can be used with chat models as well:

```python
from langchain.chat_models import ChatOpenAI
from langchain import LLMChain
from langchain.prompts.chat import(
    ChatPromptTemplate,
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate,
)
chat = ChatOpenAI(temperature=0)

template = "You are a helpful assistant that translates {input_language} to {output_language}."
system_message_prompt = SystemMessagePromptTemplate.from_template(template)
human_template = "{text}"
human_message_prompt = HumanMessagePromptTemplate.from_template(human_template)
chat_prompt = ChatPromptTemplate.from_messages([system_message_prompt
human_message_prompt])

chain = LLMChain(llm=chat, prompt=chat_prompt)
chain.run(input_language="Engilsh", output_language="French", text="I love programming.")
# -> "J'aime programmer."
```
