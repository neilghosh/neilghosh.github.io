---
layout: post
title: Run your own ChatGPT
date: 2025-12-28
description: Learn how to run large language models locally on your computer using Ollama, including Gemma, Qwen, and other models, with web UI and CLI options for privacy and cost-free AI assistance.
categories:
  - LLM
---

# Run your own ChatGPT

Well the title is a bit click bait-y. It basically means running Large Language Models locally in your computer, even in VM for that matter. There are two major advantages of it.
- It's free!! You don't have to pay for the subscriptions like ChatGPT or Gemini Pro.
- Though there are free versions of above commercial LLMs, running in your own computer makes it more private, you are not giving away your personal queries, interest and data (say you are asking health related questions)

Firstly you have to install and start `ollama`. https://ollama.com/download. On Mac you can just do

```
brew install ollama
```

Next we have to run the server. 

```
ollama serve
```

Now we have to decide which model to download, I choose to run a very large model from Gemini as I have a very powerful machine, a Mac Studio with M2 Ultra Chip.

So in another terminal we run the following. For the first time depending on the size of the model it takes some time to download the model and then gives you the prompt.

```
ollama run gemma3:27b
```
```
>>> Send a message (/? for help)
```

Now you can chat with the model.

```
>>> Who are you? Whats todays date
Hi there! I'm Gemma, an open-weights AI assistant. I'm a large language model trained by Google DeepMind. The Gemma team are my creators.

Today is November 21, 2023.
```

The date would be much in the past because it was probably trained around that time. And knowledge it would have is not beyond this date.

```
>>> What's your knowledge cut off and Parameters?
My knowledge cut-off is **September 2021**. This means I don't have information about events that occurred after that date. So, I won't know about things that happened in 2022, 2023, or beyond unless specifically included in my training data.

As for my parameters, I come in two sizes:

*   **Gemma 2B:** Has **2 billion parameters**. This is a smaller, faster model suitable for resource-constrained environments.
*   **Gemma 7B:** Has **7 billion parameters**. This is a larger, more capable model that generally delivers better performance.

I'm the open-weights version, meaning the model weights are publicly available! You can find more details on the Gemma models here: [https://ai.google.dev/gemma](https://ai.google.dev/gemma)
```

Let's test the latest knowledge.

```
>>> What's an MCP server
An MCP server refers to a server used for playing **Minecraft: Pocket Edition (MCPE)** using a third-party server software called **Multiplayer for Pocket Edition (MCPC)**, or sometimes just "MCP." Here's a breakdown of what it is and why it's different from
regular Minecraft servers:
```

See, it does not even know that MCP server has now taken over the AI space and hence the internet.

Anyway it's still very powerful as an LLM and can be used either in this terminal mode or you can choose to use `open-webui` to get a ChatGPT/Gemini style chat interface in browser. https://openwebui.com/

For that you need to install docker desktop or there are python ways to run it.

If you have docker desktop.

```
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

Once the docker container runs, just point to http://localhost:3000/

![Open WebUI Interface](/assets/2025/ollama-open-webui.png)

You could also use the API. 

```
curl http://localhost:11434/api/generate -d '{  "model": "gemma3:27b",   "prompt":"Who was first President of India"}'
```

```
neilghosh@neilghosh-mac-studio ~ % curl http://localhost:11434/api/generate -d '{  "model": "gemma3:27b",   "prompt":"Who was first President of India"}'
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:47.941022Z","response":"The","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:47.980486Z","response":" first","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.019804Z","response":" President","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.059296Z","response":" of","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.098868Z","response":" India","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.138344Z","response":" was","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.177787Z","response":" **","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.217355Z","response":"Dr","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.25691Z","response":".","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.296326Z","response":" Raj","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.335835Z","response":"endra","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:48.375877Z","response":" Prasad","done":false}
...
...
Indian","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:50.230835Z","response":" Constitution","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:50.27012Z","response":".","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:50.309673Z","response":"\n\n\n\n","done":false}
{"model":"gemma3:27b","created_at":"2025-12-28T11:43:50.34898Z","response":"","done":true,"done_reason":"stop","context":[105,2364,107,15938,691,1171,5376,529,4673,106,107,105,4368,107,818,1171,5376,529,4673,691,5213,9231,236761,15105,37888,91520,84750,236743,108,2209,7779,699,236743,236770,236819,236810,236771,531,236743,236770,236819,236825,236778,236764,1646,16492,10911,236761,1293,691,496,2307,8575,528,506,6225,21555,7296,1680,4673,16531,21555,236764,532,6657,496,3629,3853,528,66582,506,6225,20901,236761,110],"total_duration":2903714459,"load_duration":156688500,"prompt_eval_count":15,"prompt_eval_duration":337553125,"eval_count":62,"eval_duration":2382672126}
```

The responses are in small bits and pieces because of the very fundamental nature of LLMs. It generates and streams back to the client one token at a time. It's not a fancy way of displaying one word after another what someone may think by looking at UIs like ChatGPT.

While Gemma2:27b is a powerful LLM, there are other LLMs one can try out. Like for Coding tasks `qwen2.5-coder:32b` is great.

```
ollama run qwen2.5-coder:32b
```

To use this coding model in a Chat window in VS Code just like copilot chat or any other coding assistant, you can use the VS Code plugin `continue` and configure various models (running via ollama).

```
name: Local Config
version: 1.0.0
schema: v1
models:
  - name: Qwen3 30B
    provider: ollama
    model: qwen3:30b
    roles:
      - chat
      - edit
      - apply      
  - name: qwen2.5-coder:32b
    provider: ollama
    model: qwen2.5-coder:32b
    roles:
      - autocomplete
```

It will use the corresponding model for the action mentioned in the config 

![Continue Chat in VS Code](/assets/2025/vscode-continue-chat.png)

If you see it asking API keys of subscription based APIs, you may have to double check if local config is enabled for continue plugin.

![Continue Plugin Configuration](/assets/2025/vscode-continue-config.png)

For CLI fans coding, there is a continue cli as well which behaves like copilot-cli or gemini-cli.

```
npm install -g @continuedev/cli
```

![Continue CLI in Terminal](/assets/2025/continue-cli-terminal.png)

You can at any time list the models that are downloaded and running currently. If a model is not actively used, it does not really show up in `ollama ps`

```
 ~ % ollama list
NAME                 ID              SIZE     MODIFIED
deepseek-r1:32b      edba8017331d    19 GB    45 hours ago
qwen3:30b            ad815644918f    18 GB    2 days ago
qwen2.5-coder:32b    b92d6a0bd47e    19 GB    2 days ago
gemma3:27b           a418f5838eaf    17 GB    2 days ago
```


