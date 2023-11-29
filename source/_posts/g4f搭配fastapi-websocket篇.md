---
title: g4f搭配fastapi-websocket篇
date: 2023-11-29 15:07:34
tags: [Websocket, Python]
categories:
    - Python
---

### websocket 在fastapi中的使用
```python
import g4f
from fastapi import APIRouter, Body, WebSocket, WebSocketDisconnect, WebSocketException
from asyncio import sleep as async_sleep

g4f.debug.logging = True
router = APIRouter()


@router.websocket("/ws-chat")
async def ws_chat(
	websocket: WebSocket,
):
	await websocket.accept()
	try:
		while True:
			json_input = await websocket.receive_json()
			event_type = json_input.get("event_type", "")
			if event_type == "ping":
				await websocket.send_text("pong")

			if event_type == "chat":
				model = json_input.get("model", "gpt-3.5-turbo")
				messages = json_input.get("messages", [])
				response = g4f.Provider.FreeGpt.create_completion(
					model=model,
					messages=messages,
					stream=True,
				)
				for message in response:
					await websocket.send_json({"text": message, "finish_reason": ""})
				await websocket.send_json({"text": "", "finish_reason": "finish"})
			await async_sleep(0.1)

	except WebSocketException: ...
	except WebSocketDisconnect: ...
	except Exception as e:
		print(e)

```
## 调用结果
![postman调用结果](01.png)