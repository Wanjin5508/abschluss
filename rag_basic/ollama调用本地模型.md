在本地运行 Ollama 后（执行 `ollama run gemma:4b`），可以通过 HTTP 调用该模型：

```http
POST http://localhost:11434/api/generate
```


发送请求体如下：
```json
{
  "model": "gemma:4b",
  "prompt": "你是谁？",
  "stream": false
}
```

返回的是 JSON 格式的结果。

