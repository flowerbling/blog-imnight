---
title: python中使用和修复markdown
date: 2023-12-06 10:38:11
tags: [Python, Markdown]
categories:
    - Python
---

## Python 中使用和修复Markdown

> 要在Python中格式化Markdown文本 第三方库来处理Markdown语法

### markdown转html

#### markdown

- 安装markdown

    ```bash
    pip install markdown
    ```

- Markdown 文本转 html

    ```python
    from markdown import Markdown
    
    title = "My Markdown Document"
    content = "This is a **bold** sentence."
    md_text = f"# {title}\n\n{content}"
    
    md = Markdown()
    
    html = md.convert(md_text)
    print(html)
    
    # 输出
    # <h1>My Markdown Document</h1>
    # <p>This is a <strong>bold</strong> sentence.</p>
    
    ```



#### mistune

- 安装mistune

    ```bash
    pip install mistune
    ```

- Markdown 文本转 html

    ```python
    import mistune
    
    title = "My Markdown Document"
    content = "This is a **bold** sentence."
    md_text = f"# {title}\n\n{content}"
    
    html = mistune.markdown(md_text)
    
    print(html)
    
    # 输出
    # <h1>My Markdown Document</h1>
    # <p>This is a <strong>bold</strong> sentence.</p>
    ```

```markdown
`adawd `
```

`adawd`

--dawdaw--

--awd--

~~---awdaw~~

```
***
```

