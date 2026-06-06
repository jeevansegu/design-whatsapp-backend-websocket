# WhatsApp Backend MVP - System Design

Version: 1.0

Status: Design Approved

---

# 1. System Overview

## Objective

Design a WhatsApp-style messaging backend supporting:

* User Registration
* User Login
* JWT Authentication
* One-to-One Messaging
* Message Persistence
* Message History
* Online Presence
* Last Seen
* Offline Message Synchronization
* Single Tick
* Double Tick
* Blue Tick

The system is designed for learning distributed messaging fundamentals while remaining simple enough for implementation by a small team.

---

# 2. High Level Architecture

```text
┌─────────────────────┐
│    Browser Client   │
│                     │
│ HTML + CSS + JS     │
└──────────┬──────────┘
           │
           │ HTTP / WebSocket
           │
┌──────────▼──────────┐
│      FastAPI        │
│                     │
│ Auth Routes         │
│ Message Routes      │
│ Presence Routes     │
│ WebSocket Endpoint  │
│                     │
│ Auth Service        │
│ Message Service     │
│ Presence Service    │
│ Sync Service        │
│ Connection Manager  │
└───────┬─────┬───────┘
        │     │
        │     │
        ▼     ▼

 ┌──────────┐  ┌─────────┐
 │PostgreSQL│  │  Redis  │
 └──────────┘  └─────────┘
```

---

# 3. Component Responsibilities

## 3.1 Auth Service

### Purpose

Responsible for user authentication and authorization.

### Responsibilities

* Register user
* Login user
* Hash passwords
* Verify passwords
* Generate JWT
* Validate JWT

### Dependencies

* UserRepository
* JWT Library
* Password Hashing Library

---

## 3.2 Message Service

### Purpose

Handles message lifecycle.

### Responsibilities

* Create messages
* Persist messages
* Retrieve conversations
* Update statuses
* Validate state transitions

### Dependencies

* MessageRepository
* UserRepository
* ConnectionManager

---

## 3.3 Presence Service

### Purpose

Tracks user activity.

### Responsibilities

* Online tracking
* Offline tracking
* Last seen updates
* Presence queries

### Dependencies

* Redis
* UserRepository

---

## 3.4 Sync Service

### Purpose

Delivers messages missed during offline periods.

### Responsibilities

* Fetch pending messages
* Deliver messages after reconnect
* Update delivery state

### Dependencies

* MessageRepository
* ConnectionManager

---

## 3.5 Connection Manager

### Purpose

Maintains active WebSocket connections.

### Responsibilities

* Track connected users
* Store WebSocket objects
* Route messages
* Route delivery acknowledgements

### Internal Storage

```python
active_connections = {
    user_id: websocket
}
```

---

# 4. Message Lifecycle Design

## State Machine

```text
SENT
  |
  v
DELIVERED
  |
  v
READ
```

---

## Valid Transitions

```text
SENT -> DELIVERED

DELIVERED -> READ
```

---

## Invalid Transitions

```text
READ -> DELIVERED

READ -> SENT

DELIVERED -> SENT
```

---

## Single Tick Protocol

### Flow

1. User sends message.
2. Backend validates request.
3. Backend stores message.
4. Status becomes SENT.
5. Sender receives acknowledgment.

### Result

```json
{
  "messageId": 101,
  "status": "SENT"
}
```

---

## Double Tick Protocol

### Flow

1. Recipient receives message.
2. Recipient client sends DELIVERED acknowledgement.
3. Status updated to DELIVERED.
4. Sender notified.

### Result

```json
{
  "messageId": 101,
  "status": "DELIVERED"
}
```

---

## Blue Tick Protocol

### Flow

1. Recipient opens conversation.
2. Recipient client sends READ acknowledgement.
3. Status updated to READ.
4. Sender notified.

### Result

```json
{
  "messageId": 101,
  "status": "READ"
}
```

---

# 5. Connection Lifecycle

## User Login

```text
Browser
   |
   v
POST /login
   |
   v
JWT Generated
   |
   v
Open WebSocket
   |
   v
ConnectionManager.connect()
   |
   v
PresenceService.set_online()
```

---

## User Logout

```text
Browser Disconnect
         |
         v
WebSocket Closed
         |
         v
ConnectionManager.disconnect()
         |
         v
PresenceService.set_offline()
         |
         v
Update last_seen
```

---

# 6. Database Design

## users Table

```sql
CREATE TABLE users(
    id BIGSERIAL PRIMARY KEY,

    username VARCHAR(50)
        UNIQUE NOT NULL,

    password_hash TEXT
        NOT NULL,

    last_seen TIMESTAMP,

    created_at TIMESTAMP
        DEFAULT NOW()
);
```

---

## messages Table

