# WhatsApp Backend MVP - Requirements Specification

Version: 1.0

Status: Approved Requirements

---

# 1. Project Overview

## 1.1 Objective

Build a backend-only WhatsApp-style messaging system that demonstrates the core concepts behind real-time messaging systems.

The system will support multiple terminal clients connected to a central WebSocket server and will implement:

* User registration
* User login
* One-to-one messaging
* Message persistence
* Online/offline presence
* Last seen
* Offline synchronization
* Single Tick
* Double Tick
* Blue Tick

The project is intended as a learning and interview-preparation project rather than a production-scale messaging platform.

---

# 2. Scope

## In Scope

### Authentication

* User registration
* User login
* Password hashing

### Messaging

* One-to-one messaging
* Real-time message delivery
* Message persistence
* Conversation history

### Delivery States

* SENT
* DELIVERED
* READ

### Presence

* Online status
* Offline status
* Last seen timestamp

### Synchronization

* Offline message storage
* Offline message delivery after reconnect

### Infrastructure

* PostgreSQL
* Redis
* WebSockets
* Terminal-based clients

---

## Out of Scope

* Group chats
* Media messages
* Push notifications
* Message editing
* Message deletion
* Voice/video calls
* Multi-device support
* End-to-end encryption
* Kafka
* Microservices

---

# 3. User Stories

## US-001 Register User

As a new user

I want to create an account

So that I can use the messaging system.

---

## US-002 Login

As a user

I want to login

So that I can access messaging features.

---

## US-003 Send Message

As a user

I want to send a message

So that I can communicate with another user.

---

## US-004 Receive Message

As a user

I want to receive messages instantly

So that conversations occur in real time.

---

## US-005 View Delivery State

As a sender

I want to know whether my message was sent, delivered, or read

So that I know its current status.

---

## US-006 Offline Messaging

As a user

I want messages to be delivered after reconnecting

So that I never lose messages.

---

## US-007 View Presence

As a user

I want to see whether another user is online

So that I know if they are available.

---

## US-008 View Last Seen

As a user

I want to see the last active time of another user

So that I know when they were last connected.

---

# 4. Message Delivery Protocol

## SENT (Single Tick)

A message reaches SENT state when:

1. Server receives message.
2. Message is successfully persisted.
3. Message ID is generated.

Sender receives:

✓

---

## DELIVERED (Double Tick)

A message reaches DELIVERED state when:

1. Recipient receives message through WebSocket.
2. Recipient sends delivery acknowledgement.

Sender receives:

✓✓

---

## READ (Blue Tick)

A message reaches READ state when:

1. Recipient reads message.
2. Recipient sends read acknowledgement.

Sender receives:

✓✓ (Read)

---

## State Transitions

Valid transitions:

SENT → DELIVERED → READ

Invalid transitions:

READ → DELIVERED

READ → SENT

DELIVERED → SENT

---

# 5. Functional Requirements

## Authentication

WHEN a user submits registration information

THE SYSTEM SHALL create a user account.

WHEN valid credentials are submitted

THE SYSTEM SHALL authenticate the user.

WHEN invalid credentials are submitted

THE SYSTEM SHALL reject the login request.

---

## Connection Management

WHEN a user connects

THE SYSTEM SHALL establish a WebSocket session.

WHEN a user connects

THE SYSTEM SHALL mark the user ONLINE.

WHEN a user disconnects

THE SYSTEM SHALL mark the user OFFLINE.

WHEN a user disconnects

THE SYSTEM SHALL update last_seen.

---

## Messaging

WHEN a user sends a message

THE SYSTEM SHALL persist the message before delivery.

WHEN persistence succeeds

THE SYSTEM SHALL assign a unique message identifier.

WHEN persistence succeeds

THE SYSTEM SHALL update status to SENT.

WHEN recipient is online

THE SYSTEM SHALL immediately route the message.

WHEN recipient is offline

THE SYSTEM SHALL retain the message for future delivery.

---

## Delivery Receipts

WHEN recipient receives a message

THE SYSTEM SHALL update status to DELIVERED.

WHEN recipient reads a message

THE SYSTEM SHALL update status to READ.

WHEN status changes

THE SYSTEM SHALL notify the sender.

---

## Offline Synchronization

WHEN a user reconnects

THE SYSTEM SHALL retrieve pending messages.

WHEN pending messages exist

THE SYSTEM SHALL deliver messages in creation order.

WHEN synchronization completes

THE SYSTEM SHALL update delivery states.

---

## Presence

WHEN a status request is received

THE SYSTEM SHALL return online/offline status.

WHEN a user is offline

THE SYSTEM SHALL return last_seen timestamp.
