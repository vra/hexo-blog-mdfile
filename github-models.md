---
title: GitHub Models-免费的大模型Playgroud和API服务
date: 2024-09-14 08:50:04
tags:
 - LLM
 - AI
 - GitHub
 - GitHub Models
---

### 1. 功能说明
GitHub在2024年8月10号左右的时候推出了GitHub Models新功能，提供运行大模型的Playground和免费API服务，用于进行AI大模型的实验和AI应用的原型验证。目前已经支持的模型包括GPT-4o系列，phi-3系列，Llama-3系列，以及一些Embedding模型等（OpenAI o1-mini和o1-preview虽然列出来了，但需要登陆Azure来使用）。

![](/imgs/github-models/20240914083033.png)
<!--more-->

### 2. 申请waitlist
GitHub Models功能还在limited public beta阶段，需要先申请加入[waitlist](https://github.com/marketplace/models/waitlist/join)，通过后才能体验。

本来以为跟之前Copilot，Codespace等功能一样，国内无法申请或者申请通过后无法使用，但这次却没有卡这些条件，我从8月13号提交申请，9月11号通过，目前测试国内网络也可以使用免费的API服务，因为服务都是搭建在Azure云服务上面的。

### 3. 请求限制

GitHub 定位是给开发者开发AI应用原型提供免费的服务（某种程度上也是给Azure引流），所以有请求限制，具体来说，大模型限制级别分为Low和High，Low级别每分钟最多请求15次，每天上限是150，每次请求的最大输入token是8000，最大输出token数是4000，最大并发请求5个，High级别每分钟最多请求10次，每天上限是50，每次请求的最大输入token是8000，最大输出token数是4000，最大并发请求2个，所以这种quota，可能真的就够自己做原型调试用了。Embedding模型有单独的级别，具体数据见下表：

![](/imgs/github-models/20240914083717.png)

### 4. 使用流程
下面简单介绍一下使用的流程。

GitHub Models的网址是<https://github.com/marketplace/models>,除了开始图片展示的，还包含下面这些模型：
![](/imgs/github-models/20240914084921.png)

选择一个模型后，进入到详情页面，有模型的介绍，还有Web上直接使用的Playground选项，以及API调用的 Get started选项，以及请求限制级别：
![](/imgs/github-models/20240914085054.png)

点击Playground进入Web使用页面，看起来跟OpenAI网站很像，可以直接聊天，也可以调整右边的参数进行控制，同时除了Chat，还是Code和Raw模式：
![](/imgs/github-models/20240914085230.png)
Chat 模式下，直接进行提问，返回结果，还可以点赞点踩，重新提问：
![](/imgs/github-models/20240914085442.png)
Code模式下，会给出在Python代码中调用接口的示例：
![](/imgs/github-models/20240914085629.png)
Raw模式下，会以JSON格式显示用户的问题，模型的回答：
![](/imgs/github-models/20240914085721.png)

Raw模式和Chat模式都可以进行对话，JSON内容会实时更新：
![](/imgs/github-models/20240914085935.png)

点Get Started按钮后，会显示API调用的详细说明：
![](/imgs/github-models/20240914090039.png)
像这个模型，支持Python, JS， C#和REST四种形式的调用（有些模型只支持Python和JS）,
SDK可以选择OpenAI SDK（pip install openai）或者Azure AI Inference SDK(pip install  azure-ai-inference)，右边给出了详细的使用说明
![](/imgs/github-models/20240914090137.png)
### 5. API调用
首先需要在[GitHub 这里](https://github.com/settings/tokens)生成TOKEN，这个TOKEN跟OpenAI Key一样，用于模型调用的鉴权等等。

#### 5.1 使用OpenAI SDK

将上面GITHUB_TOKEN加入环境变量，然后就是熟悉的调用方式了，下面将单次对话，多次对话，流式输出，传入图片和调用工具的示例代码放上来，供参考
##### 5.1.1 单次对话
```python
import os
from openai import OpenAI

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

client = OpenAI(
    base_url=endpoint,
    api_key=token,
)

response = client.chat.completions.create(
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant.",
        },
        {
            "role": "user",
            "content": "What is the capital of France?",
        }
    ],
    model=model_name,
    temperature=1.0,
    max_tokens=1000,
    top_p=1.0
)

print(response.choices[0].message.content)
```
##### 5.1.2 多轮对话
```python
import os
from openai import OpenAI

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

client = OpenAI(
    base_url=endpoint,
    api_key=token,
)

response = client.chat.completions.create(
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant.",
        },
        {
            "role": "user",
            "content": "What is the capital of France?",
        },
        {
            "role": "assistant",
            "content": "The capital of France is Paris.",
        },
        {
            "role": "user",
            "content": "What about Spain?",
        }
    ],
    model=model_name,
)

print(response.choices[0].message.content)
```

##### 5.1.3 流式输出
```python
import os
from openai import OpenAI

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

client = OpenAI(
    base_url=endpoint,
    api_key=token,
)

response = client.chat.completions.create(
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant.",
        },
        {
            "role": "user",
            "content": "Give me 5 good reasons why I should exercise every day.",
        }
    ],
    model=model_name,
    stream=True
)

for update in response:
    if update.choices[0].delta.content:
        print(update.choices[0].delta.content, end="")
```
##### 5.1.4 图片输入
```python
import os
import base64
from openai import OpenAI

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

def get_image_data_url(image_file: str, image_format: str) -> str:
    """
    Helper function to converts an image file to a data URL string.

    Args:
        image_file (str): The path to the image file.
        image_format (str): The format of the image file.

    Returns:
        str: The data URL of the image.
    """
    try:
        with open(image_file, "rb") as f:
            image_data = base64.b64encode(f.read()).decode("utf-8")
    except FileNotFoundError:
        print(f"Could not read '{image_file}'.")
        exit()
    return f"data:image/{image_format};base64,{image_data}"


client = OpenAI(
    base_url=endpoint,
    api_key=token,
)

response = client.chat.completions.create(
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant that describes images in details.",
        },
        {
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": "What's in this image?",
                },
                {
                    "type": "image_url",
                    "image_url": {
                        "url": get_image_data_url("sample.jpg", "jpg"),
                        "detail": "low"
                    },
                },
            ],
        },
    ],
    model=model_name,
)

print(response.choices[0].message.content)
```

##### 5.1.5 工具调用
```python
import os
import json
from openai import OpenAI

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

# Define a function that returns flight information between two cities (mock implementation)
def get_flight_info(origin_city: str, destination_city: str):
    if origin_city == "Seattle" and destination_city == "Miami":
        return json.dumps({
            "airline": "Delta",
            "flight_number": "DL123",
            "flight_date": "May 7th, 2024",
            "flight_time": "10:00AM"})
    return json.dumps({"error": "No flights found between the cities"})

# Define a function tool that the model can ask to invoke in order to retrieve flight information
tool={
    "type": "function",
    "function": {
        "name": "get_flight_info",
        "description": """Returns information about the next flight between two cities.
            This includes the name of the airline, flight number and the date and time
            of the next flight""",
        "parameters": {
            "type": "object",
            "properties": {
                "origin_city": {
                    "type": "string",
                    "description": "The name of the city where the flight originates",
                },
                "destination_city": {
                    "type": "string", 
                    "description": "The flight destination city",
                },
            },
            "required": [
                "origin_city",
                "destination_city"
            ],
        },
    },
}

client = OpenAI(
    base_url=endpoint,
    api_key=token,
)

messages=[
    {"role": "system", "content": "You an assistant that helps users find flight information."},
    {"role": "user", "content": "I'm interested in going to Miami. What is the next flight there from Seattle?"},
]

response = client.chat.completions.create(
    messages=messages,
    tools=[tool],
    model=model_name,
)

# We expect the model to ask for a tool call
if response.choices[0].finish_reason == "tool_calls":

    # Append the model response to the chat history
    messages.append(response.choices[0].message)

    # We expect a single tool call
    if response.choices[0].message.tool_calls and len(response.choices[0].message.tool_calls) == 1:

        tool_call = response.choices[0].message.tool_calls[0]

        # We expect the tool to be a function call
        if tool_call.type == "function":

            # Parse the function call arguments and call the function
            function_args = json.loads(tool_call.function.arguments.replace("'", '"'))
            print(f"Calling function `{tool_call.function.name}` with arguments {function_args}")
            callable_func = locals()[tool_call.function.name]
            function_return = callable_func(**function_args)
            print(f"Function returned = {function_return}")

            # Append the function call result fo the chat history
            messages.append(
                {
                    "tool_call_id": tool_call.id,
                    "role": "tool",
                    "name": tool_call.function.name,
                    "content": function_return,
                }
            )

            # Get another response from the model
            response = client.chat.completions.create(
                messages=messages,
                tools=[tool],
                model=model_name,
            )

            print(f"Model response = {response.choices[0].message.content}")
```
#### 5.2 使用Azure AI Inference SDK
整体上与使用OpenAI SDK类似，有些函数接口有变化

##### 5.2.1 单次推理
```python
import os
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"
token = os.environ["GITHUB_TOKEN"]

client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(token),
)

response = client.complete(
    messages=[
        SystemMessage(content="You are a helpful assistant."),
        UserMessage(content="What is the capital of France?"),
    ],
    model=model_name,
    temperature=1.0,
    max_tokens=1000,
    top_p=1.0
)

print(response.choices[0].message.content)
```

##### 5.2.2 多轮推理
```python
import os
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import AssistantMessage, SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(token),
)

messages = [
    SystemMessage(content="You are a helpful assistant."),
    UserMessage(content="What is the capital of France?"),
    AssistantMessage(content="The capital of France is Paris."),
    UserMessage(content="What about Spain?"),
]

response = client.complete(messages=messages, model=model_name)

print(response.choices[0].message.content)
```
##### 5.2.3 流式输出
```python
import os
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import SystemMessage, UserMessage
from azure.core.credentials import AzureKeyCredential

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(token),
)

response = client.complete(
    stream=True,
    messages=[
        SystemMessage(content="You are a helpful assistant."),
        UserMessage(content="Give me 5 good reasons why I should exercise every day."),
    ],
    model=model_name,
)

for update in response:
    if update.choices:
        print(update.choices[0].delta.content or "", end="")

client.close()
```
##### 5.2.4 调用图片
```python
import os
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import (
    SystemMessage,
    UserMessage,
    TextContentItem,
    ImageContentItem,
    ImageUrl,
    ImageDetailLevel,
)
from azure.core.credentials import AzureKeyCredential

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(token),
)

response = client.complete(
    messages=[
        SystemMessage(
            content="You are a helpful assistant that describes images in details."
        ),
        UserMessage(
            content=[
                TextContentItem(text="What's in this image?"),
                ImageContentItem(
                    image_url=ImageUrl.load(
                        image_file="sample.jpg",
                        image_format="jpg",
                        detail=ImageDetailLevel.LOW)
                ),
            ],
        ),
    ],
    model=model_name,
)

print(response.choices[0].message.content)
```
##### 5.2.5 使用工具
```python
import os
import json
from azure.ai.inference import ChatCompletionsClient
from azure.ai.inference.models import (
    AssistantMessage,
    ChatCompletionsToolCall,
    ChatCompletionsToolDefinition,
    CompletionsFinishReason,
    FunctionDefinition,
    SystemMessage,
    ToolMessage,
    UserMessage,
)
from azure.core.credentials import AzureKeyCredential

token = os.environ["GITHUB_TOKEN"]
endpoint = "https://models.inference.ai.azure.com"
model_name = "gpt-4o-mini"

# Define a function that returns flight information between two cities (mock implementation)
def get_flight_info(origin_city: str, destination_city: str):
    if origin_city == "Seattle" and destination_city == "Miami":
        return json.dumps({
            "airline": "Delta",
            "flight_number": "DL123",
            "flight_date": "May 7th, 2024",
            "flight_time": "10:00AM"})
    return json.dumps({"error": "No flights found between the cities"})

# Define a function tool that the model can ask to invoke in order to retrieve flight information
flight_info = ChatCompletionsToolDefinition(
    function=FunctionDefinition(
        name="get_flight_info",
        description="""Returns information about the next flight between two cities.
            This includes the name of the airline, flight number and the date and
            time of the next flight""",
        parameters={
            "type": "object",
            "properties": {
                "origin_city": {
                    "type": "string",
                    "description": "The name of the city where the flight originates",
                },
                "destination_city": {
                    "type": "string",
                    "description": "The flight destination city",
                },
            },
            "required": ["origin_city", "destination_city"],
        },
    )
)

client = ChatCompletionsClient(
    endpoint=endpoint,
    credential=AzureKeyCredential(token),
)

messages = [
    SystemMessage(content="You an assistant that helps users find flight information."),
    UserMessage(content="I'm interested in going to Miami. What is the next flight there from Seattle?"),
]

response = client.complete(
    messages=messages,
    tools=[flight_info],
    model=model_name,
)

# We expect the model to ask for a tool call
if response.choices[0].finish_reason == CompletionsFinishReason.TOOL_CALLS:

    # Append the model response to the chat history
    messages.append(AssistantMessage(tool_calls=response.choices[0].message.tool_calls))

    # We expect a single tool call
    if response.choices[0].message.tool_calls and len(response.choices[0].message.tool_calls) == 1:

        tool_call = response.choices[0].message.tool_calls[0]

        # We expect the tool to be a function call
        if isinstance(tool_call, ChatCompletionsToolCall):

            # Parse the function call arguments and call the function
            function_args = json.loads(tool_call.function.arguments.replace("'", '"'))
            print(f"Calling function `{tool_call.function.name}` with arguments {function_args}")
            callable_func = locals()[tool_call.function.name]
            function_return = callable_func(**function_args)
            print(f"Function returned = {function_return}")

            # Append the function call result fo the chat history
            messages.append(ToolMessage(tool_call_id=tool_call.id, content=function_return))

            # Get another response from the model
            response = client.complete(
                messages=messages,
                tools=[flight_info],
                model=model_name,
            )

            print(f"Model response = {response.choices[0].message.content}")
```

### 6. 总结
GitHub Models总体上来说还是一个有用的工具，有下面的优点：
1. 免费
1. 服务部署在Azure云服务器，国内网络可访问
2. 有GPT-4o系列模型和对应API，对于没有OpenAI账号的开发者可以基于这里的API开发应用
3. 设计良好的SDK，支持Python, JS, C#和REST等形式

当然缺点也有：
1. 访问次数有上限，输入输出token有限制
2. 模型并不多，目前只有30个模型，像Claude就没有

希望这篇文章能让你对GitHub Models这个功能有更清晰的认识，欢迎点赞，收藏和评论！

