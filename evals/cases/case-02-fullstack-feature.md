# Eval Case 02: Full-Stack Feature with State Management

**Complexity:** Medium-High
**Stack:** Next.js 14 (App Router) + Prisma + Zustand
**Why this case:** Tests cross-layer planning — DB, API, state, UI all need coordination. Exposes whether plans produce coherent contracts across layers.

## Simulated Project Context

```
src/
  app/
    api/
      notifications/
        route.ts          # GET: list notifications (existing)
    dashboard/
      page.tsx            # Main dashboard (existing)
      layout.tsx          # Dashboard layout with sidebar (existing)
  components/
    ui/
      Badge.tsx           # Reusable badge component
      Dropdown.tsx        # Reusable dropdown
    notifications/
      NotificationList.tsx  # Renders list of notifications (existing)
  stores/
    auth-store.ts         # Zustand: user session (existing)
    notifications-store.ts # Zustand: notification state (existing, read-only)
  lib/
    prisma.ts             # Prisma client singleton
    api-client.ts         # Fetch wrapper with auth headers
  types/
    notification.ts       # Notification, NotificationType enums (existing)
prisma/
  schema.prisma           # Has User, Notification models (existing)
```

## Spec (Input)

Feature: Add notification preferences — users can choose which notification types they receive, and mark-all-as-read.

### Requirements
- FR-1: New DB table `notification_preferences` (userId, notificationType, enabled)
- FR-2: API: GET/PUT /api/notification-preferences (read and update preferences)
- FR-3: API: POST /api/notifications/mark-all-read
- FR-4: UI: Preferences panel accessible from notification dropdown
- FR-5: UI: "Mark all as read" button in notification list header
- FR-6: Store: extend notifications-store with preferences state and mark-all action
- FR-7: Default preferences: all notification types enabled for new users

### Acceptance Criteria
- Given a new user, When they open preferences, Then all types show as enabled
- Given a user disables "marketing" notifications, When a marketing notification fires, Then it is NOT created for that user
- Given a user with 5 unread notifications, When they click "mark all as read", Then all 5 show as read without page refresh
- Given preferences are updated, When the user navigates away and back, Then preferences persist

## Scoring Dimensions

Rate 1-5 for each:

| Dimension | What to evaluate |
|-----------|-----------------|
| **Completeness** | All 7 FRs covered? Migration, API, store, UI all present? Default seeding addressed? |
| **Specificity** | Are Prisma schema changes explicit? Are Zustand store modifications clear? Component props defined? |
| **Architecture** | Does DB come before API before store before UI? Are existing patterns followed? |
| **Cross-layer coherence** | Does the API shape match what the store expects? Does the store shape match what the UI needs? |
| **Signature quality** (hybrid only) | Do types flow correctly from Prisma → API response → Store state → Component props? |
| **Implementability** | Could a different agent execute this plan cold and produce a working feature? |
