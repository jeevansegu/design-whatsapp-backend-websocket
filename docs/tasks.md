# WhatsApp Backend MVP - Implementation Plan

Version: 1.0

---

# Project Structure

```text
app/

├── main.py

├── api/
│   ├── auth_routes.py
│   ├── message_routes.py
│   ├── presence_routes.py
│   └── websocket_routes.py

├── services/
│   ├── auth_service.py
│   ├── message_service.py
│   ├── presence_service.py
│   └── sync_service.py

├── repositories/
│   ├── user_repository.py
│   └── message_repository.py

├── websocket/
│   └── connection_manager.py

├── models/
│   ├── user.py
│   └── message.py

├── schemas/
│   ├── auth.py
│   └── message.py

├── database/
│   └── connection.py

├── redis/
│   └── redis_client.py

frontend/

├── login.html
├── register.html
├── chat.html

├── js/
│   ├── auth.js
│   ├── websocket.js
│   └── chat.js

└── css/
    └── styles.css
```

---

# Sprint 1 - Foundation

Goal:
Database, Models, Repositories, Infrastructure

---

## TASK-001

Owner: Jeevan

File:
database/connection.py

Purpose:
Central PostgreSQL connection management.

Responsibilities:

* Create SQLAlchemy engine
* Create session factory
* Provide database sessions

Functions:

### get_db()

Returns active database session.

Used by:
Repositories

### initialize_database()

Creates tables during startup.

---

## TASK-002

Owner: Pramod

File:
redis/redis_client.py

Purpose:
Redis abstraction layer.

Responsibilities:

* Redis connectivity
* Presence cache operations

Functions:

### set_online(user_id)

Mark user online.

### remove_online(user_id)

Mark user offline.

### is_online(user_id)

Check online status.

### get_online_users()

Return active users.

---

## TASK-003

Owner: Jeevan

File:
models/user.py

Purpose:
User database model.

Responsibilities:

* Define users table

Fields:

* id
* username
* password_hash
* last_seen
* created_at

---

## TASK-004

Owner: Pramod

File:
models/message.py

Purpose:
Message database model.

Responsibilities:

* Define messages table

Fields:

* id
* sender_id
* receiver_id
* content
* status
* created_at
* delivered_at
* read_at

---

## TASK-005

Owner: Jeevan

File:
repositories/user_repository.py

Purpose:
User database operations.

Functions:

### create_user()

Insert new user.

### get_by_username()

Lookup user.

### get_by_id()

Fetch user.

### update_last_seen()

Update timestamp.

### get_all_users()

Return user list.

---

## TASK-006

Owner: Pramod

File:
repositories/message_repository.py

Purpose:
Message persistence layer.

Functions:

### create_message()

Store message.

### get_conversation()

Fetch conversation history.

### get_pending_messages()

Fetch offline messages.

### update_status()

Update message state.

### get_message_by_id()

Lookup message.

---

# Sprint 2 - Authentication & Presence

Goal:
User management and presence tracking

---

## TASK-007

Owner: Jeevan

File:
services/auth_service.py

Purpose:
Authentication business logic.

Functions:

### register_user()

Validate and create account.

### login_user()

Validate credentials.

### hash_password()

Generate bcrypt hash.

### verify_password()

Validate password.

### generate_token()

Generate JWT.

### validate_token()

Verify JWT.

---

## TASK-008

Owner: Pramod

File:
services/presence_service.py

Purpose:
Presence management.

Functions:

### set_online()

Update Redis state.

### set_offline()

Remove Redis state.

### update_last_seen()

Persist timestamp.

### get_presence()

Return online/offline state.

---

## TASK-009

Owner: Jeevan

File:
schemas/auth.py

Purpose:
Authentication request/response schemas.

Schemas:

### RegisterRequest

### LoginRequest

### LoginResponse

---

## TASK-010

Owner: Pramod

File:
api/presence_routes.py

Purpose:
Presence APIs.

Endpoints:

### GET /presence/{user_id}

Returns:

* online
* last_seen

---

# Sprint 3 - Messaging Core

Goal:
Core WhatsApp functionality

---

## TASK-011

Owner: Jeevan

File:
services/message_service.py

Purpose:
Message lifecycle management.

Functions:

### send_message()

Validate recipient.

Create message.

Persist message.

Return SENT state.

---

### get_conversation()

