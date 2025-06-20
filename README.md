# 관리자 페이지 DB
## 1. admin_accounts – 관리자 계정 테이블
```{sql}
CREATE TABLE admin_accounts (
    admin_accounts_id       VARCHAR(50) PRIMARY KEY,
    admin_accounts_password       VARCHAR(255) NOT NULL,
    role_id        INT NOT NULL,
    admin_accounts_created_at     DATE NOT NULL,
    FOREIGN KEY (roles_id) REFERENCES roles(role_id)
);
```
+ `role_id`는 권한 종류 연결(FK)
+ 비밀번호는 암호화 저장 전제

## 2. roles – 권한 종류 테이블
```{sql}
CREATE TABLE roles (
    roles_id        INT PRIMARY KEY AUTO_INCREMENT,
    roles_name      VARCHAR(50) UNIQUE NOT NULL,
    roles_description    VARCHAR(250)
);
```
+ 예시
```{sql}
INSERT INTO roles (role_name, description) VALUES
('일반유저', '게시글/댓글 작성 및 열람, 자신이 작성한 글/댓글 삭제, 타인의 글/댓글 신고'),
('관리자', '일반 로그 열람, 공지 작성 및 수정, 신고 처리, 유저 정보 수정'),
('운영자', '전체 로그 열람, 관리자 계정 관리, 강제 탈퇴, 권한 변경');
```


## 3. notices – 공지사항 테이블
```{sql}
CREATE TABLE notices (
    notices_id      INT PRIMARY KEY AUTO_INCREMENT,
    notices_title          VARCHAR(200) NOT NULL,
    notices_author_id      VARCHAR(50) NOT NULL,
    notices_is_pinned      CHAR(1) DEFAULT 'N',
    notices_created_at     DATE NOT NULL,
    notices_view_count     INT DEFAULT 0,
    notices_content        VARCHAR(2000),
    notices_file_path      VARCHAR(255),
    FOREIGN KEY (notices_author_id) REFERENCES admin_accounts(admin_accounts_id)
);
```


## 4. reports – 신고 기록 테이블
```{sql}
CREATE TABLE reports (
    reports_id         INT PRIMARY KEY AUTO_INCREMENT,
    reports_reporter_id       VARCHAR(50) NOT NULL,
    reports_reported_user_id  VARCHAR(50) NOT NULL,
    reports_reason            VARCHAR(100),
    reports_reported_at       DATETIME NOT NULL,
    reports_detail            VARCHAR(2000),
    reports_status            ENUM('완료', '보류', '미완료') DEFAULT '미완료'
);
```
+ `reporter_id`와 `reported_user_id`는 유저 테이블의 ID를 참조

## 5. activity_logs – 유저 활동 로그
```{sql}
CREATE TABLE activity_logs (
    activity_logs_id        INT PRIMARY KEY AUTO_INCREMENT,
    user_id       VARCHAR(50) NOT NULL,
    activity_logs_action_type   VARCHAR(50), -- 예: login, write_post, delete_comment 등
    activity_logs_action_time   DATETIME NOT NULL,
    activity_logs_target_info   VARCHAR(255), -- 예: 게시글 ID 등
    activity_logs_description   VARCHAR(250)
);
```


## 6. system_logs – 시스템 로그 테이블
```{sql}
CREATE TABLE system_logs (
    system_logs_id        INT PRIMARY KEY AUTO_INCREMENT,
    system_logs_level     VARCHAR(10),   -- INFO, WARN, ERROR 등
    system_logs_message       VARCHAR(250),
    system_logs_occurred_at   DATETIME NOT NULL,
    system_logs_admin_id      VARCHAR(50),
    system_logs_stack_trace   VARCHAR(2000),   -- 애매하면 blob
    FOREIGN KEY (system_logs_admin_id) REFERENCES admin_accounts(admin_accounts_id)
);
```


## 7. admin_actions – 관리자 행동 기록 테이블
```{sql}
CREATE TABLE admin_actions (
    admin_actions_id     INT PRIMARY KEY AUTO_INCREMENT,
    admin_actions_id      VARCHAR(50) NOT NULL,
    admin_actions_type   VARCHAR(50),      -- 예: sanction_user, delete_notice
    admin_actions_target_user   VARCHAR(50),      -- 행동 대상자 등
    admin_actions_time   DATETIME NOT NULL,
    admin_actions_detail        VARCHAR(250),
    FOREIGN KEY (admin_actions_id) REFERENCES admin_accounts(admin_accounts_id)
);
```


## 8. sanctions – 유저 제재 이력 테이블
```{sql}
CREATE TABLE sanctions (
    sanctions_id     INT PRIMARY KEY AUTO_INCREMENT,
    user_id         VARCHAR(50) NOT NULL,
    sanctions_type   VARCHAR(50),         -- 예: write_ban, login_block
    sanctions_start_date      DATETIME NOT NULL,
    sanctions_end_date        DATETIME,
    sanctions_reason          VARCHAR(2000),
    sanctions_issued_by       VARCHAR(50),         -- 관리자 ID
    sanctions_created_at      DATETIME NOT NULL,
    FOREIGN KEY (sanctions_issued_by) REFERENCES admin_accounts(admin_accounts_id)
);
```
+ 검색을 위해 신고 처리와 별개로 모든 제재 내역 저장
