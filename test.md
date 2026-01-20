graph TD
    classDef trigger fill:#FFD1DC,stroke:#333,stroke-width:2px,rx:5,ry:5
    classDef action fill:#AEC6CF,stroke:#333,stroke-width:2px,rx:5,ry:5
    classDef db fill:#B0E57C,stroke:#333,stroke-width:2px,rx:5,ry:5
    classDef user fill:#FDFD96,stroke:#333,stroke-width:4px

    subgraph "☀️ 아침 루틴 (일일 문제 추천)"
        Cron["📅 <b>스케줄 트리거</b><br/>(매일 오전 09:00)"]:::trigger --> API["⚡ <b>HTTP 요청</b><br/>(Solved.ac API)"]
        API -->|필터: 실버 3~5| SlackBot["💬 <b>슬랙 봇</b><br/>(문제 추천 메시지)"]:::action
        
        User((👤 사용자)):::user -->|'수락' 클릭| Webhook["🪝 <b>웹훅 수신</b>"]:::trigger
        Webhook -->|분기: 액션 ID| Logic{"<b>승인(Approve)</b><br/>버튼인가?"}:::action
        Logic -- 네 --> NotionCreate["📝 <b>노션 페이지 생성</b><br/>(상태: 할 일)"]:::db
    end

    subgraph "💻 풀이 루틴 (자동 검증)"
        GitPush["🚀 <b>깃허브 푸시</b><br/>(코드 커밋)"]:::trigger --> GitTrigger["⚙️ <b>n8n 깃허브 트리거</b>"]:::action
        GitTrigger -->|필터| CheckCommit{"커밋 메시지에<br/>'<b>BaekjoonHub</b>' 포함?"}:::action
        
        CheckCommit -- 네 --> Parse["⚙️ <b>데이터 파싱</b><br/>(제목, 난이도, URL)"]:::action
        Parse -->|동기화| NotionUpdate["🔄 <b>노션 업데이트</b><br/>(상태: 완료, URL)"]:::db
        Parse -->|알림| SlackDone["🎉 <b>슬랙 알림</b><br/>(성공 메시지)"]:::action
    end

    NotionCreate ~~~ GitPush
