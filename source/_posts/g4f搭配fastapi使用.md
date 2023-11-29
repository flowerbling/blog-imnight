---
title: g4f搭配fastapi使用
date: 2023-11-29 14:07:21
tags: [AIGC, Python, Stream]
categories:
    - Python
---

## G4f 介绍
> [g4f(GPT4Free)](https://github.com/xtekky/gpt4free) GPT-4 目前是世界上最强的多模态大模型，能力甩 GPT-3.5 好几条街。大家都希望早日用上 GPT-4，不过目前体验 GPT-4 的渠道非常有限，要么就是开通 ChatGPT 尊贵的 Plus 会员，即使你开了会员，也是有限制的，每 3 小时只能发送 25 条消息。。。要么就去 OpenAI 官网申请 GPT-4 的 API，但是目前申请到 API 的小伙伴非常少，即使申请到 API, GPT-4 的 API 价格超级无敌贵，是 GPT-3.5 价格的 30 倍，你敢用吗？
`GPT4Free` 上线几周就在 GitHub 上揽收了接近 4w 的 Star。原因就在于其提供了对 GPT-4 及 GPT-3.5 免费且几乎无限制的访问。该项目通过对各种调用了 OpenAI API 网站的第三方 API 进行逆向工程，达到使任何人都可以免费访问该流行 AI 模型的目的。
```bash 安装g4f
pip install -U g4f
```

## sse_starlette 介绍
> sse_starlette 是一个基于 Starlette 框架的服务器端事件（Server-Sent Events）库，用于在 Python 中实现服务器端推送数据给客户端的功能。Server-Sent Events 是一种基于 HTTP 的单向实时通信协议，允许服务器向客户端发送持续的事件流。

```bash 安装sse_starlette
pip install sse-starlette
```

## fastapi 中使用 sse_starlette
```python fastapi 使用 sse_starlette发送数据
import g4f
from fastapi import APIRouter, Body
from sse_starlette.sse import EventSourceResponse

g4f.debug.logging = True

router = APIRouter()


@router.post("/chat")
def chat(
	request: dict = Body(...),
):
	def event_generator():
		model = request.get("model", "gpt-3.5-turbo")
		messages = request.get("messages", [])
		response = g4f.Provider.FreeGpt.create_completion(
			model=model,
			messages=messages,
			stream=True,
		)
		for message in response:
			yield message

	return EventSourceResponse(event_generator())
```

## 调用结果

![postman调用结果](01.png)