[← 回到 README](https://github.com/circleghost/12-factor-agents/blob/main/README.md)

### 9. 將錯誤壓縮到上下文視窗中

這個要素雖然簡短，但值得一提。Agent 的一個好處是「自我修復」——對於短任務，LLM 可能會呼叫一個失敗的工具。優秀的 LLM 有相當好的機會讀取錯誤訊息或堆疊追蹤，並找出在後續工具呼叫中需要更改的內容。

大多數框架都實現了這一點，但您可以僅僅實現這一點，而不必執行其他 11 個要素。以下是一個例子： 


```python
thread = {"events": [initial_message]}

while True:
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread["events"].append({
    "type": next_step.intent,
    "data": next_step,
  })
  try:
    result = await handle_next_step(thread, next_step) # our switch statement
  except Exception as e:
    # if we get an error, we can add it to the context window and try again
    thread["events"].append({
      "type": 'error',
      "data": format_error(e),
    })
    # loop, or do whatever else here to try to recover
```

您可能想要為特定工具呼叫實現一個 errorCounter，將單一工具的嘗試次數限制在大約 3 次，或者其他對您的使用情境有意義的邏輯。 

```python
consecutive_errors = 0

while True:

  # ... existing code ...

  try:
    result = await handle_next_step(thread, next_step)
    thread["events"].append({
      "type": next_step.intent + '_result',
      data: result,
    })
    # success! reset the error counter
    consecutive_errors = 0
  except Exception as e:
    consecutive_errors += 1
    if consecutive_errors < 3:
      # do the loop and try again
      thread["events"].append({
        "type": 'error',
        "data": format_error(e),
      })
    else:
      # break the loop, reset parts of the context window, escalate to a human, or whatever else you want to do
      break
  }
}
```
達到某些連續錯誤閾值可能是[升級給人類](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)的好地方，無論是透過模型決策還是透過控制流程的確定性接管。

[![195-factor-09-errors](https://github.com/circleghost/12-factor-agents/blob/main/img/195-factor-09-errors.gif)](https://github.com/user-attachments/assets/cd7ed814-8309-4baf-81a5-9502f91d4043)


<details>
<summary>[GIF 版本](https://github.com/circleghost/12-factor-agents/blob/main/img/195-factor-09-errors.gif)</summary>

![195-factor-09-errors](https://github.com/circleghost/12-factor-agents/blob/main/img/195-factor-09-errors.gif)

</details>

好處：

1. **自我修復**：LLM 可以讀取錯誤訊息並找出在後續工具呼叫中需要更改的內容
2. **持久性**：即使一個工具呼叫失敗，Agent 仍然可以繼續執行

我確信您會發現，如果您過度使用這種方法，您的 Agent 將開始失控，並可能一遍又一遍地重複同樣的錯誤。

這就是[要素 8 - 掌控您的控制流程](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)和[要素 3 - 掌控您的上下文建構](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)發揮作用的地方——您不需要只是將原始錯誤放回去，您可以完全重新構建它的表示方式，從上下文視窗中移除先前的事件，或者您發現有助於讓 Agent 重回正軌的任何確定性方法。

但防止錯誤失控的首要方法是擁抱[要素 10 - 小型、專注的 Agent](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)。

[← 掌控您的控制流程](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) | [小型專注的 Agent →](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
