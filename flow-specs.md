# Power Automate Flow Specifications

Build all four as cloud flows in a Solution (so they export as one package). Use only approved connectors: Microsoft Forms, SharePoint, Outlook (Office 365), Teams, Approvals (optional). All are standard licence.

**Common governance pattern (all flows)**
- Wrap the main logic in a Scope named `Try`.
- Add a second Scope `Catch`, set *Configure run after* = **has failed** and **has timed out** on `Try`.
- In `Catch`: Outlook *Send an email (V2)* to the training lead with flow name, run URL (`@{workflow()['run']['name']}`), timestamp (`utcNow()`) and error (`result('Try')`).
- Use connection references and an environment variable for `TrainingLeadEmail` so nothing is hard-coded.
- Flow owner + one co-owner.

---

## Flow A: Registration

**Trigger:** Microsoft Forms. When a new response is submitted

| # | Action | Configuration |
|---|---|---|
| 1 | Forms: Get response details | Form Id + Response Id from trigger |
| 2 | SharePoint: Get items (Sessions) | Filter Query: `Title eq '@{outputs('Get_response_details')?['body/<SessionQuestionId>']}'`, Top Count 1 |
| 3 | Condition: session found | `length(outputs('Get_items')?['body/value'])` is greater than 0. No: email training lead "Unknown session", terminate |
| 4 | Compose: Capacity | `first(body('Get_items')?['value'])?['Capacity']` |
| 5 | Compose: SeatsTaken | `first(body('Get_items')?['value'])?['SeatsTaken']` |
| 6 | Condition: seat available | `SeatsTaken` is less than `Capacity` |

**Yes branch (Confirmed)**
1. SharePoint: Create item (Registrations): EmployeeName, Email, Department, SessionLookupId = session `ID`, Status = `Confirmed`.
2. SharePoint: Update item (Sessions): SeatsTaken = `add(int(outputs('SeatsTaken')),1)`.
3. Outlook: Create event (V4): Subject = course title, Start/End from `SessionDate`, Required attendees = participant email, Body includes `TeamsLink`.
4. Teams: Post message in a chat or channel (training channel): "{Name} registered for {Course} ({SeatsTaken+1}/{Capacity})". Avoid posting email addresses publicly.

**No branch (Waitlisted)**
1. SharePoint: Create item: Status = `Waitlisted`.
2. Outlook: Send an email (V2): waitlist confirmation, with note that they'll be contacted if a seat opens.

> Race-condition note: two simultaneous submissions can both pass the capacity check. Mitigation: set the trigger's **Concurrency control = On, degree 1** (Settings on the trigger). Document this in the SOP.

---

## Flow B: Daily reminders

**Trigger:** Recurrence. Daily, 09:00, time zone = India Standard Time (or your local one)

| # | Action | Configuration |
|---|---|---|
| 1 | Initialize variable `Tomorrow` | `addDays(utcNow(),1,'yyyy-MM-dd')` |
| 2 | SharePoint: Get items (Sessions) | Filter: `SessionDate ge '@{variables('Tomorrow')}T00:00:00Z' and SessionDate lt '@{addDays(utcNow(),2,'yyyy-MM-dd')}T00:00:00Z'` |
| 3 | Apply to each session | |
| 3a | SharePoint: Get items (Registrations) | Filter: `Status eq 'Confirmed' and SessionLookupId eq @{items('Apply_to_each')?['ID']}` |
| 3b | Apply to each registration | Outlook: Send email: reminder with course, time, `TeamsLink` |

Tip: `SessionDate` is stored in UTC, so compare using a date range as above rather than `eq`.

---

## Flow C: Post-session follow-up

**Trigger:** SharePoint. When an item is created or modified (Registrations)
Settings: **Trigger condition** `@not(equals(triggerOutputs()?['body/Status/Value'],'Confirmed'))` to cut unnecessary runs.

| # | Action | Configuration |
|---|---|---|
| 1 | Switch on `Status Value` | |
| Attended | Outlook: Send email | Thank-you + feedback Form link |
| No-show | SharePoint: Get items (Sessions) | Filter: next session for the same course, `SessionDate gt utcNow()`, order by SessionDate asc, top 1 |
| | Outlook: Send email to participant | "Sorry we missed you" + next available session |
| | Outlook: Send email to manager | `ManagerEmail` column (or Office 365 Users: Get manager V2) |
| Cancelled | SharePoint: Update item (Sessions) | SeatsTaken - 1. Optional: promote first Waitlisted registrant |

Prevent loops: do not update the Registrations item from within this flow (or use a "Processed" flag column).

---

## Flow D: Weekly summary

**Trigger:** Recurrence. Weekly, Monday 08:30

| # | Action | Configuration |
|---|---|---|
| 1 | Initialize `WeekStart` | `addDays(utcNow(),-7,'yyyy-MM-dd')` |
| 2 | SharePoint: Get items (Registrations) | Filter: `Created ge '@{variables('WeekStart')}T00:00:00Z'` |
| 3 | Filter array x3 | Status = Confirmed / No-show / Waitlisted (use `length()` of each) |
| 4 | Compose summary | Registrations, attended, no-shows, waitlist; no-show rate = `div(noShows, max(1, attended+noShows))` |
| 5 | Teams: Post message **or** Outlook: Send email | to training lead |

---

## Test checklist
- [ ] Register under capacity → Confirmed, SeatsTaken +1, invite + Teams post received
- [ ] Register at capacity → Waitlisted + waitlist email
- [ ] Set tomorrow's session → Flow B emails only Confirmed people
- [ ] Mark Attended / No-show → Flow C emails sent
- [ ] Break a connection → failure email arrives (Catch scope)
- [ ] Use only test data / dummy emails for screenshots
