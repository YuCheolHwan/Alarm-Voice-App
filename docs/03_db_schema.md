# 🗄️ Alarm Voice App - Database Schema (v1)

---

# 📌 DB Overview

* RDBMS 기준 (MySQL / PostgreSQL)
* 시간은 **UTC 기준 저장**
* 모든 PK는 BIGINT (auto increment)

---

# 1. 👤 users

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    nickname VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🔥 Index

```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_nickname ON users(nickname);
```

---

# 2. 📡 devices (FCM)

```sql
CREATE TABLE devices (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    fcm_token VARCHAR(512) NOT NULL,
    device_type VARCHAR(20) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT fk_devices_user FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## 🔥 Index

```sql
CREATE INDEX idx_devices_user_id ON devices(user_id);
CREATE UNIQUE INDEX idx_devices_token ON devices(fcm_token);
```

---

# 3. 👥 friends

```sql
CREATE TABLE friends (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    friend_id BIGINT NOT NULL,
    alarm_permission VARCHAR(20) DEFAULT 'ALLOW',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT fk_friends_user FOREIGN KEY (user_id) REFERENCES users(id),
    CONSTRAINT fk_friends_friend FOREIGN KEY (friend_id) REFERENCES users(id),
    
    CONSTRAINT uq_friends UNIQUE (user_id, friend_id)
);
```

---

## 🔥 Index

```sql
CREATE INDEX idx_friends_user_id ON friends(user_id);
CREATE INDEX idx_friends_friend_id ON friends(friend_id);
```

---

## ⚠️ 설계 포인트

* (user_id, friend_id) UNIQUE → 중복 방지
* 양방향 관계는 **앱/서버 로직으로 관리**

---

# 4. 🎵 audios

```sql
CREATE TABLE audios (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    file_url TEXT NOT NULL,
    duration INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT fk_audios_user FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## 🔥 Index

```sql
CREATE INDEX idx_audios_user_id ON audios(user_id);
```

---

# 5. ⏰ alarms (핵심 테이블)

```sql
CREATE TABLE alarms (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    sender_id BIGINT NOT NULL,
    receiver_id BIGINT NOT NULL,
    audio_id BIGINT NOT NULL,
    
    alarm_time TIMESTAMP NOT NULL,
    repeat_type VARCHAR(20) DEFAULT 'NONE',
    delay_seconds INT DEFAULT 0,
    
    status VARCHAR(20) DEFAULT 'SCHEDULED',
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT fk_alarms_sender FOREIGN KEY (sender_id) REFERENCES users(id),
    CONSTRAINT fk_alarms_receiver FOREIGN KEY (receiver_id) REFERENCES users(id),
    CONSTRAINT fk_alarms_audio FOREIGN KEY (audio_id) REFERENCES audios(id)
);
```

---

## 🔥 Index (중요)

```sql
CREATE INDEX idx_alarms_receiver_time 
ON alarms(receiver_id, alarm_time);

CREATE INDEX idx_alarms_sender 
ON alarms(sender_id);

CREATE INDEX idx_alarms_status 
ON alarms(status);
```

---

## 💣 핵심 설계 이유

### 1. receiver_id + alarm_time

👉 알람 실행 / 조회 최적화

---

### 2. status 인덱스

👉 예정 알람 빠르게 조회

---

### 3. sender_id

👉 보낸 알람 목록 조회

---

# 6. 🧠 Enum 정의 (DB 기준)

---

## alarm_permission

```text
ALLOW
REQUEST
BLOCK
```

---

## alarm_status

```text
SCHEDULED
TRIGGERED
CANCELLED
```

---

## repeat_type

```text
NONE
DAILY
CUSTOM
```

---

## device_type

```text
ANDROID
IOS
```

---

# 🔥 성능 고려 사항

---

## 1. 알람 조회 쿼리 (핵심)

```sql
SELECT * FROM alarms
WHERE receiver_id = ?
AND alarm_time >= NOW()
AND status = 'SCHEDULED'
ORDER BY alarm_time ASC;
```

👉 idx_alarms_receiver_time 사용됨

---

## 2. 앱 실행 시 동기화

```sql
SELECT * FROM alarms
WHERE receiver_id = ?
AND status = 'SCHEDULED';
```

---

## 3. FCM 전송 대상 조회

```sql
SELECT fcm_token FROM devices
WHERE user_id = ?;
```

---

# ⚠️ 중요 설계 결정

---

## ❗ audio_url 대신 audio_id 사용

👉 정규화 유지
👉 나중에 확장 가능

---

## ❗ 시간은 반드시 UTC

👉 클라이언트 timezone 문제 방지

---

## ❗ 알람 실행은 DB가 아니라 디바이스

👉 DB는 “기록 + 동기화 역할”

---

# 🚀 확장 가능 구조

---

추후 추가 가능:

* alarm_logs (알람 실행 로그)
* notification_history
* blocked_users

---
* FCM + 디바이스 기반 실행 구조

👉 이 설계 그대로 쓰면 실서비스 가능
