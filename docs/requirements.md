# WhatsApp Backend MVP
Version: 1.0
Status: Requirements Phase
Project Type: Learning + Interview Scale Backend System

---

# 1. Overview

## 1.1 Problem Statement

Modern messaging applications provide real-time communication, message persistence, delivery acknowledgements, read receipts, and offline synchronization.

The objective of this project is to build a simplified WhatsApp-style backend that demonstrates these core concepts using modern backend technologies while remaining small enough to be implemented by a two-person team.

The system will allow users to:

- Register and login
- View available users
- Send messages
- Receive messages in real time
- View message delivery status
- View online/offline presence
- View last seen timestamps
- Retrieve historical conversations
- Receive missed messages after reconnecting

---

# 1.2 Goals

The primary goals are:

### Goal 1: Real-Time Communication

Messages should be delivered instantly when both users are online.

### Goal 2: Reliable Persistence

Messages should never be lost after being accepted by the server.

### Goal 3: Delivery Tracking

The sender should know whether a message was:

- Sent
- Delivered
- Read

### Goal 4: Presence Tracking

Users should know whether another user is:

- Online
- Offline
- Last seen at a specific time

### Goal 5: Offline Synchronization

Messages sent to offline users should be delivered when they reconnect.

---

# 1.3 Success Criteria

The project is considered successful if:

- Multiple users can communicate simultaneously.
- Messages persist across server restarts.
- Delivery states work correctly.
- Offline synchronization works.
- Presence tracking works.
- Last seen updates correctly.
- System supports at least 50 concurrent users in local testing.

---

# 2. Scope

## 2.1 In Scope

### Authentication

- User registration
- User login
- Password hashing
- JWT generation
- JWT validation

### Messaging

- One-to-one messaging
- Message persistence
- Message history retrieval
- Real-time delivery

### Message States

- SENT
- DELIVERED
- READ

### Presence

- Online status
- Offline status
- Last seen

### Synchronization

- Offline message storage
- Offline message delivery

### Frontend

- Login page
- Registration page
- Chat page
- User list
- Conversation view

### Infrastructure

- FastAPI
- PostgreSQL
- Redis
- WebSockets

---

## 2.2 Out Of Scope

The following features are intentionally excluded:

### Messaging

- Group chats
- Message forwarding
- Message editing
- Message deletion
- Reactions

### Media

- Images
- Videos
- Documents
- Voice notes

### Advanced Features

- Push notifications
- Multi-device support
- End-to-end encryption
- Typing indicators
- Voice calling
- Video calling

### Infrastructure

- Kafka
- Microservices
- Kubernetes
- Load balancers

---

# 3. User Roles

## Registered User

A user who has created an account and can access all messaging functionality.

No administrative roles exist in the MVP.

---

# 4. User Stories

## Authentication

### US-001 Register

As a new user

I want to create an account

So that I can access the platform.

---

### US-002 Login

As a registered user

I want to login securely

So that I can access my conversations.

---

## Messaging

### US-003 Send Message

As a user

I want to send messages

So that I can communicate with another user.

---

### US-004 Receive Message

As a user

I want to receive messages instantly

So that conversations feel real time.

---

### US-005 View Message History

As a user

I want to view previous messages

So that I can continue older conversations.

---

## Delivery States

### US-006 Single Tick

As a sender

I want to know when a message is stored

So that I know it was accepted by the system.

---

### US-007 Double Tick

As a sender

I want to know when a message reaches the recipient

So that I know delivery succeeded.

---

### US-008 Blue Tick

As a sender

I want to know when a message was read

So that I know it has been seen.

---

## Presence

### US-009 View Online Status

As a user

I want to know whether another user is online

So that I know if they are available.

---

### US-010 View Last Seen

As a user

I want to know when another user was last active

So that I know their recent availability.

---

## Synchronization

### US-011 Offline Sync

As a user

I want messages sent while I was offline

So that I do not miss conversations.

---

# 5. Message Delivery Protocol

This protocol defines message lifecycle behavior.

---

## SENT State

Meaning:

Server has successfully stored the message.

Conditions:

1. Message received.
2. Validation successful.
3. Message persisted.

Result:

Status = SENT

UI:

✓

---

## DELIVERED State

Meaning:

Recipient has successfully received the message.

Conditions:

1. Recipient online.
2. Message delivered via WebSocket.
3. Recipient acknowledges receipt.

Result:

Status = DELIVERED

UI:

✓✓

---

## READ State

Meaning:

Recipient has opened/read the message.

Conditions:

1. Recipient views message.
2. Read acknowledgement sent.

Result:

Status = READ

UI:

✓✓ (Read)

---

## Valid State Transitions

SENT -> DELIVERED

DELIVERED -> READ

---

## Invalid State Transitions

READ -> DELIVERED

READ -> SENT

DELIVERED -> SENT

Any invalid transition must be rejected.

