### 要素 13 - 預先獲取你可能需要的所有上下文

如果你的模型很可能會呼叫工具 X，就不要浪費 token 來回告訴模型去獲取它，也就是說，不要像這樣的偽提示：

```jinja
在查看部署時，你可能會想要獲取已發布的 git 標籤列表，
這樣你就可以用它來部署到生產環境。

迴今為止發生的事情：

{{ thread.events }}

下一步是什麼？

用 JSON 格式回答，使用以下意圖之一：

{
  intent: 'deploy_backend_to_prod',
  tag: string
} OR {
  intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}
```

而你的程式碼看起來像這樣

```python
thread = {"events": [initial_message]}
next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'list_git_tags':
      tags = await fetch_git_tags()
      thread["events"].append({
        type: 'list_git_tags',
        data: tags,
      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

你不如直接獲取標籤並將它們包含在上下文視窗中，像這樣：

```diff
- 在查看部署時，你可能會想要獲取已發布的 git 標籤列表，
- 這樣你就可以用它來部署到生產環境。

+ 目前的 git 標籤是：

+ {{ git_tags }}


迴今為止發生的事情：

{{ thread.events }}

下一步是什麼？

用 JSON 格式回答，使用以下意圖之一：

{
  intent: 'deploy_backend_to_prod',
  tag: string
- } OR {
-   intent: 'list_git_tags'
} OR {
  intent: 'done_for_now',
  message: string
}

```

而你的程式碼看起來像這樣

```diff
thread = {"events": [initial_message]}
+ git_tags = await fetch_git_tags()

- next_step = await determine_next_step(thread)
+ next_step = await determine_next_step(thread, git_tags)

while True:
  switch next_step.intent:
-    case 'list_git_tags':
-      tags = await fetch_git_tags()
-      thread["events"].append({
-        type: 'list_git_tags',
-        data: tags,
-      })
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append({
        "type": 'deploy_backend_to_prod',
        "data": deploy_result,
      })
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

或者更簡單地將標籤包含在執緒中，並從你的提示模板中移除特定參數：

```diff
thread = {"events": [initial_message]}
+ # 新增請求
+ thread["events"].append({
+  "type": 'list_git_tags',
+ })

git_tags = await fetch_git_tags()

+ # 新增結果
+ thread["events"].append({
+  "type": 'list_git_tags_result',
+  "data": git_tags,
+ })

- next_step = await determine_next_step(thread, git_tags)
+ next_step = await determine_next_step(thread)

while True:
  switch next_step.intent:
    case 'deploy_backend_to_prod':
      deploy_result = await deploy_backend_to_prod(next_step.data.tag)
      thread["events"].append(deploy_result)
    case 'done_for_now':
      await notify_human(next_step.message)
      break
    # ...
```

總的來說：

> #### 如果你已經知道你希望模型呼叫哪些工具，就直接確定性地呼叫它們，讓模型做困難的部分：弄清楚如何使用它們的輸出

再次強調，AI 工程完全關乎[上下文工程](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)。

[← 無狀態化簡器](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md) | [延伸閱讀 →](https://github.com/humanlayer/12-factor-agents/blob/main/README.md#related-resources)
