[← 回到 README](https://github.com/circleghost/12-factor-agents/blob/main/README.md)

### 11. 從任何地方觸發，在使用者所在的地方與他們會面

如果您正在等待 [humanlayer](https://humanlayer.dev) 的推銷，您做到了。如果您正在執行[要素 6 - 使用簡單 API 啟動/暫停/恢復](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)和[要素 7 - 使用工具呼叫聯繫人類](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)，您已經準備好整合這個要素。

![1b0-trigger-from-anywhere](https://github.com/circleghost/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)

讓使用者能夠從 Slack、電子郵件、簡訊或他們想要的任何其他管道觸發 Agent。讓 Agent 能夠透過相同的管道回應。

好處：

- **在使用者所在的地方與他們會面**：這有助於您建構感覺像真人，或至少像數位同事的 AI 應用程式
- **外層迴圈 Agent**：讓 Agent 能夠被非人類觸發，例如事件、定時任務、中斷、其他任何事物。它們可能工作 5、20、90 分鐘，但當它們到達關鍵點時，它們可以聯繫人類尋求幫助、回饋或批准
- **高風險工具**：如果您能夠快速引入各種人類，您可以讓 Agent 存取更高風險的操作，如發送外部電子郵件、更新生產資料等。維持清楚的標準可以為您帶來[執行更大更好事物](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter)的 Agent 的可稽核性和信心

[← 小型專注的 Agent](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md) | [無狀態歸約器 →](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)