---

# 6. Functional Requirements

## Authentication

### FR-001 Registration

WHEN a user submits valid registration data

THE SYSTEM SHALL create a new user account.

---

### FR-002 Duplicate Username

WHEN a username already exists

THE SYSTEM SHALL reject registration.

---

### FR-003 Password Security

WHEN a password is stored

THE SYSTEM SHALL store only a hashed version.

---

### FR-004 Login

WHEN valid credentials are submitted

THE SYSTEM SHALL authenticate the user.

---

### FR-005 JWT Generation

WHEN authentication succeeds

THE SYSTEM SHALL generate a JWT token.

---

## Connection Management

### FR-006 WebSocket Connection

WHEN a user connects

THE SYSTEM SHALL establish a WebSocket session.

---

### FR-007 Online Presence

WHEN a user connects

THE SYSTEM SHALL mark the user as ONLINE.

---

### FR-008 Offline Presence

WHEN a user disconnects

THE SYSTEM SHALL mark the user as OFFLINE.

---

### FR-009 Last Seen

WHEN a user disconnects

THE SYSTEM SHALL update last_seen.

---

## Messaging

### FR-010 Create Message

WHEN a user sends a message

THE SYSTEM SHALL create a message record.

---

### FR-011 Message Persistence

WHEN a message is received

THE SYSTEM SHALL persist it before delivery.

---

### FR-012 Single Tick

WHEN persistence succeeds

THE SYSTEM SHALL mark status as SENT.

---

### FR-013 Real-Time Delivery

WHEN recipient is online

THE SYSTEM SHALL immediately deliver the message.

---

### FR-014 Offline Handling

WHEN recipient is offline

THE SYSTEM SHALL retain the message.

---

## Delivery Receipts

### FR-015 Delivery Acknowledgement

WHEN a recipient receives a message

THE SYSTEM SHALL update status to DELIVERED.

---

### FR-016 Double Tick Notification

WHEN status changes to DELIVERED

THE SYSTEM SHALL notify the sender.

---

### FR-017 Read Acknowledgement

WHEN a recipient reads a message

THE SYSTEM SHALL update status to READ.

---

### FR-018 Blue Tick Notification

WHEN status changes to READ

THE SYSTEM SHALL notify the sender.

---

## Offline Synchronization

### FR-019 Sync Trigger

WHEN a user reconnects

THE SYSTEM SHALL initiate synchronization.

---

### FR-020 Pending Retrieval

WHEN synchronization begins

THE SYSTEM SHALL retrieve pending messages.

---

### FR-021 Ordered Delivery

WHEN pending messages exist

THE SYSTEM SHALL deliver them in chronological order.

---

## History

### FR-022 Conversation Retrieval

WHEN a conversation is requested

THE SYSTEM SHALL return historical messages ordered by timestamp.

---

## Presence

### FR-023 Presence Query

WHEN a user requests presence information

THE SYSTEM SHALL return online status.

---

### FR-024 Last Seen Query

WHEN a requested user is offline

THE SYSTEM SHALL return last_seen.

---

# 7. Non-Functional Requirements

## Availability

Backend availability target:

99%

for local deployment environments.

---

## Latency

Message delivery latency:

< 500ms

under normal load.

---

## Throughput

Support:

- 50 concurrent users
- 1000 messages/minute

---

## Reliability

Messages must not be lost after persistence.

---

## Maintainability

Business logic must be separated from routing logic.

---

## Security

Passwords must be hashed.

JWT authentication required.

Protected endpoints must require authentication.

---

## Observability

The system shall log:

- User login
- User logout
- Message creation
- Delivery acknowledgements
- Read acknowledgements

---

# 8. Edge Cases

## EC-001 Recipient Offline

Expected:

Store message and deliver later.

---

## EC-002 Duplicate Delivery Ack

Expected:

Ignore duplicate acknowledgement.

---

## EC-003 Duplicate Read Ack

Expected:

Ignore duplicate acknowledgement.

---

## EC-004 Server Restart

Expected:

Messages remain available in PostgreSQL.

---

## EC-005 Reconnect During Sync

Expected:

Synchronization resumes without duplicate delivery.

---

## EC-006 Invalid JWT

Expected:

Reject request.

---

## EC-007 Deleted Browser Tab

Expected:

User marked offline.

Last seen updated.

---

# 9. Acceptance Criteria

## Authentication

- User can register.
- User can login.
- JWT issued successfully.

---

## Messaging

- User A can send messages to User B.
- User B receives messages instantly.

---

## Delivery States

- SENT displayed after persistence.
- DELIVERED displayed after recipient acknowledgement.
- READ displayed after recipient reads message.

---

## Presence

- Online users appear online.
- Offline users display last seen.

---

## Synchronization

- Offline messages delivered after reconnect.

---

## History

- Historical conversations retrieved successfully.

---

END OF REQUIREMENTS