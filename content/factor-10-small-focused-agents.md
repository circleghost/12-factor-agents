[← 回到 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 10. 小型、專注的 Agent

與其建構試圖包攬一切的單體 Agent，不如建構專注於一件事並做好的小型 Agent。Agent 只是更大的、主要是確定性系統中的一個建構塊。

![1a0-small-focused-agents](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)

這裡的關鍵見解與 LLM 的限制有關：任務越大越複雜，需要的步驟就越多，這意味著更長的上下文視窗。隨著上下文的增長，LLM 更容易迷失或失去焦點。透過保持 Agent 專注於特定領域，最多 3-10 步，也許最多 20 步，我們可以保持上下文視窗的可管理性和 LLM 效能的高水準。

> #### 隨著上下文的增長，LLM 更容易迷失或失去焦點

小型、專注 Agent 的好處：

1. **可管理的上下文**：較小的上下文視窗意味著更好的 LLM 效能
2. **明確的責任**：每個 Agent 都有明確定義的範圍和目的
3. **更好的可靠性**：在複雜工作流程中迷失的機會較少
4. **更容易測試**：更容易測試和驗證特定功能
5. **改善除錯**：發生問題時更容易識別和修復

### 如果 LLM 變得更聰明會怎樣？

如果 LLM 變得足夠聰明，能夠處理 100 步以上的工作流程，我們還需要這個嗎？

簡而言之，是的。隨著 Agent 和 LLM 的改進，它們**可能**自然地擴展到能夠處理更長的上下文視窗。這意味著處理更大 DAG 的更多部分。這種小型、專注的方法確保您今天就能獲得結果，同時為您準備隨著 LLM 上下文視窗變得更可靠而慢慢擴展 Agent 範圍。(如果您之前重構過大型確定性程式碼庫，您現在可能正在點頭)。

[![gif](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)](https://github.com/user-attachments/assets/0cd3f52c-046e-4d5e-bab4-57657157c82f
)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif">GIF 版本</a></summary>
![gif](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)
</details>

對 Agent 的大小/範圍保持有意識的態度，並且只以允許您維持品質的方式成長，這是關鍵。正如[建構 NotebookLM 的團隊所說](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8&utm_campaign=post-share-selection&utm_medium=web)：

> 我覺得持續地，對我來說，AI 建構中最神奇的時刻總是在我真的、真的、真的非常接近模型能力邊緣的時候出現

無論這個邊界在哪裡，如果您能找到這個邊界並持續做對，您就會建構神奇的體驗。這裡有許多護城河可以建構，但像往常一樣，它們需要一些工程嚴謹性。

[← 壓縮錯誤](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-09-compact-errors.md) | [從任何地方觸發 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
