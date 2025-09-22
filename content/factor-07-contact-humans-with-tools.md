[← 回到 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 7. 透過工具呼叫聯繫人類

預設情況下，LLM API 依賴於一個基本的高風險 token 選擇：我們是回傳純文字內容，還是回傳結構化資料？

![170-contact-humans-with-tools](https://github.com/humanlayer/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)

你在第一個 token 的選擇上投入了很多權重，在 `the weather in tokyo` 案例中，它是

> "the"

但在 `fetch_weather` 案例中，它是表示 JSON 物件開始的特殊 token。

> |JSON>

你可能會透過讓 LLM *始終*輸出 JSON，然後用一些自然語言 token 如 `request_human_input` 或 `done_for_now` 來宣告其意圖 (而不是像 `check_weather_in_city` 這樣的「正規」工具) 而獲得更好的結果。

再次強調，你可能不會從這種方法中獲得任何效能提升，但你應該進行實驗，並確保你可以自由嘗試奇怪的做法以獲得最佳結果。

```python

class Options:
  urgency: Literal["low", "medium", "high"]
  format: Literal["free_text", "yes_no", "multiple_choice"]
  choices: List[str]

# 人機互動的工具定義
class RequestHumanInput:
  intent: "request_human_input"
  question: str
  context: str
  options: Options

# 在 Agent 迴圈中的使用範例
if nextStep.intent == 'request_human_input':
  thread.events.append({
    type: 'human_input_requested',
    data: nextStep
  })
  thread_id = await save_state(thread)
  await notify_human(nextStep, thread_id)
  return # 中斷迴圈並等待回應與執行緒 ID 一起回來
else:
  # ... 其他情況
```

稍後，你可能會從處理 Slack、電子郵件、簡訊或其他事件的系統收到 webhook。

```python

@app.post('/webhook')
def webhook(req: Request):
  thread_id = req.body.threadId
  thread = await load_state(thread_id)
  thread.events.push({
    type: 'response_from_human',
    data: req.body
  })
  # ... 為了簡潔而簡化，你可能不想在這裡阻塞 Web worker
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread.events.append(next_step)
  result = await handle_next_step(thread, next_step)
  # todo - 迴圈或中斷或你想要的任何操作

  return {"status": "ok"}
```

上述包含來自[要素 5 - 統一執行狀態與業務狀態](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)、[要素 8 - 擁有你的控制流程](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)、[要素 3 - 擁有你的上下文視窗](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)、[要素 4 - 工具就是結構化輸出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) 以及其他幾個要素的模式。

如果我們使用來自[要素 3 - 擁有你的上下文視窗](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)的 XML 格式，我們的上下文視窗在幾輪對話後可能看起來像這樣：

```xml

(為簡潔起見省略)

<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy backend v1.2.3 to production?
    Thread: []
</slack_message>

<request_human_input>
    intent: "request_human_input"
    question: "Would you like to proceed with deploying v1.2.3 to production?"
    context: "This is a production deployment that will affect live users."
    options: {
        urgency: "high"
        format: "yes_no"
    }
</request_human_input>

<human_response>
    response: "yes please proceed"
    approved: true
    timestamp: "2024-03-15T10:30:00Z"
    user: "alex@company.com"
</human_response>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<deploy_backend_result>
    status: "success"
    message: "Deployment v1.2.3 to production completed successfully."
    timestamp: "2024-03-15T10:30:00Z"
</deploy_backend_result>
```


好處：

1. **清晰指示**：用於不同類型人機聯繫的工具允許 LLM 更具體化
2. **內部與外部迴圈**：啟用傳統 ChatGPT 風格介面**之外**的 Agent 工作流程，其中控制流程和上下文初始化可能是 `Agent->Human` 而不是 `Human->Agent` (想想由 cron 或事件啟動的 Agent)
3. **多重人員存取**：可以透過結構化事件輕鬆追蹤和協調來自不同人員的輸入
4. **多 Agent**：簡單的抽象可以輕鬆擴展以支援 `Agent->Agent` 請求和回應
5. **持久性**：結合[要素 6 - 使用簡單 API 進行啟動/暫停/恢復](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)，這使得持久、可靠且可檢查的多人工作流程成為可能

[更多關於外部迴圈 Agent 的資訊在這裡](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)

![175-outer-loop-agents](https://github.com/humanlayer/12-factor-agents/blob/main/img/175-outer-loop-agents.png)

與[要素 11 - 從任何地方觸發，在使用者所在的地方與他們會面](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)搭配使用效果很好

[← 啟動/暫停/恢復](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) | [擁有你的控制流程 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
