# GroupGuard — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Community moderation bot for Telegram groups that automates spam prevention, new-member verification, and provides admin controls. Features include human confirmation checks, spam detection (links from new accounts, repeated messages, floods), admin-configurable thresholds, trusted user exceptions, and detailed action logs with periodic summaries.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group owners
- Moderators of public/private groups
- Admins seeking automated spam prevention

## Success criteria

- Reduces spam and bot accounts in group by 70% within first week
- Maintains 95% user confirmation rate for new members
- Provides admins with actionable moderation logs and summaries

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu for users or admin controls for admins
- **I'm human** (button, actor: user, callback: join_check:confirm) — Confirm human status after joining group
  - inputs: user_id, join_time
  - outputs: confirmation status, greeting response
- **/log** (command, actor: admin, command: /log) — View action log summary
- **/report** (command, actor: admin, command: /report) — Request detailed moderation report
- **/config** (command, actor: admin, command: /config) — Access configuration settings
- **/trusted** (command, actor: admin, command: /trusted) — Manage trusted user list
- **/warn** (command, actor: admin, command: /warn) — Warn user for rule violations
- **/mute** (command, actor: admin, command: /mute) — Mute user with duration options
- **/ban** (command, actor: admin, command: /ban) — Ban user from group

## Flows

### New-member verification
_Trigger:_ user joins group

1. Send greeting with confirmation button
2. Wait for button press or timeout
3. Confirm or remove user
4. Log action

_Data touched:_ User, JoinCheck

### Spam detection
_Trigger:_ message posted

1. Check message against spam rules
2. Apply configured action (warn/delete/mute/kick/ban)
3. Post explanation in group
4. Log action

_Data touched:_ Message, AdminAction

### Admin configuration
_Trigger:_ /config command

1. Display current settings
2. Allow editing of greeting/rules
3. Update thresholds and actions
4. Save configuration

_Data touched:_ Configuration

### Action logging
_Trigger:_ any moderation action

1. Record actor, target, action, reason
2. Store in persistent log
3. Retain for configured period

_Data touched:_ AdminAction, Configuration

### Report generation
_Trigger:_ /report command

1. Aggregate stats (joins, removals, actions)
2. Format summary with recent log entries
3. Send to admin

_Data touched:_ JoinCheck, Message, AdminAction

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram group member with status and permissions
  - fields: id, display name, role, join time, confirmation status
- **JoinCheck** _(retention: persistent)_ — Record of new member verification process
  - fields: timestamp, user id, greeting sent, confirmation button pressed, expiration, outcome, explanatory message
- **Message** _(retention: persistent)_ — Record of messages for spam detection and moderation
  - fields: message id, sender id, timestamp, content hash, contains link, is pinned, action taken, reason, actor
- **AdminAction** _(retention: persistent)_ — Record of moderation actions taken by bot or admins
  - fields: actor admin id, target user id, action type, reason, timestamp
- **Configuration** _(retention: persistent)_ — Bot settings and thresholds
  - fields: greeting text, rules text, confirmation timeout, spam thresholds, automatic actions, trusted users list, log retention period

## Integrations

- **Telegram** (required) — Bot API messaging and group management
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure greeting and rules text
- Set confirmation timeout
- Adjust spam thresholds and automatic actions
- Manage trusted user list
- View and export action logs
- Request moderation reports
- Enable/disable direct message notifications for admins

## Notifications

- In-group greetings and confirmation prompts
- In-group moderation explanations
- In-group admin summaries
- Optional direct-message reports to admins

## Permissions & privacy

- Only admins can access configuration and logs
- User data is stored only for moderation purposes
- Trusted user status is opt-in and revocable
- Action logs are retained for configured period (default 90 days)

## Edge cases

- User joins and leaves before confirmation
- Multiple spam triggers on single message
- Admin attempts to ban another admin
- Message contains both link and repeated content
- Trusted user violates rules
- Bot fails to process message during high load

## Required tests

- Verify new-member confirmation flow with timeout
- Test spam detection rules with various message patterns
- Validate admin command permissions and actions
- Confirm log retention and reporting accuracy
- Test trusted user exemptions
- Validate message rate limiting and flood detection

## Assumptions

- Telegram API will provide consistent user and message data
- Admins will configure thresholds appropriately for their group's needs
- Trusted users will be manually vetted by admins
- Group will maintain at least one admin with bot permissions
