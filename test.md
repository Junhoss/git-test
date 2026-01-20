## 4. 🏗️ 시스템 아키텍처 (System Architecture)

전체 시스템은 **'문제 추천(Morning Routine)'**과 **'풀이 인증(Solving Routine)'**의 두 가지 주요 워크플로우로 구성되어 있습니다.

```mermaid
graph TD
    %% 스타일 정의
    classDef trigger fill:#f9f,stroke:#333,stroke-width:2px;
    classDef action fill:#bbf,stroke:#333,stroke-width:2px;
    classDef db fill:#bfb,stroke:#333,stroke-width:2px;

    subgraph "☀️ Morning Routine (Daily Recommendation)"
        %% 텍스트에 괄호가 있으므로 ""로 감싸주어야 함
        Cron["📅 Schedule Trigger<br/>(Every 09:00 AM)"]:::trigger --> API["⚡ HTTP Request<br/>(Solved.ac API)"]
        API -->|Filter: Silver 3~5| SlackBot["💬 Slack Bot<br/>(Send Interactive Msg)"]:::action
        
        User((User)) -->|Click 'Accept'| Webhook[🪝 Webhook Receiver]:::trigger
        Webhook -->|Switch: Action ID| Logic{Is 'Approve'?}
        Logic -- Yes --> NotionCreate["📝 Notion: Create Page<br/>(Status: To-Do)"]:::db
    end

    subgraph "💻 Solving Routine (Automated Verification)"
        GitPush["🚀 GitHub Push<br/>(Code Commit)"]:::trigger --> GitTrigger[n8n: GitHub Trigger]
        GitTrigger -->|Filter| CheckCommit{"Msg contains<br/>'BaekjoonHub'?"}
        
        CheckCommit -- Yes --> Parse["⚙️ Data Parsing<br/>(Title, Level, URL)"]
        Parse -->|Sync| NotionUpdate["🔄 Notion: Update Page<br/>(Status: Done, URL)"]:::db
        Parse -->|Notify| SlackDone[🎉 Slack: Success Alert]:::action
    end
