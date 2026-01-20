graph TD
    %% 스타일 정의 (줄바꿈을 해야 아래 코드가 작동합니다)
    classDef trigger fill:#f9f,stroke:#333,stroke-width:2px;
    classDef action fill:#bbf,stroke:#333,stroke-width:2px;
    classDef db fill:#bfb,stroke:#333,stroke-width:2px;

    %% 1. 아침 루틴
    subgraph "☀️ 아침 루틴 (일일 문제 추천)"
        Cron["📅 스케줄 트리거<br/>(매일 오전 09:00)"]:::trigger --> API["⚡ HTTP 요청<br/>(Solved.ac API)"]
        API -->|필터: 실버 3~5| SlackBot["💬 슬랙 봇<br/>(메시지 전송)"]:::action
        
        User((사용자)) -->|'수락' 클릭| Webhook[🪝 웹훅 수신]:::trigger
        Webhook -->|분기: 액션 ID| Logic{"승인(Approve)<br/>버튼인가?"}
        Logic -- 네 --> NotionCreate["📝 노션: 페이지 생성<br/>(상태: 할 일)"]:::db
    end

    %% 2. 문제 풀이 루틴
    subgraph "💻 풀이 루틴 (자동 검증)"
        GitPush["🚀 깃허브 푸시<br/>(코드 커밋)"]:::trigger --> GitTrigger[n8n: 깃허브 트리거]
        GitTrigger -->|필터| CheckCommit{"커밋 메시지에<br/>'BaekjoonHub' 포함?"}
        
        CheckCommit -- 네 --> Parse["⚙️ 데이터 파싱<br/>(제목, 난이도, URL)"]
        Parse -->|동기화| NotionUpdate["🔄 노션: 페이지 업데이트<br/>(상태: 완료, URL)"]:::db
        Parse -->|알림| SlackDone[🎉 슬랙: 성공 알림]:::action
    end

    %% 두 그룹을 위아래로 배치하기 위한 투명 연결선
    NotionCreate ~~~ GitPush
