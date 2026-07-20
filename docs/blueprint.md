# Telegram Group Guard — Bot specification

**Archetype:** community

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Automated moderation bot for Telegram groups that handles new member verification, spam detection, and provides admin controls with action logs and statistics. The bot verifies new members with a single-tap button, applies configurable spam rules, and provides transparency for all actions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Telegram group owners
- Telegram group admins

## Success criteria

- 90% of new members complete verification within 3 minutes
- 80% reduction in spam messages reported by admins
- Admins can review action logs and statistics within 2 clicks

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **/setwelcome** (command, actor: admin, command: /setwelcome) — Edit welcome message and rules
- **/setverifytimeout** (command, actor: admin, command: /setverifytimeout) — Set verification timeout (minutes)
- **/setthresholds** (command, actor: admin, command: /setthresholds) — Configure spam thresholds and map to actions
- **/trust add** (command, actor: admin, command: /trust add) — Mark a user as trusted
- **/trust remove** (command, actor: admin, command: /trust remove) — Unmark a trusted user
- **/warn** (command, actor: admin, command: /warn) — Issue a manual warning to a user
- **/mute** (command, actor: admin, command: /mute) — Mute a user for a specified duration
- **/kick** (command, actor: admin, command: /kick) — Remove a user from the group
- **/ban** (command, actor: admin, command: /ban) — Ban a user from rejoining the group
- **/log** (command, actor: admin, command: /log) — View the action log
- **/stats** (command, actor: admin, command: /stats) — View statistics for the group
- **I'm human** (button, actor: user, callback: verify:human) — Allow verified users to lift message restrictions

## Flows

### New Member Verification
_Trigger:_ user joins group

1. Send welcome message with 'I'm human' button
2. Restrict user from sending messages
3. Wait for verification button tap or timeout
4. If verified: lift restrictions and post confirmation
5. If timeout: remove user and post explanation

_Data touched:_ Member, Welcome configuration

### Spam Detection and Response
_Trigger:_ user sends message

1. Check message against spam rules
2. If spam detected: apply configured action (warn/mute/kick/ban)
3. Post explanation message in group
4. Log action in action log

_Data touched:_ Member, Moderation ruleset, Action log entry

### Admin Command Handling
_Trigger:_ /admin command

1. Verify user is admin
2. Execute command (setwelcome, setthresholds, etc.)
3. Post confirmation message
4. Log action in action log

_Data touched:_ Group, Welcome configuration, Moderation ruleset, Action log entry

### Action Log Viewing
_Trigger:_ /log command

1. Verify user is admin
2. Display requested log entries
3. Allow filtering by type/user

_Data touched:_ Action log entry

### Statistics Viewing
_Trigger:_ /stats command

1. Verify user is admin
2. Display aggregated statistics
3. Allow period selection

_Data touched:_ Statistics

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Group** _(retention: persistent)_ — Telegram chat where the bot is installed and configured
  - fields: id, config, stats
- **Member** _(retention: persistent)_ — Telegram user who joins the group
  - fields: id, username, join_timestamp, verification_status, trust_flag, admin_flag
- **Welcome configuration** _(retention: persistent)_ — Editable welcome message and rules
  - fields: message, rules, verification_timeout, verification_required
- **Moderation ruleset** _(retention: persistent)_ — Spam detection thresholds and actions
  - fields: link_account_age_threshold, repeat_message_threshold, flood_message_threshold, actions
- **Action log entry** _(retention: persistent)_ — Record of automated or admin actions
  - fields: timestamp, actor, target_user, action, reason, duration
- **Statistics** _(retention: persistent)_ — Counts over time for joins, verifications, and actions
  - fields: joins, successful_verifications, timed_out_verifications, warnings, mutes, kicks, bans, false_positives

## Integrations

- **Telegram** (required) — Bot API messaging and group management
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Edit welcome message and rules
- Configure verification timeout
- Set spam thresholds and actions
- Mark/unmark trusted users
- View action logs
- View statistics
- Enable/disable automated moderation actions

## Notifications

- Public explanations for automated actions in group chat
- Admin notifications for high-severity events (removals, bans)

## Permissions & privacy

- Only admins can access action logs and statistics
- Automated actions never target admins or pinned messages
- Private details are sent to admins via direct message when configured

## Edge cases

- User joins and leaves before verification timeout
- Multiple spam signals trigger in sequence
- Admin tries to ban another admin
- Verification button tap after timeout has expired

## Required tests

- Verify new member verification flow with timeout
- Test spam detection with various patterns
- Validate admin command execution and logging
- Confirm action log filtering and statistics display

## Assumptions

- Verification timeout is set to 3 minutes by default
- Spam thresholds are set to default values
- Trusted users are exempt from all automated actions
- Action log retention is set to 500 entries or 90 days
