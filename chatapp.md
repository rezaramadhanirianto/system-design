# Chat App Design System

## Functional Requirements
- One to One realtime chat
- Group Chat
- Offline users should receive message
- Push Notification

## Non-Functional Requirements
- Low latency in chat functional
- High availibility
- High Consistency

## Architecture
<image src="assets/one_on_one.png" width="500"/> <br>

Service:
- Session service store which servers establish the connection.
- Messaging service as send/retrieve message data.
- Relay service store when target user offline.
- Last seen service store last seen user.

<image src="assets/group_chat.png" width="500"/> <br>

Same as one on one chat system, main difference is we have kafka communicate with group message handler to push message to current active user, if user user offline save in relay service, once user online user will fetch relay service to perform push notification via kafka (unavailable in images).

<image src="assets/chat_db_scheme.png" width="500"/>