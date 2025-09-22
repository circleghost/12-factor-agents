[← 回到 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 6. 使用簡單 API 進行啟動/暫停/恢復

Agent 只是程式，我們對如何啟動、查詢、恢復和停止它們有一定的期望。

[![pause-resume animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)](https://github.com/user-attachments/assets/feb1a425-cb96-4009-a133-8bd29480f21f)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif">GIF Version</a></summary>

![pause-resume animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)

</details>


使用者、應用程式、管道和其他 Agent 應該能夠輕鬆地透過簡單的 API 啟動 Agent。

Agent 及其編排的確定性程式碼應該能夠在需要長時間運行的操作時暫停 Agent。

像 webhook 這樣的外部觸發器應該能夠讓 Agent 從中斷的地方恢復，而無需與 Agent 編排器深度整合。

與[要素 5 - 統一執行狀態與業務狀態](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)和[要素 8 - 擁有你的控制流程](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)密切相關，但可以獨立實作。



**注意** - 通常 AI 編排器會允許暫停和恢復，但不會在工具選擇和工具執行之間進行。另請參閱[要素 7 - 透過工具呼叫聯繫人類](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)和[要素 11 - 從任何地方觸發，在使用者所在的地方與他們會面](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)。

[← 統一執行狀態](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) | [透過工具聯繫人類 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)