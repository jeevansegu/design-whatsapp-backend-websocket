# WhatsApp Backend Design Document

## Feature: Single Tick, Double Tick, Blue Tick Messaging System

---

# 1. Goal

Implement a WhatsApp-style messaging backend where:

```text
User A
   |
   | Send Message
   v
Server
   |
   | Store Message
   |
   +----> Single Tick

Receiver Online?
   |
  Yes
   |
   v
Deliver Message
   |
   +----> Double Tick

Receiver Opens Chat
   |
   v
Read Receipt
   |
   +----> Blue Tick
```

---

# 2. Functional Requirements

### Messaging

* Send message
* Receive message
* Store message history

### Status Tracking

| Status    | Meaning                      |
| --------- | ---------------------------- |
| SENT      | Stored on server             |
| DELIVERED | Received by recipient device |
| READ      | Opened by recipient          |

### Presence

* User online
* User offline

### Reliability

* No message loss
* Retry delivery

---

# 3. High Level Design (HLD)

```text
                     +----------------+
                     | API Gateway    |
                     +--------+-------+
                              |
                              |
                +-------------v-------------+
                |     Message Service       |
                +-------------+-------------+
                              |
         +-------------------+-------------------+
         |                                       |
         v                                       v

 +---------------+                   +-------------------+
 | Message DB    |                   | Kafka / Queue     |
 +---------------+                   +-------------------+
                                              |
                                              |
                                    +---------v----------+
                                    | Delivery Worker    |
                                    +---------+----------+
                                              |
                                              |
                                +-------------v-------------+
                                | Connection Manager        |
                                +-------------+-------------+
                                              |
                     +------------------------+-----------------------+
                     |                                                |
                     v                                                v

             User A Socket                                 User B Socket
```

---

# 4. Project Structure

```text
whatsapp_backend/

│
├── app.py
│
├── api/
│   ├── message_api.py
│   ├── receipt_api.py
│   └── presence_api.py
│
├── models/
│   ├── message.py
│   ├── user.py
│   └── enums.py
│
├── repositories/
│   ├── message_repository.py
│   └── presence_repository.py
│
├── services/
│   ├── message_service.py
│   ├── delivery_service.py
│   ├── read_receipt_service.py
│   └── presence_service.py
│
├── websocket/
│   ├── connection_manager.py
│   └── websocket_handler.py
│
├── workers/
│   └── delivery_worker.py
│
├── queue/
│   └── kafka_client.py
│
├── database/
│   └── db.py
│
└── tests/
    ├── test_message.py
    ├── test_delivery.py
    └── test_receipts.py
```

---

# 5. Database Design

## messages

```sql
CREATE TABLE messages(
    message_id UUID PRIMARY KEY,

    sender_id UUID,
    receiver_id UUID,

    content TEXT,

    status VARCHAR(20),

    created_at TIMESTAMP,

    delivered_at TIMESTAMP,

    read_at TIMESTAMP
);
```

---

# 6. LLD

---

## MessageStatus Enum

### File

```text
models/enums.py
```

### Purpose

Represents lifecycle of a message.

### Code

```python
class MessageStatus(Enum):
    SENT = "sent"
    DELIVERED = "delivered"
    READ = "read"
```

---

# Message Model

### File

```text
models/message.py
```

### Purpose

Represents a single message object.

### Fields

```python
class Message:

    message_id: str

    sender_id: str

    receiver_id: str

    content: str

    status: MessageStatus

    created_at: datetime

    delivered_at: datetime

    read_at: datetime
```

---

# 7. Repository Layer

Repositories interact with DB only.

---

## MessageRepository

### File

```text
repositories/message_repository.py
```

---

### save_message()

```python
def save_message(
    self,
    message: Message
):
```

### Responsibility

Insert new message into DB.

### Called By

```text
MessageService
```

---

### get_message()

```python
def get_message(
    self,
    message_id: str
)
```

### Responsibility

Fetch message details.

### Used In

```text
Delivery Worker
Read Receipt Service
```

---

### update_status()

```python
def update_status(
    self,
    message_id: str,
    status: MessageStatus
)
```

### Responsibility

Update message state.

### Used In

```text
Delivery Service
Read Receipt Service
```

---

### get_pending_messages()

```python
def get_pending_messages(
    self,
    user_id: str
)
```

### Responsibility

Fetch all messages not yet delivered.

### Used In

```text
Reconnect Flow
```

---

# PresenceRepository

### File

```text
repositories/presence_repository.py
```

Uses Redis.

---

### mark_online()

```python
def mark_online(
    user_id,
    socket_id
)
```

Stores:

```text
user_id -> socket_id
```

---

### mark_offline()

```python
def mark_offline(
    user_id
)
```

Removes mapping.

---

### is_online()

```python
def is_online(
    user_id
)
```

Returns:

```python
True / False
```

---

### get_socket()

```python
def get_socket(
    user_id
)
```

Returns active socket.

---

# 8. Service Layer

Contains business logic.

---

# MessageService

### File

```text
services/message_service.py
```

Responsible for sending messages.

---

## send_message()

