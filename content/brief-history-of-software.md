[← 回到 README](https://github.com/circleghost/12-factor-agents/blob/main/README.md)

## 完整版：我們是如何走到這裡的

### 你不必聽我的

無論你是 Agent 的新手還是像我這樣的老手，我都會試圖說服你拋棄大部分關於 AI Agent 的想法，退一步，從第一原理重新思考它們。(劇透警告：如果你沒有看到幾週前 OpenAI 回應功能的發佈，那麼將更多 Agent 邏輯推到 API 後面並不是正確方向)


## Agent 是軟體，以及軟體的簡史

讓我們談談我們是如何走到這裡的

### 60 年前

我們會大量討論有向圖 (Directed Graphs, DGs) 和它們的無環朋友 DAG。首先我要指出...軟體就是一個有向圖。這就是為什麼我們過去用流程圖來表示程式的原因。

![010-software-dag](https://github.com/circleghost/12-factor-agents/blob/main/img/010-software-dag.png)

### 20 年前

大約 20 年前，我們開始看到 DAG 編排器變得流行。我們談論的是經典工具如 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/)，一些前身工具，以及一些更新的工具如 ([dagster](https://dagster.io/)、[inngest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/))。這些都遵循相同的圖形模式，並增加了可觀察性、模組化、重試、管理等優勢。

![015-dag-orchestrators](https://github.com/circleghost/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### 10-15 年前

當機器學習模型開始變得足夠好用時，我們開始看到在 DAG 中融入機器學習模型。你可以想像像這樣的步驟：「將此欄位中的文字摘要到新欄位中」或「按嚴重程度或情感分類支援問題」。

![020-dags-with-ml](https://github.com/circleghost/12-factor-agents/blob/main/img/020-dags-with-ml.png)

但歸根結底，它仍然主要是同樣好的老式確定性軟體。

### Agent 的承諾

我不是第一個[這樣說的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但當我開始學習 Agent 時，我最大的收穫是你可以拋棄 DAG。軟體工程師不用再編碼每個步驟和邊緣情況，你可以給 Agent 一個目標和一組轉換：

![025-agent-dag](https://github.com/circleghost/12-factor-agents/blob/main/img/025-agent-dag.png)

讓 LLM 即時做決策來找出路徑

![026-agent-dag-lines](https://github.com/circleghost/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

這裡的承諾是你寫更少的軟體，你只需給 LLM 圖的「邊」，讓它找出節點。你可以從錯誤中恢復，寫更少的程式碼，而且你可能會發現 LLM 找到解決問題的新方法。

### Agent 作為循環

換句話說，你有這個由 3 個步驟組成的循環：

1. LLM 決定工作流程中的下一步，輸出結構化 JSON (「工具呼叫」)
2. 確定性程式碼執行工具呼叫
3. 結果被附加到上下文視窗
4. 重複直到下一步被確定為「完成」

```python
initial_event = {"message": "..."}
context = [initial_event]
while True:
  next_step = await llm.determine_next_step(context)
  context.append(next_step)

  if (next_step.intent === "done"):
    return next_step.final_answer

  result = await execute_step(next_step)
  context.append(result)
```

我們的初始上下文只是起始事件 (可能是用戶訊息、可能是 cron 觸發、可能是 webhook 等)，我們要求 LLM 選擇下一步 (工具) 或確定我們已經完成。

這是一個多步驟範例：

[![027-agent-loop-animation](https://github.com/circleghost/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/circleghost/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF Version</a></summary>

![027-agent-loop-animation](https://github.com/circleghost/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

而生成的「具象化」DAG 會看起來像這樣：

![027-agent-loop-dag](https://github.com/circleghost/12-factor-agents/blob/main/img/027-agent-loop-dag.png)

### 這種「循環直到解決」模式的問題

這種模式的最大問題：

- 當上下文視窗變得太長時，Agent 會迷失方向 - 它們會陷入一遍又一遍嘗試相同破損方法的困境
- 就是這樣，但這足以徹底削弱這種方法

即使你沒有手工構建 Agent，你可能也在使用 Agent 式編程工具時看到過這種長上下文問題。它們在一段時間後就會迷失方向，你需要開始新的對話。

我甚至可能會提出一些我經常聽到的觀點，而你可能也已經形成了自己的直覺：

> ### **即使模型支援越來越長的上下文視窗，你始終會通過小而專注的提示和上下文獲得更好的結果**

我交談過的大多數建造者在意識到超過 10-20 輪的任何內容都會變成 LLM 無法恢復的大混亂時，都**將「工具呼叫循環」的想法推到一邊**。即使 Agent 90% 的時間都能做對，這也遠遠達不到「足夠好到能交給客戶」的標準。你能想像一個在 10% 的頁面載入時會崩潰的網路應用程式嗎？

**2025-06-09 更新** - 我真的很喜歡 [@swyx](https://x.com/swyx/status/1932125643384455237) 的說法：

<a href="https://x.com/swyx/status/1932125643384455237"><img width="593" alt="Screenshot 2025-07-02 at 11 50 50 AM" src="https://github.com/user-attachments/assets/c7d94042-e4b9-4b87-87fd-55c7ff94bb3b" /></a>

### 真正有效的方法 - 微型 Agent

我**確實**在實際應用中經常看到的一件事是採用 Agent 模式並將其融入更廣泛、更確定性的 DAG 中。 

![micro-agent-dag](https://github.com/circleghost/12-factor-agents/blob/main/img/028-micro-agent-dag.png)

你可能會問 - 「在這種情況下為什麼要使用 Agent？」- 我們很快會討論這個問題，但基本上，讓語言模型管理範圍明確的任務集合，可以輕鬆整合即時人類回饋，將其轉化為工作流程步驟，而不會陷入上下文錯誤循環。([要素 1](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)、[要素 3](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)、[要素 7](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md))。

> #### 讓語言模型管理範圍明確的任務集合，可以輕鬆整合即時人類回饋...而不會陷入上下文錯誤循環

### 真實生活中的微型 Agent 

這裡是一個確定性程式碼如何運行一個負責處理部署中人在回路步驟的微型 Agent 的範例。 

![029-deploybot-high-level](https://github.com/circleghost/12-factor-agents/blob/main/img/029-deploybot-high-level.png)

* **人類** 將 PR 合併到 GitHub 主分支
* **確定性程式碼** 部署到測試環境
* **確定性程式碼** 對測試環境運行端到端 (e2e) 測試
* **確定性程式碼** 交給 Agent 進行生產部署，初始上下文：「deploy SHA 4af9ec0 to production」
* **Agent** 呼叫 `deploy_frontend_to_prod(4af9ec0)`
* **確定性程式碼** 請求人類對此動作的批准
* **人類** 拒絕此動作，並給出回饋「你能先部署後端嗎？」
* **Agent** 呼叫 `deploy_backend_to_prod(4af9ec0)`
* **確定性程式碼** 請求人類對此動作的批准
* **人類** 批准此動作
* **確定性程式碼** 執行後端部署
* **Agent** 呼叫 `deploy_frontend_to_prod(4af9ec0)`
* **確定性程式碼** 請求人類對此動作的批准
* **人類** 批准此動作
* **確定性程式碼** 執行前端部署
* **Agent** 確定任務已成功完成，我們完成了！
* **確定性程式碼** 對生產環境運行端到端測試
* **確定性程式碼** 任務完成，或交給回滾 Agent 檢查失敗並可能回滾

[![033-deploybot-animation](https://github.com/circleghost/12-factor-agents/blob/main/img/033-deploybot.gif)](https://github.com/user-attachments/assets/deb356e9-0198-45c2-9767-231cb569ae13)

<details>
<summary><a href="https://github.com/circleghost/12-factor-agents/blob/main/img/033-deploybot.gif">GIF Version</a></summary>

![033-deploybot-animation](https://github.com/circleghost/12-factor-agents/blob/main/img/033-deploybot.gif)

</details>

這個範例是基於一個真實的 [OSS Agent，我們在 Humanlayer 發布了管理部署的工具](https://github.com/got-agents/agents/tree/main/deploybot-ts) - 這裡是我上週與它的真實對話：

![035-deploybot-conversation](https://github.com/circleghost/12-factor-agents/blob/main/img/035-deploybot-conversation.png)


我們沒有給這個 Agent 一大堆工具或任務。LLM 的主要價值在於解析人類的純文字回饋並提出更新的行動方案。我們盡可能地隔離任務和上下文，以保持 LLM 專注於小而精的 5-10 步驟工作流程。

這裡是另一個[更經典的支援/聊天機器人演示](https://x.com/chainlit_io/status/1858613325921480922)。

### 那麼 Agent 到底是什麼？

- **提示** - 告訴 LLM 如何行為，以及它有哪些「工具」可用。提示的輸出是一個 JSON 物件，描述工作流程中的下一步 (「工具呼叫」或「函式呼叫」)。([要素 2](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md))
- **選擇陳述式** - 根據 LLM 返回的 JSON，決定如何處理它。([要素 8](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) 的一部分)
- **累積上下文** - 儲存已發生的步驟列表及其結果 ([要素 3](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md))
- **for 循環** - 直到 LLM 發出某種「終端」工具呼叫 (或純文字回應)，將選擇陳述式的結果新增到上下文視窗並要求 LLM 選擇下一步。([要素 8](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md))

![040-4-components](https://github.com/circleghost/12-factor-agents/blob/main/img/040-4-components.png)

在「deploybot」範例中，我們通過擁有控制流程和上下文累積獲得了一些好處：

- 在我們的**選擇陳述式**和 **for 循環**中，我們可以劫持控制流程以暫停等待人類輸入或等待長時間運行任務的完成
- 我們可以輕鬆地將**上下文**視窗序列化以實現暫停+恢復
- 在我們的**提示**中，我們可以大幅優化如何向 LLM 傳遞指令和「迴今為止發生的事情」


[第二部分](https://github.com/circleghost/12-factor-agents/blob/main/README.md#12-factor-agents) 將**正式化這些模式**，以便它們可以被應用於任何軟體專案中，新增令人印象深刻的 AI 功能，而無需完全採用傳統的「AI Agent」實現/定義。


[要素 1 - 自然語言到工具呼叫 →](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
