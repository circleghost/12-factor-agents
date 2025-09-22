[← 回到 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 1. 自然語言到工具呼叫

在 Agent 建構中最常見的模式之一，就是將自然語言轉換為結構化的工具呼叫。這是一個強大的模式，讓你能夠建構可以對任務進行推理和執行的 Agent。

![110-natural-language-tool-calls](https://github.com/humanlayer/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)

這個模式在原子化應用時，就是將這樣的語句進行簡單轉換：

> 你能為 Terri 贊助二月 AI 愛好者聚會建立一個 $750 的付款連結嗎？

轉換為描述 Stripe API 呼叫的結構化物件：

```json
{
  "function": {
    "name": "create_payment_link",
    "parameters": {
      "amount": 750,
      "customer": "cust_128934ddasf9",
      "product": "prod_8675309",
      "price": "prc_09874329fds",
      "quantity": 1,
      "memo": "Hey Jeff - see below for the payment link for the february ai tinkerers meetup"
    }
  }
}
```

**注意**：實際上 Stripe API 要更複雜一些，一個[真正做這件事的 Agent](https://github.com/dexhorthy/mailcrew) ([影片](https://www.youtube.com/watch?v=f_cKnoPC_Oo)) 會列出客戶、列出產品、列出價格等，以使用正確的 ID 建構這個有效載荷，或者在提示/上下文視窗中包含這些 ID (我們稍後會看到這些其實是同一件事！)

從這裡開始，確定性程式碼可以接收有效載荷並對其進行處理。(更多內容請參見[要素 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md))

```python
# LLM 接收自然語言並返回結構化物件
nextStep = await llm.determineNextStep(
  """
  為 Jeff 建立一個 $750 的付款連結
  用於贊助二月 AI 愛好者聚會
  """
  )

# 根據函數處理結構化輸出
if nextStep.function == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return  # 或者你想要的任何操作，見下文
elif nextStep.function == 'something_else':
    # ... 更多情況
    pass
else:  # 模型沒有呼叫我們知道的工具
    # 做其他事情
    pass
```

**注意**：雖然完整的 Agent 會接收 API 呼叫結果並循環處理，最終返回類似這樣的內容：

> 我已經成功為 Terri 贊助二月 AI 愛好者聚會建立了一個 $750 的付款連結。這是連結：https://buy.stripe.com/test_1234567890

**但是**，我們實際上會跳過這個步驟，把它留給另一個要素，你可以選擇是否也要包含這個步驟 (由你決定！)

[← 我們如何到達這裡](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software.md) | [擁有你的提示 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