```sql
CREATE TABLE messages(
    id BIGSERIAL PRIMARY KEY,

    sender_id BIGINT
        REFERENCES users(id),

    receiver_id BIGINT
        REFERENCES users(id),

    content TEXT
        NOT NULL,

    status VARCHAR(20)
        NOT NULL,

    created_at TIMESTAMP
        DEFAULT NOW(),

    delivered_at TIMESTAMP,

    read_at TIMESTAMP
);
```

---

# 7. Database Index Strategy

## Conversation Lookup

```sql
CREATE INDEX idx_messages_users
ON messages(sender_id, receiver_id);
```

Purpose:

Fast conversation retrieval.

---

## Pending Messages

```sql
CREATE INDEX idx_pending_messages
ON messages(receiver_id, status);
```

Purpose:

Fast offline sync.

---

## Username Lookup

```sql
CREATE UNIQUE INDEX idx_username
ON users(username);
```

Purpose:

Fast login.

---

# 8. Redis Design

## Online Presence

Key:

```text
online:user:{user_id}
```

Value:

```text
true
```

Example:

```text
online:user:5 = true
```

---

## Future Expansion

Potential future keys:

```text
typing:user:5

socket:user:5
```

Not required for MVP.

---

# 9. API Design

---

## Register

### POST /api/auth/register

Request

```json
{
  "username": "jeevan",
  "password": "password123"
}
```

Response

```json
{
  "success": true
}
```

---

## Login

### POST /api/auth/login

Request

```json
{
  "username": "jeevan",
  "password": "password123"
}
```

Response

```json
{
  "token": "jwt-token"
}
```

---

## Get Users

### GET /api/users

Response

```json
[
  {
    "id": 1,
    "username": "jeevan",
    "online": true
  }
]
```

---

## Conversation History

### GET /api/messages/{user_id}

Response

```json
[
  {
    "id": 10,
    "content": "hello",
    "status": "READ"
  }
]
```

---

## Presence

### GET /api/presence/{user_id}

Response

```json
{
  "online": false,
  "lastSeen": "2026-06-06T10:30:00"
}
```

---

# 10. WebSocket Design

## Endpoint

```text
/ws
```

---

## Authentication Event

```json
{
  "type": "AUTH",
  "token": "jwt"
}
```

---

## Send Message Event

```json
{
  "type": "SEND_MESSAGE",
  "receiverId": 2,
  "content": "Hello"
}
```

---

## Incoming Message Event

```json
{
  "type": "MESSAGE",
  "messageId": 100,
  "senderId": 1,
  "content": "Hello"
}
```

---

## Delivered Event

```json
{
  "type": "DELIVERED",
  "messageId": 100
}
```

---

## Read Event

```json
{
  "type": "READ",
  "messageId": 100
}
```

---

## Status Update Event

```json
{
  "type": "STATUS_UPDATE",
  "messageId": 100,
  "status": "READ"
}
```

---

# 11. Offline Synchronization Design

## Reconnect Flow

```text
User Connects
      |
      v
WebSocket Auth
      |
      v
SyncService Invoked
      |
      v
Query Pending Messages
      |
      v
Deliver Messages
      |
      v
Update Status
```

---

## Query

```sql
SELECT *
FROM messages
WHERE receiver_id = ?
AND status = 'SENT'
ORDER BY created_at ASC;
```

---

# 12. Security Design

## Password Storage

Algorithm:

```text
bcrypt
```

Never store plaintext passwords.

---

## Authentication

JWT Authentication

Required for:

* Message APIs
* Presence APIs
* WebSocket Authentication

---

## Authorization

Users may:

* View their own conversations
* Send messages as themselves

Users may not:

* Access other users' conversations

---

# 13. Logging Strategy

## Authentication Logs

```text
User Registered

User Login Success

User Login Failed
```

---

## Messaging Logs

```text
Message Created

Message Delivered

Message Read
```

---

## Presence Logs

```text
User Connected

User Disconnected
```

---

# 14. Testing Strategy

## Unit Tests

Auth Service

Message Service

Presence Service

Sync Service

---

## Integration Tests

PostgreSQL Integration

Redis Integration

WebSocket Integration

---

## End-to-End Tests

Scenario 1:

User A sends message.

User B receives.

---

Scenario 2:

User B offline.

User A sends message.

User B reconnects.

Message synchronized.

---

Scenario 3:

Message delivered.

Message read.

Status updates propagated.

---

# 15. Future Scalability

Future improvements intentionally excluded from MVP:

* Redis Pub/Sub
* Kafka
* Multiple FastAPI instances
* Load Balancer
* Group Chats
* End-to-End Encryption
* Push Notifications

Current design is optimized for simplicity, learning, and interview discussions.

END OF DESIGN