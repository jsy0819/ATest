erDiagram
    %% Entities (테이블 정의)

    USER {
        BIGINT id PK "사용자 고유 식별자"
        VARCHAR email__sns_id "이메일 / 소셜 ID (Unique)"
        VARCHAR password "해시된 비밀번호"
        VARCHAR name "이름"
        VARCHAR department "소속 부서"
        VARCHAR position "직책"
        BOOLEAN noti_enabled "알림 설정"
    }

    MEETING {
        BIGINT id PK "회의 식별자"
        VARCHAR title "회의 제목"
        BIGINT host_user_id FK "주최자 FK"
        DATETIME started_at "시작시각"
        MEDIUMTEXT summary_text "최종 요약 결과"
        BIGINT calendar_event_id FK "연동 일정 ID (Optional)"
    }

    PARTICIPANT {
        BIGINT id PK "참가 기록 식별자"
        BIGINT meeting_id FK "회의 FK"
        BIGINT user_id FK "참가자 FK"
        VARCHAR speaker_label "화자 구분 라벨"
    }

    SPEECHLOG {
        BIGINT id PK "발언 기록 식별자"
        BIGINT meeting_id FK "회의 FK"
        BIGINT participant_id FK "참가자 FK"
        DECIMAL timestamp "발언 시작 초(ms)"
        TEXT stt_text "변환된 텍스트"
        BOOLEAN is_action_item "액션 아이템 여부"
    }

    HIGHLIGHT {
        BIGINT id PK "하이라이트 식별자"
        BIGINT meeting_id FK "회의 FK"
        VARCHAR keyword "감지된 키워드"
        DECIMAL detected_at "감지 시각(초)"
        TEXT context_text "관련 내용/문맥"
        BOOLEAN is_user_tagged "사용자 수동 태깅 여부"
    }

    TODO {
        BIGINT id PK "할일 식별자"
        BIGINT meeting_id FK "회의 FK"
        VARCHAR content "내용"
        BIGINT responsible_user_id FK "담당자 FK (User.id)"
        DATETIME due_date "마감일"
        VARCHAR status "상태값"
        BIGINT linked_calendar_event_id FK "연동 일정 FK (Optional)"
        VARCHAR external_event_id "외부 캘린더 ID"
    }

    CALENDAREVENT {
        BIGINT id PK "일정 식별자"
        VARCHAR title "일정 제목"
        DATETIME start_time "시작시각"
        BIGINT created_by FK "생성자 FK"
    }

    KEYWORD {
        BIGINT id PK "키워드 식별자"
        VARCHAR value "키워드"
        VARCHAR category "카테고리"
        BIGINT created_by FK "등록자 FK"
        VARCHAR team_or_dept "팀/부서별 키워드"
    }

    DICTIONARY {
        BIGINT id PK "용어 식별자"
        VARCHAR term "용어 (Unique)"
        VARCHAR short_desc "짧은 설명"
        BIGINT registered_by FK "등록자 FK"
        VARCHAR team_or_dept "팀/부서"
    }

    CHATBOTQUERY {
        BIGINT id PK "챗봇 질의 식별자"
        BIGINT user_id FK "사용자 FK"
        BIGINT meeting_id FK "회의 FK (Optional)"
        TEXT query_text "질의 내용"
        TEXT response_text "응답 내용"
    }

    %% Relationships (관계 정의)

    %% 주요 관계: 사용자 - 회의
    USER ||--o{ MEETING : hosts
    USER ||--o{ PARTICIPANT : "participates_in(N:M)"
    USER ||--o{ TODO : "is_responsible_for"
    USER ||--o{ CALENDAREVENT : creates

    %% 주요 관계: 회의 - 세부 기록
    MEETING ||--o{ PARTICIPANT : includes
    MEETING ||--o{ SPEECHLOG : contains
    MEETING ||--o{ HIGHLIGHT : contains
    MEETING ||--o{ TODO : yields
    MEETING }|--o| CALENDAREVENT : "linked_to_event(Optional)"

    %% 연결 테이블 및 로그
    PARTICIPANT ||--o{ SPEECHLOG : "generates_speech"
    TODO }|--o| CALENDAREVENT : "linked_to_todo(Optional)"
    MEETING }|--o{ CHATBOTQUERY : "is_queried_about(Optional)"

    %% 설정 및 지식 관리
    USER ||--o{ KEYWORD : defines
    USER ||--o{ DICTIONARY : registers
    USER ||--o{ CHATBOTQUERY : queries
