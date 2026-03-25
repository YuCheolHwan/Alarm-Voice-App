# 📡 Alarm Voice App - API Specification (v1)

---

## 📌 Base URL

```text
https://api.your-domain.com
```

---

## 📌 Common Rules

### 🔐 Authorization

```http
Authorization: Bearer {accessToken}
```

---

### 📦 Response Format

```json
{
  "success": true,
  "data": {},
  "error": null
}
```

---

### ❌ Error Format

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERROR_CODE",
    "message": "error message"
  }
}
```

---

# 1. 🔐 Auth

---

## 1.1 Sign Up

**POST** `/auth/signup`

### Request

```json
{
  "email": "test@test.com",
  "password": "1234",
  "nickname": "cheolhwan"
}
```

### Response

```json
{
  "success": true,
  "data": {
    "userId": 1,
    "accessToken": "jwt_token"
  },
  "error": null
}
```

---

## 1.2 Login

**POST** `/auth/login`

### Request

```json
{
  "email": "test@test.com",
  "password": "1234"
}
```

### Response

```json
{
  "success": true,
  "data": {
    "userId": 1,
    "accessToken": "jwt_token"
  },
  "error": null
}
```

---

## 1.3 Get My Info

**GET** `/users/me`

### Response

```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "test@test.com",
    "nickname": "cheolhwan"
  },
  "error": null
}
```

---

# 2. 📡 Device (FCM)

---

## 2.1 Register FCM Token

**POST** `/devices`

### Request

```json
{
  "fcmToken": "xxxx",
  "deviceType": "ANDROID"
}
```

### Response

```json
{
  "success": true,
  "data": null,
  "error": null
}
```

---

## 2.2 Delete FCM Token

**DELETE** `/devices`

### Request

```json
{
  "fcmToken": "xxxx"
}
```

### Response

```json
{
  "success": true,
  "data": null,
  "error": null
}
```

---

# 3. 👥 Friends

---

## 3.1 Search Users

**GET** `/users/search?keyword={keyword}`

### Response

```json
{
  "success": true,
  "data": [
    {
      "id": 2,
      "nickname": "friendA"
    }
  ],
  "error": null
}
```

---

## 3.2 Add Friend

**POST** `/friends`

### Request

```json
{
  "friendId": 2
}
```

### Response

```json
{
  "success": true,
  "data": null,
  "error": null
}
```

---

## 3.3 Get Friends

**GET** `/friends`

### Response

```json
{
  "success": true,
  "data": [
    {
      "friendId": 2,
      "nickname": "friendA",
      "alarmPermission": "ALLOW"
    }
  ],
  "error": null
}
```

---

## 3.4 Delete Friend

**DELETE** `/friends/{friendId}`

### Response

```json
{
  "success": true,
  "data": null,
  "error": null
}
```

---

## 3.5 Update Alarm Permission

**PATCH** `/friends/{friendId}/permission`

### Request

```json
{
  "alarmPermission": "ALLOW"
}
```

### Response

```json
{
  "success": true,
  "data": null,
  "error": null
}
```

### Enum

```text
ALLOW
REQUEST
BLOCK
```

---

# 4. 🎵 Audio

---

## 4.1 Upload Audio

**POST** `/upload/audio`

### Content-Type

```text
multipart/form-data
```

### Request

```text
file: audio.m4a
```

### Response

```json
{
  "success": true,
  "data": {
    "audioUrl": "https://cdn.xxx/audio/123.m4a",
    "duration": 10
  },
  "error": null
}
```

---

# 5. ⏰ Alarm

---

## 5.1 Create Alarm

**POST** `/alarms`

### Request

```json
{
  "receiverId": 2,
  "audioUrl": "https://cdn.xxx/audio/123.m4a",
  "alarmTime": "2026-03-21T08:00:00Z",
  "repeatType": "NONE",
  "delaySeconds": 1
}
```

### Response

```json
{
  "success": true,
  "data": {
    "alarmId": 101
  },
  "error": null
}
```

---

## 5.2 Get Sent Alarms

**GET** `/alarms/sent`

### Response

```json
{
  "success": true,
  "data": [
    {
      "alarmId": 101,
      "receiverId": 2,
      "alarmTime": "2026-03-21T08:00:00Z",
      "status": "SCHEDULED"
    }
  ],
  "error": null
}
```

---

## 5.3 Get Received Alarms

**GET** `/alarms/received`

### Response

```json
{
  "success": true,
  "data": [
    {
      "alarmId": 101,
      "senderId": 1,
      "audioUrl": "https://...",
      "alarmTime": "2026-03-21T08:00:00Z",
      "status": "SCHEDULED"
    }
  ],
  "error": null
}
```

---

## 5.4 Get Alarm Detail

**GET** `/alarms/{alarmId}`

### Response

```json
{
  "success": true,
  "data": {
    "alarmId": 101,
    "senderId": 1,
    "receiverId": 2,
    "audioUrl": "https://...",
    "alarmTime": "2026-03-21T08:00:00Z",
    "status": "SCHEDULED"
  },
  "error": null
}
```

---

## 5.5 Update Alarm

**PATCH** `/alarms/{alarmId}`

### Request

```json
{
  "alarmTime": "2026-03-21T09:00:00Z"
}
```

### Response

```json
{
  "success": true,
  "data": null,
  "error": null
}
```

---

## 5.6 Delete Alarm

**DELETE** `/alarms/{alarmId}`

### Response

```json
{
  "success": true,
  "data": null,
  "error": null
}
```

---

# 6. 🔔 FCM Payload

---

## Alarm Create Push

```json
{
  "type": "ALARM_CREATE",
  "alarmId": 101,
  "alarmTime": "2026-03-21T08:00:00Z",
  "audioUrl": "https://..."
}
```

---

# 7. 🧠 Enums

---

## Alarm Status

```text
SCHEDULED
TRIGGERED
CANCELLED
```

---

## Repeat Type

```text
NONE
DAILY
CUSTOM
```

---

## Device Type

```text
ANDROID
IOS
```

---
