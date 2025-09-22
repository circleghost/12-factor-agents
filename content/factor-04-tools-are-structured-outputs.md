[← 回到 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 4. 工具只是結構化輸出

工具不需要複雜。在核心上，它們只是來自你的 LLM 的結構化輸出，觸發確定性程式碼。

![140-tools-are-just-structured-outputs](https://github.com/humanlayer/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)

例如，假設你有兩個工具 `CreateIssue` 和 `SearchIssues`。要求 LLM「使用幾個工具中的一個」就是要求它輸出我們可以解析為代表這些工具的物件的 JSON。

```python

class Issue:
  title: str
  description: str
  team_id: str
  assignee_id: str

class CreateIssue:
  intent: "create_issue"
  issue: Issue

class SearchIssues:
  intent: "search_issues"
  query: str
  what_youre_looking_for: str
```

模式很簡單：
1. LLM 輸出結構化 JSON
3. 確定性程式碼執行適當的動作 (例如呼叫外部 API)
4. 結果被捕獲並回饋到上下文中

這在 LLM 的決策制定和你的應用程式動作之間建立了清晰的分離。LLM 決定要做什麼，但你的程式碼控制如何完成。僅僅因為 LLM「呼叫了一個工具」並不意味著你必須每次都以相同的方式執行特定的對應函數。

如果你回想我們上面的 switch 語句

```python
if nextStep.intent == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return # 或者你想要的任何操作，見下文
elif nextStep.intent == 'wait_for_a_while': 
    # 做一些 monadic 的事情
else: # ... 模型沒有呼叫我們知道的工具
    # 做其他事情
```

**注意**：關於「純粹提示」vs.「工具呼叫」vs.「JSON 模式」的好處以及各自的性能權衡已經有很多討論。我們很快會連結一些相關資源，但這裡不會深入討論。參見 [Prompting vs JSON Mode vs Function Calling vs Constrained Generation vs SAP](https://www.boundaryml.com/blog/schema-aligned-parsing)、[何時應該使用函數呼叫、結構化輸出或 JSON 模式？](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode#:~:text=We%20don%27t%20recommend%20using%20JSON,always%20use%20Structured%20Outputs%20instead) 和 [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)。

「下一步」可能不會像「運行純函數並返回結果」那樣原子化。當你將「工具呼叫」視為只是模型輸出描述確定性程式碼應該做什麼的 JSON 時，你會解鎖很多靈活性。將這與[要素 8 擁有你的控制流程](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)結合起來。

[← 擁有你的上下文視窗](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) | [統一執行狀態 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
