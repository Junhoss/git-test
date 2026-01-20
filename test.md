## 4. 🏗️ 시스템 아키텍처 (System Architecture)

전체 시스템은 **'문제 추천(Morning Routine)'**과 **'풀이 인증(Solving Routine)'**의 두 가지 주요 워크플로우로 구성되어 있습니다.

```mermaid
graph TD
    %% 스타일 정의
    classDef trigger fill:#f9f,stroke:#333,stroke-width:2px;
    classDef action fill:#bbf,stroke:#333,stroke-width:2px;
    classDef db fill:#bfb,stroke:#333,stroke-width:2px;

    %% 1. 아침 루틴
    subgraph "☀️ Morning Routine (Daily Recommendation)"
        Cron["📅 Schedule Trigger<br/>(Every 09:00 AM)"]:::trigger --> API["⚡ HTTP Request<br/>(Solved.ac API)"]
        API -->|Filter: Silver 3~5| SlackBot["💬 Slack Bot<br/>(Send Interactive Msg)"]:::action
        
        User((User)) -->|Click 'Accept'| Webhook[🪝 Webhook Receiver]:::trigger
        Webhook -->|Switch: 액션 ID 확인| Logic{"승인(Approve)<br/>버튼인가?"}
        Logic -- Yes --> NotionCreate["📝 Notion: Create Page<br/>(Status: To-Do)"]:::db
    end

    %% 2. 문제 풀이 루틴
    subgraph "💻 Solving Routine (Automated Verification)"
        GitPush["🚀 GitHub Push<br/>(Code Commit)"]:::trigger --> GitTrigger[n8n: GitHub Trigger]
        GitTrigger -->|Filter| CheckCommit{"Msg contains<br/>'BaekjoonHub'?"}
        
        CheckCommit -- Yes --> Parse["⚙️ Data Parsing<br/>(Title, Level, URL)"]
        Parse -->|Sync| NotionUpdate["🔄 Notion: Update Page<br/>(Status: Done, URL)"]:::db
        Parse -->|Notify| SlackDone[🎉 Slack: Success Alert]:::action
    end

    %% 위아래 배치 강제
    NotionCreate ~~~ GitPush
