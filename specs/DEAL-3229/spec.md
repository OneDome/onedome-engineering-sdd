---
schema_version: 1
jira_key: DEAL-3229
jira_title: Add a new notification type and update users about new comments on tasks in DR
status: approved
created_at: 2026-09-05T21:15:00+03:00
updated_at: 2026-09-05T21:37:00+03:00
approval:
  status: approved
  approved_by: oksanaodocuk
  approved_at: 2026-09-05T21:37:00+03:00
---

# Specification: `DEAL-3229`

Approved **backend** business specification. WHAT and WHY only. Do not invent behaviour. Do not describe HOW. Do not specify frontend/UI.

Human Approval recorded from chat (`I approve this spec`) on 2026-09-05. Includes category `DEAL_COMMENT_NOTIFICATION` and `tickets-service`. Job title is irrelevant.

## Distinctions

| Kind | Where it belongs |
| --- | --- |
| Explicitly requested future behaviour | Expected Behaviour, Business Rules, Acceptance Criteria |
| Verified current behaviour | Current Behaviour |
| Assumptions | Assumptions (never copied into requirements) |
| Unknowns | Open Questions |

## Business Context

Conveyancers already leave comments on Deal Room tasks. PMAs asked them not to, because nobody is notified. The comment feature looks usable, but the notification part is missing.

