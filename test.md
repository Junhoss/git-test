## 4. 🏗️ 시스템 아키텍처 (System Architecture)

전체 시스템은 **'문제 추천(Morning Routine)'**과 **'풀이 인증(Solving Routine)'**의 두 가지 주요 워크플로우로 구성되어 있습니다.

```mermaid
graph TD
    %% 스타일 정의
    classDef trigger fill:#f9f,stroke:#333,stroke-width:2px;
    classDef action fill:#bbf,stroke:#333,stroke-width:2px;
    classDef db fill:#bfb,stroke:#333,stroke-width:2px;

    %% 1. 아침 루틴
    subgraph "☀️ 아침 루틴 (일일 문제 추천)"
        Cron["📅 스케줄 트리거<br/>(매일 오전 09:00)"]:::trigger --> API["⚡ HTTP 요청<br/>(Solved.ac API)"]
        API -->|필터: 실버 3~5| SlackBot["💬 슬랙 봇<br/>(버튼 포함 메시지 전송)"]:::action
        
        User((사용자)) -->|'수락' 클릭| Webhook[🪝 웹훅 수신]:::trigger
        Webhook -->|Switch: 액션 ID 확인| Logic{승인(Approve)<br/>버튼인가?}
        Logic -- Yes --> NotionCreate["📝 노션: 페이지 생성<br/>(상태: 할 일)"]:::db
    end

    %% 2. 문제 풀이 루틴
    subgraph "💻 문제 풀이 루틴 (자동 인증)"
        GitPush["🚀 깃허브 푸시<br/>(코드 커밋)"]:::trigger --> GitTrigger[n8n: 깃허브 트리거]
        GitTrigger -->|필터링| CheckCommit{"메시지에<br/>'BaekjoonHub'<br/>포함?"}
        
        CheckCommit -- Yes --> Parse["⚙️ 데이터 파싱<br/>(제목, 난이도, URL)"]
        Parse -->|동기화| NotionUpdate["🔄 노션: 페이지 업데이트<br/>(상태: 완료, URL)"]:::db
        Parse -->|알림| SlackDone[🎉 슬랙: 풀이 성공 알림]:::action
    end

    %% [레이아웃 조정] 아침 루틴이 위, 풀이 루틴이 아래에 오도록 투명 연결선 추가
    NotionCreate ~~~ GitPush