Retrieve ordered messages.

---

### mark_delivered()

Transition:

SENT -> DELIVERED

Update timestamp.

---

### mark_read()

Transition:

DELIVERED -> READ

Update timestamp.

---

### validate_transition()

Prevent invalid state changes.

---

## TASK-012

Owner: Pramod

File:
websocket/connection_manager.py

Purpose:
Active connection tracking.

Responsibilities:

* User socket mapping
* Message routing

Functions:

### connect()

Store websocket.

### disconnect()

Remove websocket.

### get_connection()

Return active websocket.

### send_to_user()

Deliver event.

### broadcast_presence()

Notify status updates.

---

## TASK-013

Owner: Jeevan

File:
schemas/message.py

Purpose:
Message payload schemas.

Schemas:

### SendMessageRequest

### MessageResponse

### StatusUpdateResponse

---

## TASK-014

Owner: Pramod

File:
services/sync_service.py

Purpose:
Offline message synchronization.

Functions:

### sync_pending_messages()

Retrieve pending messages.

Deliver in order.

Update delivery state.

---

# Sprint 4 - APIs & WebSockets

Goal:
Expose backend functionality

---

## TASK-015

Owner: Jeevan

File:
api/auth_routes.py

Purpose:
Authentication endpoints.

Endpoints:

### POST /auth/register

### POST /auth/login

Dependencies:

AuthService

---

## TASK-016

Owner: Pramod

File:
api/websocket_routes.py

Purpose:
Real-time communication endpoint.

Endpoint:

### /ws

Responsibilities:

* Authenticate socket
* Register connection
* Receive events
* Route events

Supported Events:

AUTH

SEND_MESSAGE

DELIVERED

READ

---

## TASK-017

Owner: Jeevan

File:
api/message_routes.py

Purpose:
Message retrieval APIs.

Endpoints:

### GET /messages/{user_id}

Conversation history.

### GET /users

User list.

---

## TASK-018

Owner: Pramod

File:
main.py

Purpose:
Application bootstrap.

Responsibilities:

* Start FastAPI
* Register routes
* Initialize DB
* Initialize Redis

---

# Sprint 5 - Frontend

Goal:
Connect UI to backend

---

## TASK-019

Owner: Jeevan

Files:

frontend/login.html

frontend/register.html

frontend/js/auth.js

Purpose:
Authentication UI.

Responsibilities:

* Register form
* Login form
* JWT storage

Functions:

### login()

Call login API.

### register()

Call register API.

### saveToken()

Persist JWT.

---

## TASK-020

Owner: Pramod

Files:

frontend/chat.html

frontend/js/chat.js

frontend/js/websocket.js

frontend/css/styles.css

Purpose:
Messaging UI.

Responsibilities:

* User list
* Conversation panel
* WebSocket communication
* Presence indicators
* Message status rendering

Functions:

### connectWebSocket()

Establish websocket.

### sendMessage()

Send event.

### renderMessage()

Render chat message.

### updateMessageStatus()

Update ticks.

### loadConversation()

Fetch history.

### refreshPresence()

Update online status.

---

# Sprint 6 - Integration & Testing

Goal:
System validation

---

## TASK-021

Owner: Jeevan

Purpose:
Authentication testing.

Validation:

* Register
* Login
* JWT validation

---

## TASK-022

Owner: Pramod

Purpose:
Messaging testing.

Validation:

* SENT
* DELIVERED
* READ

---

## TASK-023

Owner: Jeevan

Purpose:
Offline sync testing.

Validation:

* User offline
* Messages stored
* Messages delivered after reconnect

---

## TASK-024

Owner: Pramod

Purpose:
Presence testing.

Validation:

* Online
* Offline
* Last seen

---

# Ownership Summary

Backend Tasks

Jeevan:
001
003
005
007
009
011
013
015
017
021
023

Pramod:
002
004
006
008
010
012
014
016
018
022
024

Frontend

Jeevan:
019

Pramod:
020

Total Tasks

Jeevan: 12

Pramod: 12

Backend Split:
50% / 50%

Frontend kept until Sprint 5 after backend APIs stabilize.

---

# Critical Path

Sprint 1
→ Sprint 2
→ Sprint 3
→ Sprint 4
→ Sprint 5
→ Sprint 6

No frontend work should begin until Sprint 4 APIs are complete.

END OF TASKS