```python
def send_message(
    sender_id,
    receiver_id,
    content
)
```

### Flow

```text
Validate Request
      |
Generate Message ID
      |
Create Message Object
      |
Save To DB
      |
Publish Event
      |
Return SENT ACK
```

### Returns

```json
{
  "message_id":"123",
  "status":"sent"
}
```

---

# DeliveryService

### File

```text
services/delivery_service.py
```

Responsible for device delivery.

---

## deliver_message()

```python
def deliver_message(
    message_id
)
```

### Flow

```text
Fetch Message
      |
Check Online Status
      |
Send Through WebSocket
      |
Wait For ACK
      |
Update DELIVERED
```

---

## notify_sender_delivered()

```python
def notify_sender_delivered(
    sender_id,
    message_id
)
```

### Purpose

Send double tick event to sender.

### Payload

```json
{
  "type":"delivered",
  "message_id":"123"
}
```

---

# ReadReceiptService

### File

```text
services/read_receipt_service.py
```

Responsible for blue ticks.

---

## mark_as_read()

```python
def mark_as_read(
    receiver_id,
    message_id
)
```

### Flow

```text
User Opens Chat
      |
Message Read
      |
Update READ
      |
Notify Sender
```

---

## notify_sender_read()

```python
def notify_sender_read(
    sender_id,
    message_id
)
```

### Payload

```json
{
  "type":"read",
  "message_id":"123"
}
```

---

# PresenceService

### File

```text
services/presence_service.py
```

Handles user connectivity.

---

## user_connected()

```python
def user_connected(
    user_id,
    websocket
)
```

### Flow

```text
Store Socket
Mark Online
Fetch Pending Messages
Trigger Delivery
```

---

## user_disconnected()

```python
def user_disconnected(
    user_id
)
```

### Flow

```text
Remove Socket
Mark Offline
```

---

# 9. WebSocket Layer

---

# ConnectionManager

### File

```text
websocket/connection_manager.py
```

Maintains active sockets.

---

### connect()

```python
def connect(
    user_id,
    websocket
)
```

Stores:

```text
user_id -> websocket
```

---

### disconnect()

```python
def disconnect(
    user_id
)
```

Removes connection.

---

### send()

```python
def send(
    user_id,
    payload
)
```

Push data to user.

---

### exists()

```python
def exists(
    user_id
)
```

Checks active connection.

---

# WebSocketHandler

### File

```text
websocket/websocket_handler.py
```

Routes incoming socket events.

---

### handle_message()

```python
def handle_message(
    event
)
```

Processes:

```text
SEND_MESSAGE
DELIVERY_ACK
READ_ACK
```

---

# 10. Queue Layer

---

# KafkaClient

### File

```text
queue/kafka_client.py
```

---

## publish()

```python
def publish(
    topic,
    payload
)
```

Used when:

```text
New message created
```

---

## consume()

```python
def consume(
    topic
)
```

Used by worker.

---

# 11. Worker Layer

---

# DeliveryWorker

### File

```text
workers/delivery_worker.py
```

Runs continuously.

---

## process_message_created_event()

```python
def process_message_created_event(
    event
)
```

### Flow

```text
Message Created Event
        |
Fetch Message
        |
Receiver Online?
        |
      Yes
        |
Deliver
        |
Wait ACK
        |
Update DELIVERED
```

---

# 12. Tick Protocol

---

## Single Tick

### Trigger

```text
Message stored in DB
```

### Sender UI

```text
✓
```

### Status

```python
SENT
```

---

## Double Tick

### Trigger

```text
Receiver device sends delivery ACK
```

### Sender UI

```text
✓✓
```

### Status

```python
DELIVERED
```

---

## Blue Tick

### Trigger

```text
Receiver opens chat
```

### Sender UI

```text
✓✓ 🔵
```

### Status

```python
READ
```

---

# 13. Complete Sequence Diagram

```text
USER A
 |
 | Send Message
 |
 v

MessageService
 |
 | Save Message
 |
 v

Database
 |
 | Status = SENT
 |
 +-------------------------+
                           |
                           v

Sender gets ACK

✓

------------------------------------------------

DeliveryWorker
 |
 | Receiver Online?
 |
 +---- No --> Wait
 |
 +---- Yes
          |
          v

Receiver Device

Message Received

Receiver -> DELIVERY_ACK

DeliveryService

Status = DELIVERED

Sender gets:

✓✓

------------------------------------------------

Receiver Opens Chat

READ_ACK

ReadReceiptService

Status = READ

Sender gets:

✓✓ 🔵
```

---

# 14. Future Enhancements

### Group Messaging

```text
conversation_members table
```

### Typing Indicators

```text
USER_TYPING event
```

### Last Seen

```text
presence service
```

### End-to-End Encryption

```text
Signal Protocol
```

### Multi Device Support

```text
user_id -> multiple sockets
```

### Media Messages

```text
S3/GCS object storage
```

This is roughly the level of detail you'd see in a design task generated by an autonomous coding agent before it starts creating files and implementing code. It clearly maps **requirements → architecture → files → classes → functions → message flow → responsibilities**, making it easy for multiple agents to work in parallel.
