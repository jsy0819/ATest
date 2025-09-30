erDiagram
    %% Entities (테이블 정의)
    USER {
        BIGINT id PK "사용자 고유 식별자"
        VARCHAR email__sns_id "이메일 / 소셜 ID"
        VARCHAR password "해시된 비밀번호"
        VARCHAR name "이름"
        VARCHAR department "소속 부서"
        VARCHAR position "직책"
        VARCHAR social_type "소셜 유형"
        VARCHAR profile_img_url "프로필 이미지"
        BOOLEAN noti_enabled "알림 활성 여부"
    }

    MEETING {
        BIGINT id PK "회의 식별자"
        VARCHAR title "회의 제목"
        BIGINT host_user_id FK "주최자 FK"
        DATETIME started_at "시작시각"
        DATETIME ended_at "종료시각"
        VARCHAR status "상태(진행/예정 등)"
        VARCHAR raw_audio_path "오디오 파일 주소"
        MEDIUMTEXT stt_raw_text "회의 STT 원본"
        MEDIUMTEXT summary_text "요약 결과"
        MEDIUMTEXT highlight_text "하이라이트 원본"
        BIGINT calendar_event_id FK "연동 일정 ID"
    }

    PARTICIPANT {
        BIGINT id PK "참가 기록 식별자"
        BIGINT meeting_id FK "회의 FK"
        BIGINT user_id FK "참가자 FK"
        DATETIME join_time "입장 시각"
        DATETIME leave_time "퇴장 시각"
        VARCHAR speaker_label "화자 구분 라벨"
    }

    SPEECHLOG {
        BIGINT id PK "발언 기록 식별자"
        BIGINT meeting_id FK "회의 FK"
        BIGINT participant_id FK "참가자 FK"
        DECIMAL timestamp "발언 시작 초(ms)"
        VARCHAR speaker_label "화자 정보"
        TEXT stt_text "변환된 텍스트"
        BOOLEAN is_highlighted "하이라이트 여부"
        BOOLEAN is_action_item "액션 아이템 여부"
        VARCHAR keyword_matched "탐지 키워드"
    }

    HIGHLIGHT {
        BIGINT id PK "하이라이트 식별자"
        BIGINT meeting_id FK "회의 FK"
        VARCHAR keyword "감지된 키워드"
        DECIMAL detected_at "감지 시각(초)"
        TEXT context_text "관련 내용/문맥"
    }

    TODO {
        BIGINT id PK "할일 식별자"
        BIGINT meeting_id FK "회의 FK"
        VARCHAR content "내용"
        BIGINT responsible_participant_id FK "담당자 FK (참가 기록)"
        DATETIME due_date "마감일"
        VARCHAR status "상태값"
        BIGINT linked_calendar_event_id FK "연동 일정 FK"
    }

    CALENDAREVENT {
        BIGINT id PK "일정 식별자"
        VARCHAR title "일정 제목"
        TEXT description "설명"
        DATETIME start_time "시작시각"
        DATETIME end_time "종료시각"
        VARCHAR location "장소"
        BIGINT created_by FK "생성자 FK"
    }

    KEYWORD {
        BIGINT id PK "키워드 식별자"
        VARCHAR value "키워드"
        VARCHAR category "카테고리"
        BOOLEAN is_active "활성 여부"
        BIGINT created_by FK "등록자 FK"
    }

    DICTIONARY {
        BIGINT id PK "용어 식별자"
        VARCHAR term "용어"
        VARCHAR short_desc "짧은 설명"
        TEXT full_desc "상세 설명"
        BIGINT registered_by FK "등록자 FK"
        VARCHAR team_or_dept "팀/부서"
    }

    CHATBOTQUERY {
        BIGINT id PK "챗봇 질의 식별자"
        BIGINT user_id FK "사용자 FK"
        BIGINT meeting_id FK "회의 FK"
        TEXT query_text "질의 내용"
        TEXT response_text "응답 내용"
        DATETIME queried_at "질의 시각"
    }

    NOTIFICATION {
        BIGINT id PK "알림 식별자"
        BIGINT user_id FK "사용자 FK"
        BIGINT meeting_id FK "회의 FK"
        VARCHAR type "알림 종류"
        BOOLEAN is_read "읽음 여부"
        DATETIME created_at "생성 시각"
    }

    %% Relationships (관계 정의)

    %% User 관련
    USER ||--o{ MEETING : hosts
    USER ||--o{ PARTICIPANT : "participates_in(N:M)"
    USER ||--o{ CALENDAREVENT : creates
    USER ||--o{ KEYWORD : creates
    USER ||--o{ DICTIONARY : registers
    USER ||--o{ CHATBOTQUERY : queries
    USER ||--o{ NOTIFICATION : receives

    %% Meeting 관련
    MEETING ||--o{ PARTICIPANT : includes
    MEETING ||--o{ SPEECHLOG : contains
    MEETING ||--o{ HIGHLIGHT : contains
    MEETING ||--o{ TODO : yields
    MEETING }|--o| CALENDAREVENT : "linked_to(Optional)"

    %% Participant 관련 (회의별 참가 기록)
    PARTICIPANT ||--o{ SPEECHLOG : "generates_speech"
    PARTICIPANT ||--o{ TODO : "is_responsible_for"

    %% ToDo 관련
    TODO }|--o| CALENDAREVENT : "linked_to_event(Optional)"

    %% 기타
    MEETING }|--o{ CHATBOTQUERY : "is_queried_about(Optional)"
    MEETING }|--o{ NOTIFICATION : "related_to(Optional)"
