# Local Review: mobile/non-chat-push (2026-04-26)

## What was reviewed

Non-chat push notification support for the mobile app. The branch adds `queuePushNotification` — a new helper that gates on device registration and user preferences before creating a `PUSH` channel notification row. Every existing notification queue function (band invite, gig invite, gig update, gig RSVP, slot candidate, poll invite, poll response) was extended to call this helper alongside the existing email queue.

Also includes:
- `usePushNotifications.ts` cold-start + dedup fix (handles app opened from tap vs. live listener race)
- Unqueue functions extended to also remove pending PUSH rows when an invite is cancelled
- RICE field removal from `FeedbackReport` schema (unrelated to push, separate cleanup)
- `BandMembersPage` and `updateBandMemberRole` resolver changes

## Conclusion

**[BLOCK]** The push unqueue in `unqueueGigInviteMessages`, `unqueueGigSlotCandidateMessages`, and `unqueueGigUpdateMessages` removes push rows for the entire gig scope without userId filtering. The gig-scoped scope (`band:${bandId}:gig:${gigId}`) is shared across all members, so cancelling one member's invite wipes pending push notifications for other members too. Fix: use a member-specific scope for push rows, or add a userId filter to the unqueue call.

**[WARN]** No tests for the unqueue paths — a test would have caught the bug above.

**[INFO]** Minor: device check should run before preference check in `queuePushNotification`; `queueGigUpdateMessages` adds per-input queries (pre-existing pattern).

## What was excellent

- `push.notifications.queue.test.ts` is comprehensive and uses real DB — device gating, preference gating, email+push coexistence, payload shape, RSVP exclusion, leader exclusion all covered.
- `queuePushNotification` helper is clean and well-abstracted.
- Cold-start fix in `usePushNotifications.ts` with `handledIdentifier` guard is the correct approach.

## Review file

`giggable-worktree-chanthru-tron/review-mobile-non-chat-push.md`
