# 提速代码实例（Python / JS）

> 对应 SKILL.md 中各类提速场景的可运行片段。
> 替换 `API_URL` / `API_KEY` / `MODEL` 即可使用。流式依赖服务端支持（如 OpenAI 的 `stream=True`）。

---

## A. 流式输出（Streaming）—— 改善单条体感

### Python（OpenAI 兼容）
```python
from openai import OpenAI
client = OpenAI(base_url="https://api.example.com/v1", api_key="API_KEY")

stream = client.chat.completions.create(
    model="MODEL",
    messages=[{"role": "user", "content": "讲讲并发和流式的区别"}],
    stream=True,            # 关键：逐 token 返回
)
for chunk in stream:
    delta = chunk.choices[0].delta.content or ""
    print(delta, end="", flush=True)   # 边生成边显示
```

### JavaScript（fetch 流式）
```javascript
const res = await fetch("https://api.example.com/v1/chat/completions", {
  method: "POST",
  headers: { "Content-Type": "application/json", Authorization: "Bearer API_KEY" },
  body: JSON.stringify({ model: "MODEL", stream: true,
    messages: [{ role: "user", content: "讲讲并发和流式的区别" }] }),
});
const reader = res.body.getReader();
const dec = new TextDecoder();
while (true) {
  const { value, done } = await reader.read();
  if (done) break;
  process.stdout.write(dec.decode(value));   // 逐块渲染
}
```

---

## B. 异步 + 并发（asyncio）—— 独立任务并行

```python
import asyncio, aiohttp

async def call_one(session, prompt):
    async with session.post("https://api.example.com/v1/chat/completions",
        json={"model": "MODEL", "messages": [{"role": "user", "content": prompt}]}) as r:
        data = await r.json()
        return data["choices"][0]["message"]["content"]

async def main(prompts):
    # 连接池：TCPConnector 复用连接；同一 session 内请求自动复用
    connector = aiohttp.TCPConnector(limit=10)        # 并发度 / 池大小
    timeout = aiohttp.ClientTimeout(total=30)
    async with aiohttp.ClientSession(connector=connector, timeout=timeout) as session:
        tasks = [call_one(session, p) for p in prompts]
        return await asyncio.gather(*tasks)           # 并发执行，互不等待

results = asyncio.run(main(["翻译A", "翻译B", "翻译C"]))   # 仅限彼此独立的任务
```

---

## C. 连接池（同步）—— 复用热连接

```python
import requests

# Session 复用 TCP 连接（连接池），避免每次请求重新握手
session = requests.Session()
session.mount("https://", requests.adapters.HTTPAdapter(
    pool_connections=10, pool_maxsize=10))

for prompt in prompts:
    r = session.post("https://api.example.com/v1/chat/completions",
                     json={"model": "MODEL", "messages": [{"role": "user", "content": prompt}]},
                     timeout=30)
    print(r.json())
```

---

## 选型对照

| 场景 | 用哪段 |
|---|---|
| 单条回复慢、想更快看到内容 | A 流式 |
| 多个独立任务、想一起跑完 | B 异步并发（连接池内置） |
| 高频调同一接口、想省握手 | C 连接池 |
| 单条慢且是模型生成 | A 流式；能拆则转 B 并发（对不可拆单条 B 无效） |
