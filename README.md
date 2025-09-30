```mermaid
erDiagram
    %% AI 회의록 시스템 ERD (알림 테이블 제외)
    
    %% Entities (테이블 정의 및 상세 정보)

    USER {
        BIGINT id PK "사용자 고유 식별자"
        VARCHAR email__sns_id "이메일 / 소셜 ID (Unique) (VARCHAR(100))"
        VARCHAR password "해시된 비밀번호 (VARCHAR(200))"
        VARCHAR name "이름 (VARCHAR(50))"
        VARCHAR department "소속 부서 (VARCHAR(50))"
        VARCHAR position "직책 (VARCHAR(50))"
        VARCHAR social_type "소셜 유형 (VARCHAR(20))"
        VARCHAR profile_img_url "프로필 이미지 (VARCHAR(255))"
        BOOLEAN noti_enabled "알림 설정"
    }

    MEETING {
        BIGINT id PK "회의 식별자"
        VARCHAR title "회의 제목 (VARCHAR(150))"
        BIGINT host_user_id FK "주최자 FK"
        DATETIME started_at "시작시각"
        DATETIME ended_at "종료시각"
        VARCHAR status "상태(진행/예정 등) (VARCHAR(50))"
        VARCHAR raw_audio_path "오디오 파일 주소 (VARCHAR(255))"
        MEDIUMTEXT stt_raw_text "회의 STT 원본"
        MEDIUMTEXT summary_text "최종 요약 결과"
        MEDIUMTEXT highlight_text "하이라이트 원본"
        BIGINT calendar_event_id FK "연동 일정 ID (Optional)"
    }

    PARTICIPANT {
        BIGINT id PK "참가 기록 식별자"
        BIGINT meeting_id FK "회의 FK"
        BIGINT user_id FK "참가자 FK"
        DATETIME join_time "입장 시각"
        DATETIME leave_time "퇴장 시각"
        VARCHAR speaker_label "화자 구분 라벨 (VARCHAR(30))"
    }

    SPEECHLOG {
        BIGINT id PK "발언 기록 식별자"
        BIGINT meeting_id FK "회의 FK"
        BIGINT participant_id FK "참가자 FK"
        DECIMAL timestamp "발언 시작 초(ms) (DECIMAL(10,3))"
        VARCHAR speaker_label "화자 정보 (VARCHAR(30))"
        TEXT stt_text "변환된 텍스트"
        BOOLEAN is_highlighted "하이라이트 여부"
        BOOLEAN is_action_item "액션 아이템 여부"
        VARCHAR keyword_matched "탐지 키워드 (VARCHAR(100))"
    }

    HIGHLIGHT {
        BIGINT id PK "하이라이트 식별자"
        BIGINT meeting_id FK "회의 FK"
        VARCHAR keyword "감지된 키워드 (VARCHAR(100))"
        DECIMAL detected_at "감지 시각(초) (DECIMAL(10,3))"
        TEXT context_text "관련 내용/문맥"
        BOOLEAN is_user_tagged "사용자 수동 태깅 여부"
    }

    TODO {
        BIGINT id PK "할일 식별자"
        BIGINT meeting_id FK "회의 FK"
        VARCHAR content "내용 (VARCHAR(255))"
        BIGINT responsible_user_id FK "담당자 FK (User.id)"
        DATETIME due_date "마감일"
        VARCHAR status "상태값 (VARCHAR(30))"
        BIGINT linked_calendar_event_id FK "연동 일정 FK (Optional)"
        VARCHAR external_event_id "외부 캘린더 ID (VARCHAR(100))"
    }

    CALENDAREVENT {
        BIGINT id PK "일정 식별자"
        VARCHAR title "일정 제목 (VARCHAR(150))"
        TEXT description "설명"
        DATETIME start_time "시작시각"
        DATETIME end_time "종료시각"
        VARCHAR location "장소 (VARCHAR(100))"
        BIGINT created_by FK "생성자 FK"
    }

    KEYWORD {
        BIGINT id PK "키워드 식별자"
        VARCHAR value "키워드 (VARCHAR(100))"
        VARCHAR category "카테고리 (VARCHAR(50))"
        BOOLEAN is_active "활성 여부"
        BIGINT created_by FK "등록자 FK"
        VARCHAR team_or_dept "팀/부서별 키워드 (VARCHAR(50))"
    }

    DICTIONARY {
        BIGINT id PK "용어 식별자"
        VARCHAR term "용어 (Unique) (VARCHAR(100))"
        VARCHAR short_desc "짧은 설명 (VARCHAR(200))"
        TEXT full_desc "상세 설명"
        BIGINT registered_by FK "등록자 FK"
        VARCHAR team_or_dept "팀/부서 (VARCHAR(50))"
    }

    CHATBOTQUERY {
        BIGINT id PK "챗봇 질의 식별자"
        BIGINT user_id FK "사용자 FK"
        BIGINT meeting_id FK "회의 FK (Optional)"
        TEXT query_text "질의 내용"
        TEXT response_text "응답 내용"
        DATETIME queried_at "질의 시각"
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
