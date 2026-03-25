# 📱 Alarm Voice App - Product Specification (v1)

---

# 1. 📌 서비스 개요

## 🎯 서비스 목적

사용자가 친구에게 **음성 기반 알람을 전송**하여
일반 알람보다 더 강력하게 깨워주는 커뮤니케이션형 알람 서비스

---

## 💡 핵심 컨셉

* 기존 알람: 개인 중심
* 본 서비스: **사람 ↔ 사람 기반 알람**

👉 감정과 관계를 활용한 알람 경험 제공

---

# 2. 👤 사용자 시나리오 (User Flow)

```text
회원가입 → 로그인 → 친구 추가 → 알람 수신 허용 설정 → 음성 녹음 → 알람 전송 → 상대방 수신 → 알람 실행
```

---

# 3. 📱 화면 구성 (Screen Structure)

---

## 🏠 3.1 홈 화면 (Home)

### 기능

* 내가 받은 알람 개수 표시
* 가장 가까운 알람 정보 표시
* 사용자 프로필 간략 정보 표시

---

## ⏰ 3.2 알람 관리 화면 (Alarm List)

### 기능

* 전체 알람 리스트 조회
* 알람 상태 표시 (예정 / 완료 / 취소)
* 스와이프 액션

  * 수정
  * 삭제

---

## 📤 3.3 알람 전송 화면 (Send Alarm)

### 기능

* 친구 선택
* 알람 시간 설정 (필수)
* 음성 녹음 (n초 이하 제한)
* 반복 여부 설정
* (옵션) 재생 딜레이 설정
* 알람 전송

---

## 👥 3.4 친구 관리 화면 (Friends)

### 기능

* 친구 목록 조회
* 친구 추가 (ID / 닉네임 기반)
* 친구 삭제 (스와이프)
* 알람 수신 허용 설정

  * 항상 허용
  * 요청 후 수락
  * 차단

---

## ⚙️ 3.5 설정 화면 (Settings)

### 기능

* 로그아웃
* 계정 정보
* 약관 / 개인정보 처리방침

---

# 4. 🧱 도메인 모델 (Domain Model)

---

## 👤 User

* id (PK)
* email
* password (또는 socialId)
* nickname (unique)
* createdAt

---

## 👥 Friend

* id (PK)
* userId
* friendId
* alarmPermission (ALLOW / REQUEST / BLOCK)
* createdAt

---

## ⏰ Alarm

* id (PK)
* senderId
* receiverId
* audioUrl
* alarmTime (UTC 기준)
* repeatType (NONE / DAILY / CUSTOM)
* delaySeconds
* status (SCHEDULED / TRIGGERED / CANCELLED)
* createdAt

---

## 🎵 Audio

* id (PK)
* userId
* fileUrl
* duration
* createdAt

---

## 📡 Device (FCM)

* id (PK)
* userId
* fcmToken
* deviceType (ANDROID / IOS)
* createdAt

---

# 5. 🔐 핵심 기능 요구사항

---

## 5.1 인증 (Authentication)

* 이메일 기반 로그인
* (확장) 소셜 로그인 지원
* JWT 기반 인증

---

## 5.2 친구 관리

* ID / 닉네임 검색 기반 추가
* 중복 추가 방지
* 자기 자신 추가 금지
* 알람 수신 권한 설정 필수

---

## 5.3 알람 전송

* 알람 시간 선택 필수
* 음성 파일 업로드 필요
* 수신자 권한 확인 후 전송

---

## 5.4 알람 수신

* FCM push를 통한 알람 전달
* 디바이스 로컬 알람 등록

---

## 5.5 알람 실행

* 지정 시간에 자동 실행
* 음성 반복 재생 (Android 기준)

---

## 5.6 동기화 (Sync)

* 앱 실행 시 서버 기준으로 알람 재동기화
* FCM 누락 대비

---

# 6. 📡 시스템 동작 구조

---

## 📤 알람 전송 흐름

```text
1. 사용자 → 음성 녹음
2. 서버 → 음성 파일 업로드
3. 알람 생성 API 호출
4. 서버 DB 저장
5. 서버 → 수신자에게 FCM push 전송
```

---

## 📥 알람 수신 흐름

```text
[앱 비활성 상태]
FCM 수신 → AlarmManager 등록

[앱 활성 상태]
서버 알람 조회 → 로컬 알람 등록
```

---

# 7. 🔔 알람 처리 구조 (중요)

---

## 핵심 구조

```text
FCM (트리거) → AlarmManager (실행)
```

👉 FCM은 알람 실행이 아닌 “등록 트리거”

---

## 보완 구조

```text
FCM 수신 + 앱 실행 시 동기화
```

👉 안정성 확보

---

# 8. ⚠️ 플랫폼별 제약

---

## 🤖 Android

* AlarmManager 사용 가능
* 백그라운드 알람 실행 가능
* 반복 재생 가능

---

## 🍎 iOS

* Notification 기반 알람
* 사운드 30초 제한
* 반복 재생 제한

👉 완전한 알람 기능 구현 불가

---

# 9. 🚨 예외 및 제약 조건

---

* 알람 시간 과거 설정 금지
* 친구 권한 없으면 알람 전송 불가
* 음성 길이 제한 필요 (ex: 30초)
* 알람 중복 등록 방지 (alarmId 기준)
* 앱 최초 실행 시 FCM 토큰 등록 필수

---

# 10. 📦 MVP 범위

---

## 포함 기능

* 회원가입 / 로그인
* 친구 추가 / 삭제
* 음성 녹음 및 업로드
* 알람 생성
* Android 알람 실행

---

## 제외 (추후)

* 소셜 로그인
* 반복 고급 옵션
* 푸시 커스터마이징
* 통계 / 히스토리

---

# 11. 🧠 기술 스택

---

## Mobile

* Android: Kotlin, MVVM, Multi-module
* iOS: SwiftUI

---

## Backend

* Node.js (Express)
* RESTful API
* JWT 인증

---

## Infra

* Firebase Cloud Messaging (FCM)
* (옵션) AWS S3 (파일 저장)

---

# 12. 📂 Repository 구조

```bash
alarm-app/
 ├── android/
 ├── ios/
 ├── server/
 └── docs/
```

---
