---
schema_version: 1
jira_key: DEAL-3229
title: Add a new notification type and update users about new comments on tasks in DR
status: approved
spec_status_required: approved
---

# Plan: `DEAL-3229`

HOW only. Must implement the approved specification. Do not change WHAT/WHY here.

## Gate

- Specification path: `specs/DEAL-3229/spec.md`
- Specification is approved in `task-state.yaml` (`approved_by: oksanaodocuk`, `method: chat`).
- Plan Human Approval recorded from chat (`I approve this plan`) on 2026-09-05.

## Approach

Change the existing comment-notification producer in `deals-service` and align `tickets-service` constants + ES mapping. Do not add a second producer. Do not change `sync-notifications` unless a write is rejected. Frontend stays out of scope.

`DealService.addEventAndReturnDeal` already calls `MenuNotificationService.sendCommentAddedNotification` when the event name is `actionComment`. Keep that hook. Change what that method writes. `tickets-service` `POST /api/tickets/notification` already persists any `notificationType` / `notificationCategory` string; search is keyword match. Still update tickets so the type-group used for comments is `COMMENT`, not `DEAL_COMMENT_ADDED`.

1. **Type.** Add `COMMENT` to `DealsConstants.MenuNotificationTypes` and `TicketsConstants.MenuNotificationTypes`. Send `COMMENT` on new comment notifications. Leave `DEAL_COMMENT_ADDED` in both constants (historical records). Stop using it in `sendCommentAddedNotification`. In tickets, change unused set `DEAL_COMMENT_NOTIFICATION` from `Set.of(DEAL_COMMENT_ADDED)` to `Set.of(COMMENT)`.
2. **Category.** Add `DEAL_COMMENT_NOTIFICATION` to `DealsConstants.MenuNotificationCategories`. Send that category on new comment notifications. Do not use `DEAL_UPDATED_NOTIFICATION` on this path. Tickets has no category enum; the stored string is `DEAL_COMMENT_NOTIFICATION` (same name as the tickets type-group set). Leave other categories unchanged.
3. **Body.** Replace `COMMENT_ADDED_BODY_TEMPLATE` with `[User_name] added a new comment to the task [task_name]`: `"%s %s added a new comment to the task %s"` using `firstName`, `lastName`, and existing `getTaskGenericName`. Do not put role in the body. Do not copy comment text onto the notification. Keep `commentId` as the event timestamp string.
4. **Recipients.** Do not use `getAllowedToReceiveNotificationUserUuids` on this path (it drops B2C and requires task access). Build a set:
   - participants whose role equals the task `ownerRole` or is in `confirmingRoles` (`MilestoneAction` for the event’s milestone item id);
   - union participants whose role is `PMA` on **this** deal, for any task;
   - minus the comment author.
   Deduplicate. Keep the existing throw if the milestone action is missing (`0119`). Do not change document/note notification recipient helpers.
5. **Redirection (BR-003).** Keep `dealUuid` and `activeTab` = task id. Add a comments-open flag the same way notes use `notesTab`: extra redirect param `commentsTab` = `"commentsTab"`. Keep top-level `commentId`. Add `commentsTab` to tickets ES mapping `redirection.redirectParams` (same shape as `activeTab`). Clients are out of scope; this is the backend identity.

`createNotification` still posts `MenuNotification` to `POST /api/tickets/notification`. Initiator fields stay as today.

`movedin-lambdas` `sync-notifications` copies `notificationType` and `notificationCategory` as strings. No lambda change.

## Per-repository changes

### `deals-service`

- **Path:** `deals-service`
- **Change:**
  - `DealsConstants.MenuNotificationTypes`: add `COMMENT = "COMMENT"`.
  - `DealsConstants.MenuNotificationCategories`: add `DEAL_COMMENT_NOTIFICATION = "DEAL_COMMENT_NOTIFICATION"`.
  - `MenuNotificationService.sendCommentAddedNotification`: type `COMMENT`; category `DEAL_COMMENT_NOTIFICATION`; new body template; recipient set as in Approach; redirect params `dealUuid`, `activeTab` (task id), `commentsTab` (`"commentsTab"`); still exclude author; still set `actionId`, `commentId`, header, initiator.
  - Do not alter `sendNoteAddedNotification` / `sendDocumentUploadedNotification` / `getAllowedToReceiveNotificationUserUuids`.
  - Unit tests for `sendCommentAddedNotification` (mock `ButlerIntegration`, `TicketsIntegration`, `DealsConfigsManager`, `Deals`): type, category, body, recipients (owner + confirming + PMA, include B2C owner/confirming, exclude author even if PMA), no comment text in body, redirect params. Account for async `createNotification` (`CurrentContextProvider` executor) the same way other tests in this service do, or capture after the executor runs.
