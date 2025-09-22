[← 回到 README](https://github.com/circleghost/12-factor-agents/blob/main/README.md)

### 3. 擁有你的上下文視窗

你不一定需要使用標準的基於訊息的格式來向 LLM 傳達上下文。

> #### 在任何給定的時刻，你對 Agent 中 LLM 的輸入就是「到目前為止發生了什麼，下一步是什麼」

<!-- todo syntax highlighting -->
<!-- ![130-own-your-context-building](https://github.com/circleghost/12-factor-agents/blob/main/img/130-own-your-context-building.png) -->

一切都是上下文工程。[LLM 是無狀態函數](https://thedataexchange.media/baml-revolution-in-ai-engineering/)，將輸入轉換為輸出。要獲得最佳輸出，你需要給它們最佳輸入。

建立優秀的上下文意味著：

- 你給模型的提示和指令
- 你檢索的任何文件或外部資料 (例如 RAG)
- 任何過去的狀態、工具呼叫、結果或其他歷史記錄
- 來自相關但獨立的歷史記錄/對話的任何過去訊息或事件 (記憶)
- 關於要輸出什麼種類結構化資料的指令

![image](https://github.com/user-attachments/assets/0f1f193f-8e94-4044-a276-576bd7764fd0)


### 關於上下文工程

本指南全部是關於從今天的模型中獲得盡可能多的價值。值得注意的是，沒有提到的包括：

- 修改模型參數，如 temperature、top_p、frequency_penalty、presence_penalty 等
- 訓練你自己的完成或嵌入模型
- 微調現有模型

再次強調，我不知道向 LLM 傳遞上下文的最佳方式是什麼，但我知道你希望能夠嘗試一切的靈活性。

#### 標準 vs 自訂上下文格式

大多數 LLM 客戶端使用這樣的標準基於訊息的格式：

```yaml
[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": "Can you deploy the backend?"
  },
  {
    "role": "assistant",
    "content": null,
    "tool_calls": [
      {
        "id": "1",
        "name": "list_git_tags",
        "arguments": "{}"
      }
    ]
  },
  {
    "role": "tool",
    "name": "list_git_tags",
    "content": "{\"tags\": [{\"name\": \"v1.2.3\", \"commit\": \"abc123\", \"date\": \"2024-03-15T10:00:00Z\"}, {\"name\": \"v1.2.2\", \"commit\": \"def456\", \"date\": \"2024-03-14T15:30:00Z\"}, {\"name\": \"v1.2.1\", \"commit\": \"abe033d\", \"date\": \"2024-03-13T09:15:00Z\"}]}",
    "tool_call_id": "1"
  }
]
```

雖然這對大多數使用案例都很好用，但如果你想真正從今天的 LLM 中獲得最大收益，你需要以最有效的標記和注意力方式將你的上下文輸入到 LLM 中。

作為標準基於訊息格式的替代方案，你可以建構針對你的使用案例最佳化的自己的上下文格式。例如，你可以使用自訂物件並將它們打包/展開到一個或多個使用者、系統、助理或工具訊息中，視情況而定。

這是將整個上下文視窗放入單一使用者訊息的例子：
```yaml

[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": |
            Here's everything that happened so far:
        
        <slack_message>
            From: @alex
            Channel: #deployments
            Text: Can you deploy the backend?
        </slack_message>
        
        <list_git_tags>
            intent: "list_git_tags"
        </list_git_tags>
        
        <list_git_tags_result>
            tags:
              - name: "v1.2.3"
                commit: "abc123"
                date: "2024-03-15T10:00:00Z"
              - name: "v1.2.2"
                commit: "def456"
                date: "2024-03-14T15:30:00Z"
              - name: "v1.2.1"
                commit: "ghi789"
                date: "2024-03-13T09:15:00Z"
        </list_git_tags_result>
        
        what's the next step?
    }
]
```

模型可能會根據你提供的工具架構推斷出你在問它「下一步是什麼」，但將其納入你的提示模板中永遠不會有害。

### 程式碼範例

我們可以用類似這樣的方式來建構：

```python

class Thread:
  events: List[Event]

class Event:
  # 可以只使用字串，或者可以明確 - 由你決定
  type: Literal["list_git_tags", "deploy_backend", "deploy_frontend", "request_more_information", "done_for_now", "list_git_tags_result", "deploy_backend_result", "deploy_frontend_result", "request_more_information_result", "done_for_now_result", "error"]
  data: ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation |  
        ListGitTagsResult | DeployBackendResult | DeployFrontendResult | RequestMoreInformationResult | string

def event_to_prompt(event: Event) -> str:
    data = event.data if isinstance(event.data, str) \
           else stringifyToYaml(event.data)

    return f"<{event.type}>\n{data}\n</{event.type}>"


def thread_to_prompt(thread: Thread) -> str:
  return '\n\n'.join(event_to_prompt(event) for event in thread.events)
```

#### 上下文視窗範例

使用這種方法，上下文視窗可能會是這樣的：

**初始 Slack 請求：**
```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
</slack_message>
```

**列出 Git 標籤之後：**
```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<list_git_tags>
    intent: "list_git_tags"
</list_git_tags>

<list_git_tags_result>
    tags:
      - name: "v1.2.3"
        commit: "abc123"
        date: "2024-03-15T10:00:00Z"
      - name: "v1.2.2"
        commit: "def456"
        date: "2024-03-14T15:30:00Z"
      - name: "v1.2.1"
        commit: "ghi789"
        date: "2024-03-13T09:15:00Z"
</list_git_tags_result>
```

**錯誤和恢復之後：**
```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<error>
    error running deploy_backend: Failed to connect to deployment service
</error>

<request_more_information>
    intent: "request_more_information_from_human"
    question: "I had trouble connecting to the deployment service, can you provide more details and/or check on the status of the service?"
</request_more_information>

<human_response>
    data:
      response: "I'm not sure what's going on, can you check on the status of the latest workflow?"
</human_response>
```

從這裡開始，你的下一步可能是： 

```python
nextStep = await determine_next_step(thread_to_prompt(thread))
```

```python
{
  "intent": "get_workflow_status",
  "workflow_name": "tag_push_prod.yaml",
}
```

XML 風格的格式只是一個例子 - 重點是你可以建構對你的應用程式有意義的自己的格式。如果你有靈活性來實驗不同的上下文結構以及你儲存什麼 vs. 你傳遞給 LLM 什麼，你會獲得更好的品質。

擁有你的上下文視窗的主要好處：

1. **資訊密度**：以最大化 LLM 理解的方式結構化資訊
2. **錯誤處理**：以幫助 LLM 恢復的格式包含錯誤資訊。考慮在錯誤和失敗呼叫解決後從上下文視窗中隱藏它們。
3. **安全性**：控制傳遞給 LLM 的資訊，過濾敏感資料
4. **靈活性**：隨著你了解什麼對你的使用案例最有效而調整格式
5. **標記效率**：為標記效率和 LLM 理解最佳化上下文格式

上下文包括：提示、指令、RAG 文件、歷史記錄、工具呼叫、記憶


記住：上下文視窗是你與 LLM 的主要介面。控制你如何結構化和呈現資訊可以顯著提升你的 Agent 性能。

範例 - 資訊密度 - 相同訊息，更少標記：

![Loom Screenshot 2025-04-22 at 09 00 56](https://github.com/user-attachments/assets/5cf041c6-72da-4943-be8a-99c73162b12a)


### 不要相信我的話

十二要素 Agent 發布約 2 個月後，上下文工程開始成為一個相當流行的術語。

<a href="https://x.com/karpathy/status/1937902205765607626"><img width="378" alt="Screenshot 2025-06-25 at 4 11 45 PM" src="https://github.com/user-attachments/assets/97e6e667-c35f-4855-8233-af40f05d6bce" /></a> <a href="https://x.com/tobi/status/1935533422589399127"><img width="378" alt="Screenshot 2025-06-25 at 4 12 59 PM" src="https://github.com/user-attachments/assets/7e6f5738-0d38-4910-82d1-7f5785b82b99" /></a>

還有一個相當不錯的 [上下文工程速查表](https://x.com/lenadroid/status/1943685060785524824)，來自 [@lenadroid](https://x.com/lenadroid)，發布於 2025 年 7 月。

<a href="https://x.com/lenadroid/status/1943685060785524824"><img width="256" alt="image" src="https://github.com/user-attachments/assets/cac88aa3-8faf-440b-9736-cab95a9de477" /></a>



這裡的反覆主題：我不知道什麼是最好的方法，但我知道你希望能夠嘗試一切的靈活性。


[← 擁有你的提示](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-02-own-your-prompts.md) | [工具只是結構化輸出 →](https://github.com/circleghost/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)
