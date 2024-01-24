# Creating Your First Agent
<!--
In this tutorial, we will explore the `ChatAgent` class. The topics covered include:

1.
2.
3.
4.  -->

## Introduction

The `ChatAgent()` class is a cornerstone of CAMEL 🐫. We will first start with the single agent setting, where the agent can interact with users, process and store messages, and utilize external tools to generate responses and accomplish tasks.

## Quick Start
Let's first play with a `ChatAgent` instance by simply initialize it with a system message and interact with user messages.

0. Import basic classes.
    ```python
    from camel.messages import BaseMessage as bm
    from camel.agents import ChatAgent
    ```

1. Create a system message, which defines its default role and behaviors.
    ```python
    sys_msg = bm.make_assistant_message(
        role_name='system',
        content='you are a helpful assistant.')
    ```
2. Initialize the agent as simply as providing the system message.
    ```python
    agent = ChatAgent(
        system_message=sys_msg,
        message_window_size=10,    # [Optional] Num. of prior messages to store
        )
    ```
3. Now you can interact with the agent using the `.step()` method.
    ```python
    # Define a user message
    usr_msg = bm.make_user_message(
        role_name='claude shannon',
        content='dear, what is information in your mind?')

    # Sending the message to the agent
    response = agent.step(usr_msg)

    # Check the response (just for illustrative purpose)
    print(response.msgs[0].content)
    >>> information is the resolution of uncertainty.
    ```

## [Optional] Advanced Features

### Some Useful Methods
Setting the agent to its initial state:
```python
agent.reset()
```
Set the output language for the agent. (Notice that this method is achieved by appending language choice instruction to the system message; you may reinforce through additional messages for better intruction following.)
```python
agent.set_output_language('french')
```
The `ChatAgent` class also offers several advanced initialization options, including `model_type`, `model_config`, `memory`, `message_window_size`, `token_limit`, `output_language`, `function_list`, and `response_terminators`. Check [`chat_agent.py`](https://github.com/camel-ai/camel/blob/master/camel/agents/chat_agent.py) for detailed usage guidance.

### Function Calling
```python
# Import the necessary functions
from camel.functions import MATH_FUNCS, SEARCH_FUNCS

# Initialize the agent with list of functions
agent = ChatAgent(
    system_message=sys_msg,        
    function_list=[*MATH_FUNCS, *SEARCH_FUNCS]
    )

# Check if functions are enabled
agent.is_function_calling_enabled()
>>> True
```

### Playing with the Agent's Memory
By default our agent is initialized with `ChatHistoryMemory`. Assume that you have followed the setup in [Quick Start](#quick-start). Let's first check what is inside its brain.
```python
agent.memory.get_context()
>>> ([{'role': 'system', 'content': 'you are a helpful assistant.'},
      {'role': 'user', 'content': 'dear, what is information in your mind?'}],
      30)
```
By default, we store only the user messages. You may update the agent's memory with any externally provided message in the format of `BaseMessage`; for example, using the agent's own response:
```python
# Update the memory
agent.record_message(response.msgs[0])
```
```python
# Check the current memory
agent.memory.get_context()
>>> ([{'role': 'system', 'content': 'you are a helpful assistant.'},
      {'role': 'user', 'content': 'dear, what is information in your mind?'}],
      {'role': 'assistant', 'content': 'information is the resolution of uncertainty.'}
      44)
```



### Using Open-Source Models
(This section echos our instruction in the setup chapter. We put it here for completeness of the content.)

The high-level idea is to deploy a server with the local model in the backend and use it as a local drop-in replacement for the API. We here use [FastChat](https://github.com/lm-sys/FastChat/blob/main/docs/openai_api.md) as an example.

0. Install the FastChat package with the following command, or see [here](https://github.com/lm-sys/FastChat/tree/main#install) for other options.
    ```bash
    pip3 install "fschat[model_worker,webui]"
    ```

1. Starting the FastChat server in the backend.
    ```python
    # Launch the fastchat controller
    python -m fastchat.serve.controller

    # Launch the model worker
    python -m fastchat.serve.model_worker \
        --model-path meta-llama/Llama-2-7b-chat-hf  # a local folder or HuggingFace repo Name

    # Launch the API server
    python -m fastchat.serve.openai_api_server \
        --host localhost \
        --port 8000
    ```


2. Initialize the agent.
    ```python
    # Import the necessary classes
    from camel.configs import ChatGPTConfig, OpenSourceConfig
    from camel.types import ModelType

    # Set the arguments
    agent_kwargs = dict(
        model_type=ModelType.LLAMA_2,                    # Specify the model type

        model_config=OpenSourceConfig(
            model_path='meta-llama/Llama-2-7b-chat-hf',  # a local folder or HuggingFace repo Name
            server_url='http://localhost:8000/v1',       # The url with the set port number
        ),

        token_limit=2046,                                # [Optional] Choose the ideal limit
    )

    # Now your agent is ready to play
    agent = ChatAgent(sys_msg, **agent_kwargs)
    ```

### Remarks
Awesome. Now you have made your first step in creating a single agent 🐫. In the next chapter, we will explore the creation of different types agents along with the role playing features. Stay tuned 🦖🐆🐘🦒🦘🦕!