- **Code evidence inspected:** `src/main/java/co/movedin/deals/DealsConstants.java`; `src/main/java/co/movedin/deals/service/MenuNotificationService.java` (`sendCommentAddedNotification`, `validateAllowedUsersForCurrentTask`, `getAllowedToReceiveNotificationUserUuids`, `createNotification`); `src/main/java/co/movedin/deals/service/DealService.java` (`addEventAndReturnDeal`); `src/main/java/co/movedin/deals/model/MilestoneAction.java`; `src/main/java/co/movedin/deals/integration/tickets/TicketsClient.java`; `src/it/java/co/movedin/cucumber/ActionTestSteps.java` (comment event; does not assert notifications).
- **Branch:** null

### `tickets-service`

- **Path:** `tickets-service`
- **Change:**
  - `TicketsConstants.MenuNotificationTypes`: add `COMMENT = "COMMENT"`; set `DEAL_COMMENT_NOTIFICATION` to `Set.of(COMMENT)`. Keep `DEAL_COMMENT_ADDED` for old rows.
  - `src/main/resources/elasticsearch/mappings/notifications.json`: add `redirection.redirectParams.commentsTab` with the same text+keyword mapping as `activeTab`.
  - Do not add a type/category whitelist on `NotificationService.createNotification`. Search stays criteria on keyword fields.
  - Optional cucumber row for `COMMENT` / `DEAL_COMMENT_NOTIFICATION` only if an existing notification search scenario is a cheap place; not required for AC.
- **Code evidence inspected:** `src/main/java/co/movedin/tickets/TicketsConstants.java`; `src/main/java/co/movedin/tickets/notification/service/NotificationService.java`; `src/main/java/co/movedin/tickets/notification/controller/NotificationController.java`; `src/main/resources/elasticsearch/mappings/notifications.json`; `src/it/resources/cucumber/features/notifications.feature`.
- **Branch:** null

### `movedin-lambdas`

- **Path:** `movedin-lambdas`
- **Change:** none. Confirm during implementation that INSERT still indexes opaque `COMMENT` type and `DEAL_COMMENT_NOTIFICATION` category (`index.js` has no enum).
- **Code evidence inspected:** `dynamodb-streams/sync-notifications/index.js`; `dynamodb-streams/terraform/config/sync-notifications-lambda.tf`.
- **Branch:** null

No other catalog repository is an SDD delivery target. Frontend is a consumer only.

## Sequencing

1. `deals-service`: add `COMMENT` type and `DEAL_COMMENT_NOTIFICATION` category constants.
2. `tickets-service`: add `COMMENT`; point type-group `DEAL_COMMENT_NOTIFICATION` at `COMMENT`; map `commentsTab`.
3. `deals-service`: change `sendCommentAddedNotification` (type, category, body, recipients, redirect param).
4. `deals-service`: unit tests for AC-001–AC-005.
5. No lambda deploy.

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| `tickets-service` type-group stays on `DEAL_COMMENT_ADDED` | B2B “Comment” grouping (if it ever uses that set) misses new rows | Update the set to `COMMENT` in the same change |
| `commentsTab` missing from tickets ES mapping | Search/index may drop the comments-open flag | Add `commentsTab` next to `activeTab`. `notesTab` is already absent; do not backfill notes |
| `commentsTab` is a new param; clients may ignore it | Task opens without comments section (AC-003) | Mirror `notesTab`. Frontend is out of scope; backend still sends deal, task, comments flag, `commentId` |
| Historical `DEAL_COMMENT_ADDED` rows remain | Old list items keep the old type | Spec requires new type for new comments only. Do not migrate |
| Async `createNotification` | Flaky unit tests | Run or wait on the context executor; assert on `TicketsIntegration.createNotification` |

## Test plan

- **AC-001:** Comment on a task → `TicketsIntegration.createNotification` once per other participant with `ownerRole` or `confirmingRoles`; type `COMMENT`; body `{first} {last} added a new comment to the task {genericName}`. Include a B2C participant when their role is owner/confirming.
- **AC-002:** Deal PMA who is not the author receives one `COMMENT` notification even if PMA is not owner/confirming and would fail today’s `canAccessToAction` filter.
- **AC-003:** Each written notification has redirect params `dealUuid`, `activeTab` = task id, `commentsTab` = `"commentsTab"`, plus `commentId`.
- **AC-004:** `notificationType` is `COMMENT`, not `DEAL_COMMENT_ADDED`. `notificationCategory` is `DEAL_COMMENT_NOTIFICATION`, not `DEAL_UPDATED_NOTIFICATION`.
- **AC-005:** Author is PMA and/or owner/confirming → no notification for `authorUuid`. Other recipients still receive.

No cucumber change required unless an existing comment scenario starts failing.

## Knowledge updates

```yaml
- path: knowledge/notifications/deal-task-comment.md
  action: add
  notes: After implementation, record COMMENT type, DEAL_COMMENT_NOTIFICATION category, recipient set (ownerRole, confirmingRoles, PMA of this DR, exclude author), and redirect params. Verify against MenuNotificationService, not Confluence.
```

## Open Questions

None.
