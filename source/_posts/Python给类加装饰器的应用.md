---
title: Python给类加装饰器的应用
date: 2023-12-21 16:16:31
tags: [Python, 装饰器]
categories:
    - Python
---
> 一个语言如果支持将函数作为参数传递，那么它就可以使用装饰器。
> 昨天刚看到这句话，今天就恰好需要用装饰器来实现一个功能了。

## 需求
现在需要记录llm每次运行时消耗的token数量。
```python
class Assistant(BaseModel):
    model: str = Field(default="gpt-3.6-turbo")
    temperature: float = Field(default=0)
    top_p: float = Field(default=1)
    n: int = Field(default=1)
    max_tokens: int = Field(default=4095)
    presence_penalty: float = Field(default=0)
    frequency_penalty: float = Field(default=0)
    system_prompt: str = Field(default="")
    messages: list[dict[str, str]] = Field(default_factory=list[dict[str, str]])
    response_format: str = Field(default="text")
    format_prompt: str = Field(default="")

    @retry(stop=stop_after_attempt(3), wait=wait_random(min=1, max=5))
    def run(self, callbacks: Any = None, **kwargs) -> Any:
        ...

    @retry(stop=stop_after_attempt(3), wait=wait_random(min=1, max=5))
    async def arun(self, callbacks: Any = None, **kwargs) -> Any:
        ...

    @retry(stop=stop_after_attempt(3), wait=wait_random(min=1, max=5))
    def stream(self, queue: Optional[asyncio.Queue], callbacks: Any = None, **kwargs) -> Any:
        ...
    @retry(stop=stop_after_attempt(3), wait=wait_random(min=1, max=5))
    async def astream(self, queue: Optional[asyncio.Queue], callbacks: Any = None, **kwargs) -> Any:
        ...

```
看到这里的时候 就想到了装饰器 不我需要再关心每个函数内部的实现， 不需要给每个函数都去加一段计算和记录token的逻辑
只需要一个装饰器 就可以将每次执行的token数量记录下来

## 实现装饰器
```python
def token_usage_track(**kwargs):
    def decorator(cls):
        for method_name, method in cls.__dict__.items():
            funcs = kwargs.get("funcs", [])
            if method_name in funcs and callable(method):
                setattr(cls, method_name, llm_runner(method))
        return cls

    def llm_runner(method):
        def wrapper(self, *args, **kwargs):
            with get_openai_token_usage() as cb:
                result, run_id = method(self, *args, **kwargs)
                self.token_usages[run_id] = {
                    model_name: usage.dict() for model_name, usage in cb.model_token_usage.items()
                }
            return result, run_id

        return wrapper
    return decorator
```
实现了装饰器之后发现 这个装饰器只能处理同步函数 不能处理异步函数。
所以需要进一步完善来处理所有函数
```python
def token_usage_track(**kwargs):
    def decorator(cls):
        for method_name, method in cls.__dict__.items():
            funcs = kwargs.get("funcs", [])
            async_funcs = kwargs.get("async_funcs", [])
            if method_name in funcs and callable(method):
                setattr(cls, method_name, llm_runner(method))
            if method_name in async_funcs and callable(method):
                setattr(cls, method_name, async_llm_runner(method))
        return cls

    def llm_runner(method):
        def wrapper(self, *args, **kwargs):
            with get_openai_token_usage() as cb:
                result, run_id = method(self, *args, **kwargs)
                self.token_usages[run_id] = {
                    model_name: usage.dict() for model_name, usage in cb.model_token_usage.items()
                }
            return result, run_id

        return wrapper

    def async_llm_runner(method):
        async def wrapper(self, *args, **kwargs):
            with get_openai_token_usage() as cb:
                result, run_id = await method(self, *args, **kwargs)
                self.token_usages[run_id] = {
                    model_name: usage.dict() for model_name, usage in cb.model_token_usage.items()
                }
            return result, run_id

        return wrapper

    return decorator
```

## 测试装饰器
```python
llm_setting = get_default_slide_assistant()
assistant = Assistant.parse_obj(llm_setting)
result, run_id = assistant.stream(content=USER, queue=None)
print(assistant.token_usages)
# {UUID('f061f53f-544a-4cba-ae5b-7386fea0c309'): {'gpt-4-1106-preview': {'model_name': 'gpt-4-1106-preview', 'total_tokens': 43, 'prompt_tokens': 18, 'completion_tokens': 25, 'successful_requests': 1, 'total_cost': 0.00093}}}
```

看langchain的代码， 每个调用也都是需要实现 同步和异步两个版本的，差点就忘记了！！！