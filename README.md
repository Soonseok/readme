# 관리자 페이지 DB
## 1. admin_accounts – 관리자 계정 테이블
```{sql}
CREATE TABLE admin_accounts (
    admin_id       VARCHAR(50) PRIMARY KEY,
    password       VARCHAR(255) NOT NULL,
    role_id        INT NOT NULL,
    created_at     DATE NOT NULL,
    FOREIGN KEY (role_id) REFERENCES roles(role_id)
);
```
+ `role_id`는 권한 종류 연결(FK)
+ 비밀번호는 암호화 저장 전제

## 2. roles – 권한 종류 테이블
```{sql}
CREATE TABLE roles (
    role_id        INT PRIMARY KEY AUTO_INCREMENT,
    role_name      VARCHAR(50) UNIQUE NOT NULL,
    description    TEXT
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
    notice_id      INT PRIMARY KEY AUTO_INCREMENT,
    title          VARCHAR(200) NOT NULL,
    author_id      VARCHAR(50) NOT NULL,
    is_pinned      CHAR(1) DEFAULT 'N',
    created_at     DATE NOT NULL,
    view_count     INT DEFAULT 0,
    content        TEXT,
    file_path      VARCHAR(255),
    FOREIGN KEY (author_id) REFERENCES admin_accounts(admin_id)
);
```


## 4. reports – 신고 기록 테이블
```{sql}
CREATE TABLE reports (
    report_id         INT PRIMARY KEY AUTO_INCREMENT,
    reporter_id       VARCHAR(50) NOT NULL,
    reported_user_id  VARCHAR(50) NOT NULL,
    reason            VARCHAR(100),
    reported_at       DATETIME NOT NULL,
    detail            TEXT,
    status            ENUM('완료', '보류', '미완료') DEFAULT '미완료'
);
```
+ `reporter_id`와 `reported_user_id`는 유저 테이블의 ID를 참조

## 5. activity_logs – 유저 활동 로그
```{sql}
CREATE TABLE activity_logs (
    log_id        INT PRIMARY KEY AUTO_INCREMENT,
    user_id       VARCHAR(50) NOT NULL,
    action_type   VARCHAR(50), -- 예: login, write_post, delete_comment 등
    action_time   DATETIME NOT NULL,
    target_info   VARCHAR(255), -- 예: 게시글 ID 등
    description   TEXT
);
```


## 6. system_logs – 시스템 로그 테이블
```{sql}
CREATE TABLE system_logs (
    log_id        INT PRIMARY KEY AUTO_INCREMENT,
    log_level     VARCHAR(10),   -- INFO, WARN, ERROR 등
    message       TEXT,
    occurred_at   DATETIME NOT NULL,
    ip_address    VARCHAR(50),
    admin_id      VARCHAR(50),
    stack_trace   TEXT,
    FOREIGN KEY (admin_id) REFERENCES admin_accounts(admin_id)
);
```


## 7. admin_actions – 관리자 행동 기록 테이블
```{sql}
CREATE TABLE admin_actions (
    action_id     INT PRIMARY KEY AUTO_INCREMENT,
    admin_id      VARCHAR(50) NOT NULL,
    action_type   VARCHAR(50),      -- 예: sanction_user, delete_notice
    target_user   VARCHAR(50),      -- 제재 대상자 등
    action_time   DATETIME NOT NULL,
    detail        TEXT,
    FOREIGN KEY (admin_id) REFERENCES admin_accounts(admin_id)
);
```


## 8. sanctions – 유저 제재 이력 테이블
```{sql}
CREATE TABLE sanctions (
    sanction_id     INT PRIMARY KEY AUTO_INCREMENT,
    user_id         VARCHAR(50) NOT NULL,
    sanction_type   VARCHAR(50),         -- 예: write_ban, login_block
    start_date      DATETIME NOT NULL,
    end_date        DATETIME,
    reason          TEXT,
    issued_by       VARCHAR(50),         -- 관리자 ID
    created_at      DATETIME NOT NULL,
    FOREIGN KEY (issued_by) REFERENCES admin_accounts(admin_id)
);
```
+ 검색을 위해 신고 처리와 별개로 모든 제재 내역 저장
