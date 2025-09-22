[← 回到 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 5. 統一執行狀態與業務狀態

即使在 AI 世界之外，許多基礎設施系統都試圖將「執行狀態」與「業務狀態」分離。對於 AI 應用程式來說，這可能涉及複雜的抽象來追蹤諸如當前步驟、下一步驟、等待狀態、重試次數等事項。這種分離會產生複雜性，雖然可能是值得的，但對於你的使用案例來說可能過於複雜。

一如既往，由你來決定什麼適合你的應用程式。但不要認為你*必須*分別管理它們。

更明確地說：

- **執行狀態**：當前步驟、下一步驟、等待狀態、重試次數等。
- **業務狀態**：到目前為止在 Agent 工作流程中發生的事情 (例如 OpenAI 訊息列表、工具呼叫和結果列表等)。

如果可能的話，簡化 - 盡可能統一這些狀態。 

[![155-unify-state](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)](https://github.com/user-attachments/assets/e5a851db-f58f-43d8-8b0c-1926c99fc68d)


<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif">GIF Version</a></summary>

![155-unify-state](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)

</details>

實際上，你可以設計你的應用程式，讓你能夠從上下文視窗推斷出所有執行狀態。在許多情況下，執行狀態 (當前步驟、等待狀態等) 只是關於到目前為止發生了什麼的元資料。

你可能有一些無法放入上下文視窗的內容，如會話 ID、密碼上下文等，但你的目標應該是盡量減少這些內容。透過採用[要素 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)，你可以控制實際進入 LLM 的內容

這種方法有幾個好處：

1. **簡潔性**：所有狀態的唯一真實來源
2. **序列化**：執行緒可以輕鬆序列化/反序列化
3. **除錯**：整個歷史記錄在一個地方可見
4. **靈活性**：只需添加新的事件類型即可輕鬆添加新狀態
5. **恢復**：只需載入執行緒即可從任何點恢復
6. **分叉**：可以透過將執行緒的某些子集複製到新的上下文/狀態 ID 中，在任何點分叉執行緒
7. **人機介面和可觀測性**：可以輕鬆將執行緒轉換為人類可讀的 Markdown 或豐富的 Web 應用程式 UI

[← 工具是結構化輸出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) | [啟動/暫停/恢復 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
