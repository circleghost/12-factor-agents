[← 回到 README](https://github.com/circleghost/12-factor-agents/blob/main/README.md)

### 8. 擁有你的控制流程

如果你擁有你的控制流程，你可以做很多有趣的事情。

![180-control-flow](https://github.com/circleghost/12-factor-agents/blob/main/img/180-control-flow.png)


建立適合你特定使用案例的自己的控制結構。具體來說，某些類型的工具呼叫可能是跳出迴圈並等待人類回應或另一個長時間運行任務 (如訓練管道) 的原因。你可能還想要整合自訂實作：

- 工具呼叫結果的摘要或快取
- 結構化輸出上的 LLM 作為評判者
- 上下文視窗壓縮或其他[記憶體管理](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- 記錄、追蹤和指標
- 客戶端限速
- 持久睡眠/暫停/「等待事件」


下面的範例顯示了三種可能的控制流程模式：


- request_clarification：模型要求更多資訊，跳出迴圈並等待人類回應
- fetch_git_tags：模型要求 git 標籤列表，擷取標籤，附加到上下文視窗，並直接傳回給模型
- deploy_backend：模型要求部署後端，這是高風險的事情，所以跳出迴圈並等待人類批准

```python
def handle_next_step(thread: Thread):

  while True:
    next_step = await determine_next_step(thread_to_prompt(thread))
    
    # 為了清晰而內聯 - 實際上你可以將此放在方法中，
    # 使用例外來控制流程，或任何你想要的方式
    if next_step.intent == 'request_clarification':
      thread.events.append({
        type: 'request_clarification',
          data: nextStep,
        })

      await send_message_to_human(next_step)
      await db.save_thread(thread)
      # 非同步步驟 - 跳出迴圈，稍後我們會收到 webhook
      break
    elif next_step.intent == 'fetch_open_issues':
      thread.events.append({
        type: 'fetch_open_issues',
        data: next_step,
      })

      issues = await linear_client.issues()

      thread.events.append({
        type: 'fetch_open_issues_result',
        data: issues,
      })
      # 同步步驟 - 將新上下文傳遞給 LLM 以確定下一個步驟
      continue
    elif next_step.intent == 'create_issue':
      thread.events.append({
        type: 'create_issue',
        data: next_step,
      })

      await request_human_approval(next_step)
      await db.save_thread(thread)
      # 非同步步驟 - 跳出迴圈，稍後我們會收到 webhook
      break
```

這種模式允許你根據需要中斷和恢復 Agent 的流程，創造更自然的對話和工作流程。

**範例** - 我對所有 AI 框架的首要功能要求是我們需要能夠中斷正在工作的 Agent 並稍後恢復，特別是在工具**選擇**時刻和工具**呼叫**時刻之間。

如果沒有這種層級的可恢復性/粒度，就無法在工具呼叫執行前審查/批准它，這意味著你被迫：

1. 在等待長時間運行的任務完成時將任務暫停在記憶體中 (想想 `while...sleep`)，如果程序中斷則從頭開始重新啟動
2. 將 Agent 限制為僅進行低風險、低賭注的呼叫，如研究和摘要
3. 給 Agent 存取權限去做更大、更有用的事情，然後只是隨機希望它不會搞砸


你可能會注意到這與[要素 5 - 統一執行狀態與業務狀態](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)和[要素 6 - 使用簡單 API 進行啟動/暫停/恢復](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)密切相關，但可以獨立實作。

[← 透過工具聯繫人類](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) | [壓縮錯誤 →](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
