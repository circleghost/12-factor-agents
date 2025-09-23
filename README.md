# 十二要素 Agents - 建構可靠大語言模型應用程式的原則

<div align="center">
<a href="https://www.apache.org/licenses/LICENSE-2.0">
        <img src="https://img.shields.io/badge/Code-Apache%202.0-blue.svg" alt="Code License: Apache 2.0"></a>
<a href="https://creativecommons.org/licenses/by-sa/4.0/">
        <img src="https://img.shields.io/badge/Content-CC%20BY--SA%204.0-lightgrey.svg" alt="Content License: CC BY-SA 4.0"></a>
<a href="https://humanlayer.dev/discord">
    <img src="https://img.shields.io/badge/chat-discord-5865F2" alt="Discord Server"></a>
<a href="https://www.youtube.com/watch?v=8kMaTybvDUw">
    <img src="https://img.shields.io/badge/aidotengineer-conf_talk_(17m)-white" alt="YouTube
Deep Dive"></a>
<a href="https://www.youtube.com/watch?v=yxJDyQ8v6P0">
    <img src="https://img.shields.io/badge/youtube-deep_dive-crimson" alt="YouTube
Deep Dive"></a>
    
</div>

<p></p>

*秉承 [十二要素應用程式](https://12factor.net/) 的精神*。*本專案的原始碼公開在 https://github.com/circleghost/12-factor-agents ，歡迎你的回饋和貢獻。讓我們一起探索這個領域！*

> [!TIP]
> 錯過了 AI Engineer World's Fair？[觀看演講錄影](https://www.youtube.com/watch?v=8kMaTybvDUw)
>
> 想了解 Context Engineering？[直接跳到要素 3](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
>
> 想為 `npx/uvx create-12-factor-agent` 貢獻？查看[討論串](https://github.com/circleghost/12-factor-agents/discussions/61)


<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=2acad99a-c2d9-48df-86f5-9ca8061b7bf9" />

<a href="#visual-nav"><img width="907" alt="Screenshot 2025-04-03 at 2 49 07 PM" src="https://github.com/user-attachments/assets/23286ad8-7bef-4902-b371-88ff6a22e998" /></a>


嗨，我是 Dex。我一直在[探索](https://youtu.be/8bIHcttkOTE) [AI agents](https://theouterloop.substack.com) 已經[一段時間了](https://humanlayer.dev)。


**我已經試過市面上所有的 agent 框架**，從隨插即用的 crew/langchains 到世界上「極簡主義」的 smolagents，再到「生產級」的 langraph、griptape 等等。

**我與許多非常優秀的創辦人交談過**，無論是 YC 內外的，他們都在用 AI 建造令人印象深刻的東西。他們大多數都在自己搭建技術棧。我在生產環境面向客戶的 agents 中沒有看到太多框架的身影。

**我驚訝地發現**，市面上大多數自稱為「AI Agents」的產品實際上並不那麼具有智能體特性。它們大多是確定性的程式碼，只是在恰當的地方點綴一些 LLM 步驟，讓體驗真正神奇。

Agents，至少是好的 agents，不會遵循[「這是你的提示詞，這是一包工具，循環直到達成目標」](https://www.anthropic.com/engineering/building-effective-agents#agents)的模式。相反，它們主要由軟體組成。

所以，我著手回答：

> ### **我們可以使用什麼原則來建構真正足夠好、能夠交付給生產客戶使用的 LLM 驅動軟體？**

歡迎來到十二要素 agents。正如芝加哥自 Daley 以來的每一位市長都在該市主要機場一致地到處張貼的標語一樣，我們很高興你來到這裡。

*特別感謝 [@iantbutler01](https://github.com/iantbutler01)、[@tnm](https://github.com/tnm)、[@hellovai](https://www.github.com/hellovai)、[@stantonk](https://www.github.com/stantonk)、[@balanceiskey](https://www.github.com/balanceiskey)、[@AdjectiveAllison](https://www.github.com/AdjectiveAllison)、[@pfbyjy](https://www.github.com/pfbyjy)、[@a-churchill](https://www.github.com/a-churchill) 以及 SF MLOps 社群對本指南的早期回饋。*

## 簡短版本：十二要素

即使 LLMs [持續以指數方式變得更強大](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter)，仍然會有核心的工程技術能夠讓 LLM 驅動的軟體更可靠、更可擴展、更易於維護。

- [我們如何走到這裡：軟體簡史](https://github.com/circleghost/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [要素 1：自然語言到工具呼叫](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [要素 2：擁有你的提示詞](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [要素 3：擁有你的上下文視窗](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [要素 4：工具只是結構化輸出](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [要素 5：統一執行狀態與業務狀態](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [要素 6：用簡單的 APIs 啟動/暫停/恢復](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [要素 7：透過工具呼叫聯繫人類](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [要素 8：擁有你的控制流程](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [要素 9：將錯誤壓縮到上下文視窗](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [要素 10：小型、專注的 Agents](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [要素 11：從任何地方觸發，在使用者所在的地方與他們相遇](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [要素 12：讓你的 agent 成為無狀態的 reducer](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)

### 視覺導航

|    |    |    |
|----|----|-----|
|[![factor 1](https://github.com/circleghost/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md) | [![factor 2](https://github.com/circleghost/12-factor-agents/blob/main/img/120-own-your-prompts.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md) | [![factor 3](https://github.com/circleghost/12-factor-agents/blob/main/img/130-own-your-context-building.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md) |
|[![factor 4](https://github.com/circleghost/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md) | [![factor 5](https://github.com/circleghost/12-factor-agents/blob/main/img/150-unify-state.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md) | [![factor 6](https://github.com/circleghost/12-factor-agents/blob/main/img/160-pause-resume-with-simple-apis.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md) |
| [![factor 7](https://github.com/circleghost/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md) | [![factor 8](https://github.com/circleghost/12-factor-agents/blob/main/img/180-control-flow.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md) | [![factor 9](https://github.com/circleghost/12-factor-agents/blob/main/img/190-factor-9-errors-static.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-09-compact-errors.md) |
| [![factor 10](https://github.com/circleghost/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md) | [![factor 11](https://github.com/circleghost/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md) | [![factor 12](https://github.com/circleghost/12-factor-agents/blob/main/img/1c0-stateless-reducer.png)](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md) |

## 我們如何走到這裡

想深入了解我的 agent 之旅以及是什麼讓我們走到這裡，請查看[軟體簡史](https://github.com/circleghost/12-factor-agents/blob/main/content/brief-history-of-software.md) - 這裡是快速摘要：

### Agents 的承諾

我們將大量討論有向圖 (DGs) 和它們的無環朋友 DAGs。我首先要指出的是...嗯...軟體就是一個有向圖。我們過去用流程圖來表示程式是有原因的。

![010-software-dag](https://github.com/circleghost/12-factor-agents/blob/main/img/010-software-dag.png)

### 從程式碼到 DAGs

大約 20 年前，我們開始看到 DAG 編排器變得流行。我們說的是像 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/) 這樣的經典工具，一些前身，以及一些較新的工具如 ([dagster](https://dagster.io/)、[inggest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/))。這些都遵循相同的圖形模式，但增加了可觀測性、模組化、重試、管理等好處。

![015-dag-orchestrators](https://github.com/circleghost/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### Agents 的承諾

我不是第一個[這樣說的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但當我開始學習 agents 時，我最大的收穫是你可以拋棄 DAG。軟體工程師不用編碼每個步驟和邊緣情況，你可以給 agent 一個目標和一組轉換：

![025-agent-dag](https://github.com/circleghost/12-factor-agents/blob/main/img/025-agent-dag.png)

並讓 LLM 即時做決定來找出路徑

![026-agent-dag-lines](https://github.com/circleghost/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

這裡的承諾是你寫更少的軟體，你只需要給 LLM 圖的「邊」，讓它來找出節點。你可以從錯誤中恢復，你可以寫更少的程式碼，你可能會發現 LLMs 為問題找到新穎的解決方案。


### Agents 作為迴圈

正如我們稍後會看到的，事實證明這並不完全奏效。

讓我們再深入一步 - 使用 agents 時，你有一個包含 3 個步驟的迴圈：

1. LLM 決定工作流程中的下一步，輸出結構化 json (「tool calling」)
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

我們的初始上下文只是開始事件 (可能是使用者訊息，可能是 cron 觸發，可能是 webhook 等)，我們要求 llm 選擇下一步 (工具) 或決定我們已經完成。

這是一個多步驟的例子：

[![027-agent-loop-animation](https://github.com/circleghost/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/circleghost/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF Version</a></summary>

![027-agent-loop-animation](https://github.com/circleghost/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

## 為什麼是十二要素 agents？

歸根究底，這種方法並不像我們希望的那樣有效。

在建構 HumanLayer 時，我與至少 100 位 SaaS 建構者 (主要是技術創辦人) 交談過，他們希望讓現有產品更具智能體特性。這個旅程通常是這樣的：

1. 決定你想建構一個 agent
2. 產品設計、UX 對應、要解決什麼問題
3. 想要快速行動，所以抓取 $FRAMEWORK 並*開始建構*
4. 達到 70-80% 的品質標準
5. 意識到 80% 對於大多數面向客戶的功能來說是不夠的
6. 意識到要超越 80% 需要逆向工程框架、提示詞、流程等
7. 從頭開始

<details>
<summary>一些免責聲明</summary>

**免責聲明**：我不確定在哪裡說這個是最合適的，但這裡似乎和其他地方一樣好：**這絕不是想要批評市面上的許多框架，或者那些從事這些框架工作的相當聰明的人們**。它們實現了令人難以置信的事情，並加速了 AI 生態系統的發展。

我希望這篇文章的一個結果是，agent 框架建構者可以從我和其他人的旅程中學習，讓框架變得更好。

特別是對於那些想要快速行動但需要深度控制的建構者。

**免責聲明 2**：我不會談論 MCP。我確信你能看出它適合放在哪裡。

**免責聲明 3**：我主要使用 typescript，有[原因](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA4oHTkByAiD-wZjnGsMBUL_JT6nyyhOh30)的，但所有這些東西在 python 或任何其他你喜歡的語言中都有效。


總之，回到正題...

</details>

### 優秀 LLM 應用程式的設計模式

在挖掘了數百個 AI 函式庫並與數十位創辦人合作後，我的直覺是這樣的：

1. 有一些核心的東西讓 agents 變得優秀
2. 全力投入一個框架並建構本質上是全新重寫的東西可能會適得其反
3. 有一些核心原則讓 agents 變得優秀，如果你引入一個框架，你會得到大部分/全部這些原則
4. 但是，我見過的讓建構者將高品質 AI 軟體交付給客戶的最快方式是從 agent 建構中取用小型、模組化的概念，並將它們融入到現有產品中
5. 這些來自 agents 的模組化概念可以由大多數熟練的軟體工程師定義和應用，即使他們沒有 AI 背景

> #### 我見過的讓建構者將優秀 AI 軟體交付給客戶的最快方式是從 agent 建構中取用小型、模組化的概念，並將它們融入到現有產品中


## 十二要素 (再次)


- [我們如何走到這裡：軟體簡史](https://github.com/circleghost/12-factor-agents/blob/main/content/brief-history-of-software.md)
- [要素 1：自然語言到工具呼叫](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md)
- [要素 2：擁有你的提示詞](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md)
- [要素 3：擁有你的上下文視窗](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
- [要素 4：工具只是結構化輸出](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
- [要素 5：統一執行狀態與業務狀態](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)
- [要素 6：用簡單的 APIs 啟動/暫停/恢復](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-06-launch-pause-resume.md)
- [要素 7：透過工具呼叫聯繫人類](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools.md)
- [要素 8：擁有你的控制流程](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)
- [要素 9：將錯誤壓縮到上下文視窗](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-09-compact-errors.md)
- [要素 10：小型、專注的 Agents](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md)
- [要素 11：從任何地方觸發，在使用者所在的地方與他們相遇](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere.md)
- [要素 12：讓你的 agent 成為無狀態的 reducer](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)

## 榮譽提及 / 其他建議

- [要素 13：預取所有你可能需要的上下文](https://github.com/circleghost/12-factor-agents/blob/main/content/appendix-13-pre-fetch.md)

## 相關資源

- 在[這裡](https://github.com/circleghost/12-factor-agents)為本指南做出貢獻
- [我在 2025 年 3 月的 Tool Use podcast 節目中談論了很多這些內容](https://youtu.be/8bIHcttkOTE)
- 我在 [The Outer Loop](https://theouterloop.substack.com) 寫一些關於這些內容的文章
- 我與 [@hellovai](https://github.com/hellovai) 一起舉辦[關於最大化 LLM 效能的網路研討會](https://github.com/hellovai/ai-that-works/tree/main)
- 我們在 [got-agents/agents](https://github.com/got-agents/agents) 下使用這種方法建構 OSS agents
- 我們忽略了自己所有的建議，建構了一個[在 kubernetes 中執行分散式 agents 的框架](https://github.com/humanlayer/kubechain)
- 本指南的其他連結：
  - [12 Factor Apps](https://12factor.net)
  - [建構有效的 Agents (Anthropic)](https://www.anthropic.com/engineering/building-effective-agents#agents)
  - [Prompts are Functions](https://thedataexchange.media/baml-revolution-in-ai-engineering/ )
  - [函式庫模式：為什麼框架是邪惡的](https://tomasp.net/blog/2015/library-frameworks/)
  - [錯誤的抽象](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
  - [Mailcrew Agent](https://github.com/dexhorthy/mailcrew)
  - [Mailcrew 示範影片](https://www.youtube.com/watch?v=f_cKnoPC_Oo)
  - [Chainlit 示範](https://x.com/chainlit_io/status/1858613325921480922)
  - [TypeScript for LLMs](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e)
  - [Schema Aligned Parsing](https://www.boundaryml.com/blog/schema-aligned-parsing)
  - [Function Calling vs Structured Outputs vs JSON Mode](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode)
  - [BAML on GitHub](https://github.com/boundaryml/baml)
  - [OpenAI JSON vs Function Calling](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)
  - [Outer Loop Agents](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)
  - [Airflow](https://airflow.apache.org/)
  - [Prefect](https://www.prefect.io/)
  - [Dagster](https://dagster.io/)
  - [Inngest](https://www.inngest.com/)
  - [Windmill](https://www.windmill.dev/)
  - [The AI Agent Index (MIT)](https://aiagentindex.mit.edu/)
  - [NotebookLM on Finding Model Capability Boundaries](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8)

## 貢獻者

感謝所有為十二要素 agents 做出貢獻的人！

[<img src="https://avatars.githubusercontent.com/u/3730605?v=4&s=80" width="80px" alt="dexhorthy" />](https://github.com/dexhorthy) [<img src="https://avatars.githubusercontent.com/u/50557586?v=4&s=80" width="80px" alt="Sypherd" />](https://github.com/Sypherd) [<img src="https://avatars.githubusercontent.com/u/66259401?v=4&s=80" width="80px" alt="tofaramususa" />](https://github.com/tofaramususa) [<img src="https://avatars.githubusercontent.com/u/18105223?v=4&s=80" width="80px" alt="a-churchill" />](https://github.com/a-churchill) [<img src="https://avatars.githubusercontent.com/u/4084885?v=4&s=80" width="80px" alt="Elijas" />](https://github.com/Elijas) [<img src="https://avatars.githubusercontent.com/u/39267118?v=4&s=80" width="80px" alt="hugolmn" />](https://github.com/hugolmn) [<img src="https://avatars.githubusercontent.com/u/1882972?v=4&s=80" width="80px" alt="jeremypeters" />](https://github.com/jeremypeters)

[<img src="https://avatars.githubusercontent.com/u/380402?v=4&s=80" width="80px" alt="kndl" />](https://github.com/kndl) [<img src="https://avatars.githubusercontent.com/u/16674643?v=4&s=80" width="80px" alt="maciejkos" />](https://github.com/maciejkos) [<img src="https://avatars.githubusercontent.com/u/85041180?v=4&s=80" width="80px" alt="pfbyjy" />](https://github.com/pfbyjy) [<img src="https://avatars.githubusercontent.com/u/36044389?v=4&s=80" width="80px" alt="0xRaduan" />](https://github.com/0xRaduan) [<img src="https://avatars.githubusercontent.com/u/7169731?v=4&s=80" width="80px" alt="zyuanlim" />](https://github.com/zyuanlim) [<img src="https://avatars.githubusercontent.com/u/15862501?v=4&s=80" width="80px" alt="lombardo-chcg" />](https://github.com/lombardo-chcg) [<img src="https://avatars.githubusercontent.com/u/160066852?v=4&s=80" width="80px" alt="sahanatvessel" />](https://github.com/sahanatvessel)
 
## 授權

所有內容和圖像均根據 <a href="https://creativecommons.org/licenses/by-sa/4.0/">CC BY-SA 4.0 授權</a> 授權

程式碼根據 <a href="https://www.apache.org/licenses/LICENSE-2.0">Apache 2.0 授權</a> 授權


