# WhatsApp Backend MVP - Implementation Tasks

# Project Structure

```text
whatsapp-backend/

├── server/
│   ├── main.py
│   ├── websocket/
│   │   ├── server.py
│   │   ├── connection_manager.py
│   │   └── events.py
│   │
│   ├── services/
│   │   ├── auth_service.py
│   │   ├── message_service.py
│   │   ├── presence_service.py
│   │   └── sync_service.py
│   │
│   ├── repositories/
│   │   ├── user_repository.py
│   │   └── message_repository.py
│   │
│   └── models/
│       ├── user.py
│       └── message.py
│
├── database/
│   ├── connection.py
│   └── schema.sql
│
├── redis/
│   └── redis_client.py
│
├── client/
│   └── terminal_client.py
│
└── tests/
```

# TASK-001

## Create Database Schema

Files

database/schema.sql

Responsibilities

* Create users table
* Create messages table
* Create indexes

Validation

Database initializes successfully.

---

# TASK-002

## Create Database Connection Layer

Files

database/connection.py

Functions

create_connection()

get_session()

close_connection()

Validation

Queries execute successfully.

---

# TASK-003

## Create User Repository

Files

repositories/user_repository.py

Functions

create_user()

get_user_by_username()

get_user_by_id()

update_last_seen()

Purpose

Encapsulate user database access.

Validation

User CRUD operations work.

---

# TASK-004

## Create Message Repository

Files

repositories/message_repository.py

Functions

create_message()

get_conversation()

update_status()

get_pending_messages()

Purpose

Encapsulate message database access.

Validation

Messages persist correctly.

---

# TASK-005

## Create Authentication Service

Files

services/auth_service.py

Functions

register()

login()

hash_password()

verify_password()

Validation

Authentication works.

---

# TASK-006

## Create Connection Manager

Files

websocket/connection_manager.py

Functions

connect()

disconnect()

send_to_user()

broadcast()

is_online()

Validation

Connections managed correctly.

---

# TASK-007

## Create Presence Service

Files

services/presence_service.py

Functions

set_online()

set_offline()

get_status()

Validation

Presence updates correctly.

---

# TASK-008

## Create Message Service

Files

services/message_service.py

Functions

send_message()

mark_delivered()

mark_read()

get_history()

Validation

Message states work correctly.

---

# TASK-009

## Create Sync Service

Files

services/sync_service.py

Functions

sync_pending_messages()

Validation

Offline messages synchronize.

---

# TASK-010

## Create WebSocket Event Layer

Files

websocket/events.py

Events

REGISTER

LOGIN

SEND_MESSAGE

DELIVERED

READ

GET_HISTORY

GET_STATUS

Validation

Events parsed correctly.

---

# TASK-011

## Create WebSocket Server

Files

websocket/server.py

Responsibilities

* Accept connections
* Route events
* Invoke services

Validation

Multiple users can connect.

---

# TASK-012

## Create Terminal Client

Files

client/terminal_client.py

Commands

/register

/login

/send

/history

/status

/read

Validation

User can interact with server.

---

# TASK-013

## Integration Testing

Scenario 1

User A → User B

Verify:

SENT

DELIVERED

READ

Scenario 2

User B offline

Verify:

Offline storage

Reconnect sync

Scenario 3

Presence tracking

Verify:

Online

Offline

Last Seen

# Implementation Order

1. Database
2. Repositories
3. Auth Service
4. Connection Manager
5. Presence Service
6. Message Service
7. Sync Service
8. WebSocket Layer
9. Terminal Client
10. Testing

# Estimated Effort

Database Layer: 2h

Repositories: 2h

Services: 5h

WebSocket Layer: 4h

Terminal Client: 2h

Testing: 3h

Total: ~18 Hours
