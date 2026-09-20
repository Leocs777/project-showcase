# Study Softtrack Case Study

Study Softtrack is a personal study planner for iPhone, iPad, and the web. It turns course work and self-study into an editable daily checklist, with time-aware planning, two native reminder channels, and recoverable file backups. The app is in private personal testing; it has not been released on the App Store.

The source repository remains private. This case study omits actual course details, personal schedules, learning records, device configuration, and account information.

## Product Summary

Learning plans often live separately from the time available to carry them out. Study Softtrack connects each task to a concrete next action: what to study, when it is due, which device it needs, and what counts as finished.

- **Daily execution:** editable tasks, completion checkboxes, session timers, and notes. Checking off a task does not invent time spent studying.
- **Course and self-study planning:** weekly organization, explicit deadlines, suggested study dates, and earlier material that can be marked complete manually.
- **Practical scheduling:** available time, device requirements, fixed classes, blackout dates, and locked plans constrain automatic scheduling.
- **Flexible reminders:** a task can use its own reminder time, inherit a task-type rule, or explicitly disable reminders. iOS notifications and Apple Reminders can be enabled separately.
- **Recoverable backups:** after choosing an iCloud Drive folder, saves update a device-specific JSON backup and retain the previous distinct snapshot. Import validates the file and previews the replacement before applying it.

## Architecture

```text
React + TypeScript interface and scheduling rules
                 |
       Small platform boundary
          /                 \
 Web browser             Capacitor iOS
 localStorage            Native Preferences
 Web Locks               Swift service bridge
 JSON import/export      |-- UserNotifications
                         |-- EventKit / Apple Reminders
                         `-- Files picker / iCloud Drive backup
```

One shared interface and domain model serve the web and iOS app. A small Swift bridge connects to platform services; there is no custom backend, account system, routing library, or global state library. The iOS build bundles its interface so the core workflow does not depend on a development server.

## Engineering Decisions

### Keep dates and completion honest

An exact class or deadline instant is separate from a suggested study date. Unknown deadline times stay unknown. Course imports use stable identifiers so repeated imports preserve edits and completion, while rescheduling protects past work, active sessions, and locked blocks.

### Make reminder precedence predictable

An individual override takes precedence over the task-type rule, including an explicit off choice. Permissions are requested when the user enables a service. Apple Reminders receives app-managed entries in a dedicated list; its integration is clearly labelled as one-way export.

### Preserve local data when integrations fail

The local save completes before reminder or backup work begins. Native service operations are serialized, and a service failure surfaces a warning without rolling back the saved learning record. Browser writes use locking and revision checks to reject stale overwrites.

### Keep backup distinct from synchronization

The app writes coordinated, atomic files in the user-selected folder and keeps the previous distinct version. iCloud handles upload. A successful local write is not presented as proof of cloud delivery, and backups do not silently merge data between devices. Restore validates the schema and preserves a recovery snapshot before replacement.

## UX Direction

The interface prioritizes today's checklist and one useful next task. Warm ivory, forest green, serif Chinese headings, open spacing, and a mobile bottom navigation keep the experience quiet and readable. Task editing, completion, and reminder controls stay close to the work they affect.

## Technical Stack

| Layer | Choices |
| --- | --- |
| Shared interface | React, TypeScript, Vite, CSS, Phosphor icons |
| iOS container | Capacitor, bundled web assets, Swift |
| Persistence | Native Preferences on iOS; localStorage and Web Locks on web |
| Native reminders | UserNotifications and EventKit |
| Backup and recovery | Files document picker, security-scoped folder access, coordinated JSON writes |
| Validation | Type checking, domain/storage tests, browser checks, physical-device checks |

## Verification and Current Limits

The current implementation passes 19 business, scheduling, and storage tests, four static-hosting packaging checks, TypeScript checks, and production builds. A development-signed iOS build was installed and launched on a physical iPhone.

Apple Reminders export was independently observed in its dedicated list. Notification receipt was confirmed by the tester. Actual latest and previous iCloud backup files were read on a second device; the latest matched the device data, and both passed the app's recovery parser without replacing live records.

The app remains a personal-use build. Web and iOS storage are separate, Apple Reminders edits do not flow back into the app, and iCloud backup is not live synchronization. Notification-tap navigation and completion/cancellation propagation have not yet received physical-device end-to-end verification; the backup check validated recovery parsing rather than replacing live device data.

## What I Owned

- Product scope, information architecture, and visual direction
- Shared task model, scheduling rules, editable course planning, and daily execution
- Responsive web interface and Capacitor iOS delivery
- Native reminder integration, persistence, backup, and recovery boundaries
- Automated checks, browser validation, and coordinated physical-device testing

## Safe Review Notes

This public page contains no private source, original course documents, personal progress, real task screenshots, signing configuration, or cloud backup paths. A guided demo with fictional data can be prepared separately.
