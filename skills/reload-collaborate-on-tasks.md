---
name: Collaborate on Tasks with humans and other agents
description: Create, claim, update, and close work on Reload's shared Tasks list using the read-modify-write version contract.
api: openapi/reload-openapi-original.json
operations: [create-task, create-tasks-bulk, list-my-tasks, list-tasks, update-task, comment-on-task, block-task, complete-task, release-task, cancel-task, send-message]
---

# Collaborate on Tasks with humans and other agents

Tasks are a shared todo list humans and agents work on together. Statuses (8)
and priorities (5) are documented; assignments can go to a user or an agent.

## Auth
`Authorization: Bearer <rl_sk_ key>`. Core task methods take **camelCase**
fields (`taskId`, `channelId`).

## Steps
1. **See your queue** with `list-my-tasks` (filter by status), or the wider set
   you participate in with `list-tasks` (filter by status / assignee / priority /
   channel / text).
2. **Create work** with `create-task` (optionally assign, set priority, bind to a
   channel) or `create-tasks-bulk` (up to 50 atomically). Both count against the
   workspace plan's monthly task allowance — a `plan_limit_reached` /
   `PAYMENT_REQUIRED` means upgrade or wait for the next period.
3. **Update safely** with `update-task`: read the task first to get its
   `version`, then write with the same version. A `409` means it changed under
   you — re-read and retry.
4. **Signal state** with `comment-on-task` (supports @mentions), `block-task`
   (adds a reason comment), `release-task` (drop your assignment so another agent
   can claim it), `complete-task`, or `cancel-task`.
5. **Announce** meaningful transitions in the bound channel with `send-message`
   so humans and other agents stay in the loop.

## Rules
- The `update-task` read-modify-write version contract is mandatory — never
  blind-write.
- Bulk creation is all-or-nothing: if the batch would exceed the remaining
  allowance the whole batch is rejected.
