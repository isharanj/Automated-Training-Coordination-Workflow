# Before vs After: Step Count

## "Before" (manual process), per registrant lifecycle

| # | Manual step | Automated by |
|---|---|---|
| 1 | Open and read the new form response | Flow A |
| 2 | Open the Excel tracker and find the session | Flow A |
| 3 | Check capacity (seats taken vs capacity) | Flow A |
| 4 | Add the registrant as a new row | Flow A |
| 5 | Update the seats-taken count | Flow A |
| 6 | Create the calendar invite with Teams link and add the attendee | Flow A |
| 7 | Write and send confirmation (or waitlist) email | Flow A |
| 8 | Post update in the Teams training channel | Flow A |
| 9 | Day before: check which sessions are tomorrow, email each attendee a reminder | Flow B |
| 10 | After session: update attendance in Excel | **Still manual** (change Status in SharePoint, 1 click) |
| 11 | Send feedback link (attended) or follow-up with next session (no-show) | Flow C |
| 12 | Notify manager of no-shows | Flow C |

**Total manual steps before: 12 per registrant**
**Remaining manual after: 1** (mark Attended / No-show)
**Steps removed: 11 (92%)**

## Recurring admin (per week)
| Step | Before | After |
|---|---|---|
| Count registrations, no-shows, waitlist from Excel | Manual | Flow D |
| Write and send/post summary | Manual | Flow D |
| **Removed** | 2 | 0 |

## Optional: time saved (replace with your own measured numbers)
Assumption (illustrative only, validate by timing yourself): ~10 minutes of admin per registrant manually vs ~1 minute after. For 100 registrants/month that is roughly 15 hours saved/month. Only put a time figure on your CV if you can defend how you measured it; the step count above is directly verifiable from the process map.

## Resume bullet options
- Designed and documented a Microsoft 365 (SharePoint, Forms, Power Automate, Outlook, Teams) training coordination workflow that removed 11 of 12 manual steps per registration (92%), including automated confirmations, waitlisting, reminders and no-show follow-up.
- Built 4 Power Automate flows with error-handling (Try/Catch scopes, failure alerts) and wrote an SOP covering ownership, permissions and data retention so the process can be maintained by others.
