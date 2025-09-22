[← 回到 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 2. 擁有你的提示

不要將你的提示工程外包給框架。

![120-own-your-prompts](https://github.com/humanlayer/12-factor-agents/blob/main/img/120-own-your-prompts.png)

順便說一下，[這遠非新穎的建議：](https://hamel.dev/blog/posts/prompt/)

![image](https://github.com/user-attachments/assets/575bab37-0f96-49fb-9ce3-9a883cdd420b)

一些框架提供這樣的「黑盒」方法：

```python
agent = Agent(
  role="...",
  goal="...",
  personality="...",
  tools=[tool1, tool2, tool3]
)

task = Task(
  instructions="...",
  expected_output=OutputModel
)

result = agent.run(task)
```

這對於引入一些頂級的提示工程來幫你入門很有用，但要調整和/或逆向工程以獲得確切的正確標記輸入到你的模型中往往很困難。

相反，擁有你的提示並將其視為一流的程式碼：

```rust
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
  prompt #"
    {{ _.role("system") }}
    
    You are a helpful assistant that manages deployments for frontend and backend systems.
    You work diligently to ensure safe and successful deployments by following best practices
    and proper deployment procedures.
    
    Before deploying any system, you should check:
    - The deployment environment (staging vs production)
    - The correct tag/version to deploy
    - The current system status
    
    You can use tools like deploy_backend, deploy_frontend, and check_deployment_status
    to manage deployments. For sensitive deployments, use request_approval to get
    human verification.
    
    Always think about what to do first, like:
    - Check current deployment status
    - Verify the deployment tag exists
    - Request approval if needed
    - Deploy to staging before production
    - Monitor deployment progress
    
    {{ _.role("user") }}

    {{ thread }}
    
    What should the next step be?
  "#
}
```

(上面的例子使用 [BAML](https://github.com/boundaryml/baml) 來生成提示，但你可以使用任何你想要的提示工程工具，甚至只是手動模板化)

如果簽名看起來有點奇怪，我們會在[要素 4 - 工具只是結構化輸出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)中介紹

```typescript
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
```

擁有你的提示的主要好處：

1. **完全控制**：編寫你的 Agent 需要的確切指令，沒有黑盒抽象
2. **測試和評估**：為你的提示建構測試和評估，就像對任何其他程式碼一樣
3. **迭代**：根據實際性能快速修改提示
4. **透明度**：確切知道你的 Agent 正在使用什麼指令
5. **角色 Hacking**：利用支援非標準使用者/助理角色用法的 API - 例如，現已棄用的 OpenAI「completions」API 的非聊天版本。這包括一些所謂的「模型煽動」技術

記住：你的提示是你的應用程式邏輯和 LLM 之間的主要介面。

完全控制你的提示為你提供了生產級 Agent 所需的靈活性和提示控制。

我不知道什麼是最好的提示，但我知道你希望能夠嘗試一切的靈活性。

[← 自然語言到工具呼叫](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls.md) | [擁有你的上下文視窗 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)
