---
title: 重温Go
date: 2023-12-06 11:55:29
tags: [Golang, 编程]
categories:
    - Golang
---

## 重温Go
> 一眨眼已经将近一年不写Golang的项目了 今天又接触Golang
### 克隆项目
github 权限开通之后 克隆项目到本地
```bash
git clone git@xxxxx
cd xxxx
code .

go mod tidy # 先看一下依赖 go.mod go 1.18  看了下本地 go version go1.18.3 darwin/amd64 不会有问题
go run main.go # 没什么问题 开始撸代码
```

### 项目结构
看了下 项目是gin框架的 项目结构如下 简单的一批 加个API就好了
```markdown
- api
    - index.go
- tts
    - ali.go
- utils
    - queue.go
- go.mod
- go.sum
- main.go
- README.md
- s.yaml
```

### 新加API
需求是用Go封装一个API 调用openai的API 生成文本后 steam流的方式返回文本的tts
1. 定义输入的结构
```go
type Message struct {
	Role    string `json:"role"`
	Content string `json:"content"`
}

type Request struct {
	Messages []Message `json:"messages"`
	Model    string  `json:"model,omitempty"`
	Speed    float64 `json:"speed,omitempty"`
	Voice    string  `json:"voice,omitempty"`
	MaxTokens int     `json:"max_tokens,omitempty"`
}
```

2. 实现openai的调用和tts的调用
```go
func replyWithTTS(ctx *gin.Context, clientChan ClientChan) {
	var request Request
	err := ctx.BindJSON(&request)
	if request.Model == "" {
		request.Model = openai.GPT3Dot5Turbo
	}
	if request.MaxTokens == 0 {
		request.MaxTokens = 1024
	}
	if request.Speed == 0 {
		request.Speed = 1.0
	}
	if request.Voice == "" {
		request.Voice = string(openai.VoiceAlloy)
	}
	if err != nil {
		sendMessage(clientChan, "error", gin.H{"error": err.Error()})
		sendMessageToChan(clientChan, "")
		return
	}

	c := getOpenAiClient()

	messages := []openai.ChatCompletionMessage{}
	for _, m := range request.Messages {
		messages = append(messages, openai.ChatCompletionMessage{
			Role: m.Role,
			Content: m.Content,
		})
	}

	req := openai.ChatCompletionRequest{
		Model:     request.Model,
		MaxTokens: request.MaxTokens,
		Messages: messages,
		Stream: true,
	}
	stream, err := c.CreateChatCompletionStream(ctx, req)
	if err != nil {
		sendMessage(clientChan, "error", gin.H{"error": fmt.Sprintf("ChatCompletionStream error: %v\n", err)})
		sendMessageToChan(clientChan, "")
		return
	}
	defer stream.Close()

	sentence := ""
	queue := utils.NewQueue()
	worker := func(item interface{}) {
		responseTTS(clientChan, item.(string), request.Speed, request.Voice)
	}
	queue.StartProcessing(worker)

	for {
		response, err := stream.Recv()
		if errors.Is(err, io.EOF) {
			if len(sentence) > 0 {
				queue.Enqueue(sentence)
			}
			queue.CloseQueue()
			sendMessage(clientChan, "stop", gin.H{"message": "Stream finished"})
			sendMessageToChan(clientChan, "")
			return
		}

		if err != nil {
			queue.CloseQueue()
			sendMessage(clientChan, "error", gin.H{"error": fmt.Sprintf("\nStream error: %v\n", err)})
			sendMessageToChan(clientChan, "")
			return
		}

		delta := response.Choices[0].Delta.Content
		sentence += delta

		sentence = strings.ReplaceAll(sentence, "\n\n", "\n")
		if sentence == "" {
			continue
		}

		reg := regexp.MustCompile(`[\p{L}\p{N}\s]+[。？；……\.?!;]*`)
		// 使用正则表达式的FindAllStringIndex方法来获得所有匹配的索引
		matches := reg.FindAllStringIndex(sentence, -1)
		sentences := []string{}
		// 遍历索引并使用它来提取并输出句子
		for _, match := range matches {
			sentences = append(sentences, sentence[match[0]:match[1]])
		}
		leftSentence := ""
		for i := 0; i < len(sentences)-1; i++ {
			s := leftSentence + sentences[i]
			if len(s) < 4 {
				leftSentence = s
				continue
			}
			leftSentence = ""
			queue.Enqueue(s)
		}
		sentence = leftSentence + sentences[len(sentences)-1]
	}
}

func responseTTS(clientChan ClientChan, sentence string, speed float64, voice string) {
	c := getOpenAiClient()
	body, err := c.CreateSpeech(context.Background(), openai.CreateSpeechRequest{
		Model: openai.TTSModel1,
		Voice: openai.SpeechVoice(voice),
		Input: sentence,
		Speed: speed,
	})
	if err != nil {
		sendMessage(clientChan, "error", gin.H{"error": err})
		return
	}
	bytes, _ := io.ReadAll(body)
	base64Text := base64.StdEncoding.EncodeToString(bytes)

	sendMessage(clientChan, "processing", gin.H{"message": sentence, "voice": base64Text})
}
```

3. stream请求和返回
```go
	...
	router.POST("/api", HeadersMiddleware(), stream.serveHTTP(), func(c *gin.Context) {
		v, ok := c.Get("clientChan")
		if !ok {
			return
		}
		clientChan, ok := v.(ClientChan)
		if !ok {
			return
		}
		go answer(c, clientChan)

		c.Stream(func(w io.Writer) bool {
			// Stream message to client from message channel
			if msg, ok := <-clientChan; ok {
				if msg == "" { // 退出信号
					return false
				}
				c.SSEvent("message", msg)
				return true
			}
			return false
		})
	})
	...
```