Evidence: Confluence [Add a new notification type and update users about new comments on tasks in DR](https://onedome.atlassian.net/wiki/spaces/TB/pages/756744195/Add+a+new+notification+type+and+update+users+about+new+comments+on+tasks+in+DR) (page id `756744195`). Locator: [DEAL-3229](https://onedome.atlassian.net/browse/DEAL-3229).

## Problem

Confluence: conveyancers can comment on tasks, but PMAs asked them not to because nobody receives notifications.

Code already writes a comment notification (`DEAL_COMMENT_ADDED`). The gap versus Confluence is type name, action text, B2C, recipients, and opening the comments section — not a missing producer.

## Current Behaviour

Task comments are `actionComment` events on the deal. They are stored on the deal and shown on the task. Adding one goes through `DealService.addEventAndReturnDeal`, which then calls `MenuNotificationService.sendCommentAddedNotification`.

That method already creates a menu notification:

- type `DEAL_COMMENT_ADDED`, category `DEAL_UPDATED_NOTIFICATION`
- body `{firstName} {lastName} ({role}) left a new comment to the task {taskName}` — not the Confluence sentence
- `commentId` = event timestamp as a string (not the comment text)
- `actionId` = task / milestone item id
- header = deal address (or readable flow type)
- redirection params: `dealUuid`, `activeTab` = task id. No flag that the comments section should open
- comment text is not copied onto the notification

Recipients today:

- start from deal participants who are not B2C (`BUYER` / `SELLER`) and who can access that task
- then keep only participants whose role is the task’s `confirmingRoles`, or `PMA`, or the task’s `ownerRole`
- drop the comment author

B2C participants are never notified. PMA means the PMA participant on that deal, and only for tasks they can access.

The write is `tickets.createNotification` from `deals-service`. `tickets-service` `NotificationService.createNotification` persists the `MenuNotification` as given (shopkeeper then Elasticsearch). It does not whitelist type or category. Search is by keyword fields `notificationType` / `notificationCategory`.

`TicketsConstants.MenuNotificationTypes` already has `DEAL_COMMENT_ADDED` and an unused type-group `Set DEAL_COMMENT_NOTIFICATION = Set.of(DEAL_COMMENT_ADDED)`. There is no `COMMENT` type. Category strings on documents match deals-service names (e.g. cucumber uses `DEAL_DOCUMENT_NOTIFICATION`). ES mapping lists `activeTab` / `dealUuid` under `redirectParams`, not `notesTab` or `commentsTab`; `commentId` is read in `ElasticSearchService` but missing from that mapping file.

**`movedin-lambdas` / `sync-notifications`:** does not create notifications. On DynamoDB `MenuNotification` INSERT it copies the record into OpenSearch index `notifications` and bumps Firebase `users/{userUuid}.countOfUpdates`. On REMOVE it deletes the OpenSearch document. On MODIFY it marks read. `notificationType` and `notificationCategory` are copied as opaque fields.

Evidence: `deals-service` `DealService.addEventAndReturnDeal`, `MenuNotificationService.sendCommentAddedNotification`, `DealsConstants.MenuNotificationTypes`, `DealBarsService.addActionComments`; `tickets-service` `NotificationService.createNotification`, `TicketsConstants.MenuNotificationTypes`; `movedin-lambdas/dynamodb-streams/sync-notifications/index.js`.

## Expected Behaviour

Backend only. Screens, filters, columns, colours, tooltips, and click handling are out of scope.

1. Task-comment notifications use a **new** type `COMMENT`. Stored types in `deals-service` are `UPPER_SNAKE_CASE` (`DEAL_CREATED`, `DEAL_NOTE_ADDED`, `SLA_RULE_EXPIRED`). Confluence’s `Comment` / `comment` is the same type; `COMMENT` is the stored value. Do not reuse `DEAL_COMMENT_ADDED`.
2. They use a **new** category `DEAL_COMMENT_NOTIFICATION`. Existing categories are `DEAL_UPDATED_NOTIFICATION`, `DEAL_NOTE_NOTIFICATION`, `DEAL_DOCUMENT_NOTIFICATION`. Do not reuse `DEAL_UPDATED_NOTIFICATION` for this behaviour.
3. When someone adds a comment on a task, persist and deliver a `COMMENT` / `DEAL_COMMENT_NOTIFICATION` notification to:
   - other participants of that task whose role is the task’s `ownerRole` or is in `confirmingRoles`;
   - the PMA participant of **this** Deal Room, for comments on any task on this DR.
4. The comment author does not receive a notification for that comment, even if they are PMA, `ownerRole`, or a confirming role.
5. Action text (Confluence requirement, not the “ideally” line): `[User_name] added a new comment to the task [task_name]`. `User_name` is the author’s name; `task_name` is that task. The comment body itself is not required on the notification.
6. Each such notification must identify the Deal Room, the task, and that the comments section should be opened, so a consumer can navigate there. Exact URI/params are not specified in evidence.
7. Recipients who are B2B or B2C get the same action text and the same navigation identity. List-column layout and B2B type filter are frontend.

## User / System Flows

### Flow 1 — Comment notifies other task participants

1. **Given** a Deal Room task whose `ownerRole` and/or `confirmingRoles` match one or more deal participants other than the comment author
2. **When** a user adds a comment on that task
3. **Then** each of those other participants receives a `COMMENT` notification in category `DEAL_COMMENT_NOTIFICATION`, with action text from BR-002, identifying that Deal Room and task with comments to be opened
4. **And** the author does not receive one

### Flow 2 — PMA of this Deal Room is notified

1. **Given** a Deal Room with a PMA participant who is not the comment author
2. **When** a user adds a comment on any task on that Deal Room
3. **Then** that PMA receives a `COMMENT` notification in category `DEAL_COMMENT_NOTIFICATION`, with BR-002 text, identifying that Deal Room and task with comments to be opened

### Flow 3 — Notification is listable by type (backend contract)

1. **Given** a stored comment notification
2. **When** notifications are retrieved for a user
3. **Then** the record’s type is `COMMENT` and its category is `DEAL_COMMENT_NOTIFICATION`, so a B2B type filter can include it. The filter UI itself is out of scope.

## Business Rules

- **BR-001:** A new comment on a Deal Room task produces a `COMMENT` notification in category `DEAL_COMMENT_NOTIFICATION` for other participants whose role is the task’s `ownerRole` or is in `confirmingRoles`, and for the PMA of this Deal Room.
- **BR-002:** Action text is `[User_name] added a new comment to the task [task_name]`.
- **BR-003:** The notification identifies the Deal Room, the task, and that the comments section should be opened.
- **BR-004:** B2B and B2C share BR-002 and BR-003. Presentation differences are frontend.
- **BR-005:** The comment author is not a recipient of that comment’s notification.
- **BR-006:** The stored notification type for this behaviour is `COMMENT`, not `DEAL_COMMENT_ADDED`.
- **BR-007:** The stored notification category for this behaviour is `DEAL_COMMENT_NOTIFICATION`, not `DEAL_UPDATED_NOTIFICATION`.

## Acceptance Criteria

- **AC-001:** Given a task with other participants in `ownerRole` or `confirmingRoles`, when a comment is added, then each of those users (except the author) has a `COMMENT` notification with BR-002 text.
- **AC-002:** Given this Deal Room’s PMA is not the author, when a comment is added on any task on this DR, then that PMA has a `COMMENT` notification with BR-002 text.
- **AC-003:** Given such a notification, when it is read by a consumer, then it identifies the Deal Room, the task, and that comments should be opened.
- **AC-004:** Given stored notifications, when they are listed, then these records have type `COMMENT` and category `DEAL_COMMENT_NOTIFICATION`.
- **AC-005:** Given the author is also PMA or a task `ownerRole` / confirming role, when they add a comment, then they have no notification for that comment.

## Edge Cases

- If the PMA is the author, they are excluded (BR-005); other `ownerRole` / `confirmingRoles` participants still receive it.
- Comment body on the notification is not required. Confluence “ideally… show comment” is frontend / out of scope.
- Colour for the new B2B type is assigned to Svitlana in Confluence; it is frontend.

## Assumptions

- None recorded.

## Dependencies

- None.

## Potentially Affected Systems

| System / repository id | Why it might be affected | Confidence |
| --- | --- | --- |
| deals-service | Creates `actionComment` events and `DEAL_COMMENT_ADDED` menu notifications. | high |
| tickets-service | Persists and searches menu notifications. Constants still group comments as `DEAL_COMMENT_ADDED`. | high |
| movedin-lambdas | `sync-notifications` already copies `notificationType`, `notificationCategory`, `commentId`, `body`, and `redirection`. A new type/category does not by itself require a lambda change. | medium |

## Out of Scope

- Frontend / UI (screens, client apps, styling). SDD delivers backend only.
- B2B Notification page Type filter control.
- B2B table columns (user name and surname, DR address, Stage, date and time, action, delete button).
- B2B “show comment” tooltip/button.
- B2B colour for the new type (Confluence: Svitlana).
- Click handling in B2B/B2C clients. Backend supplies identity for navigation (BR-003); clients are out of scope.

## Open Questions

None.

## Evidence / References

### Jira

- [DEAL-3229](https://onedome.atlassian.net/browse/DEAL-3229) — locator (title, Confluence remote link). Description not used as requirements.

### Confluence

- Space `TB`, page id `756744195`, title: Add a new notification type and update users about new comments on tasks in DR
- URL: https://onedome.atlassian.net/wiki/spaces/TB/pages/756744195/Add+a+new+notification+type+and+update+users+about+new+comments+on+tasks+in+DR
- Tiny link (same page): https://onedome.atlassian.net/wiki/x/AwAbLQ

### Source code

- `movedin-lambdas` `dynamodb-streams/sync-notifications/index.js`
- `movedin-lambdas` `dynamodb-streams/terraform/config/sync-notifications-lambda.tf`
- `deals-service` `src/main/java/co/movedin/deals/service/DealService.java` (`addEventAndReturnDeal`)
- `deals-service` `src/main/java/co/movedin/deals/service/MenuNotificationService.java` (`sendCommentAddedNotification`)
- `deals-service` `src/main/java/co/movedin/deals/DealsConstants.java` (`MenuNotificationTypes`, `ACTION_COMMENT_EVENT_NAME`)
- `deals-service` `src/main/java/co/movedin/deals/service/DealBarsService.java` (`addActionComments`)
- `tickets-service` `src/main/java/co/movedin/tickets/TicketsConstants.java` (`MenuNotificationTypes`)
- `tickets-service` `src/main/java/co/movedin/tickets/notification/service/NotificationService.java` (`createNotification`)
- `tickets-service` `src/main/resources/elasticsearch/mappings/notifications.json`

### Previous specs

- none used

### Human Review (this session)

- New stored type: `COMMENT` (`UPPER_SNAKE_CASE`, not `DEAL_COMMENT_ADDED`). Confluence `Comment` / `comment` is this type.
- New stored category: `DEAL_COMMENT_NOTIFICATION` (same pattern as `DEAL_NOTE_NOTIFICATION` / `DEAL_DOCUMENT_NOTIFICATION`). Not `DEAL_UPDATED_NOTIFICATION`.
- PMA: participant of this Deal Room, all tasks on this DR.
- Author does not receive; other task participants do (`ownerRole` and `confirmingRoles`).
- Required text is Confluence action name: `[User_name] added a new comment to the task [task_name]`. Comment body on the payload is not required.
- “Uploaders/approvers” means task `ownerRole` and `confirmingRoles`.

### Engineering knowledge

- none used
