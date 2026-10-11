# SharePoint Repository Setup

**Site:** Training Team Repository (Team site, Microsoft 365 group-connected)

## List 1: Sessions

| Column | Type | Settings |
|---|---|---|
| Title | Single line of text | Course name. Must match the Forms choice text exactly |
| SessionDate | Date and time | Include time; required |
| Trainer | Person or Group | Required |
| Mode | Choice | Virtual / In-person / Hybrid |
| Capacity | Number | Whole number, min 1, required |
| SeatsTaken | Number | Whole number, default 0 |
| TeamsLink | Hyperlink | Meeting URL (blank for in-person) |

Optional helper column: `SeatsLeft` (Calculated) = `=[Capacity]-[SeatsTaken]`

## List 2: Registrations

| Column | Type | Settings |
|---|---|---|
| EmployeeName | Single line of text | Required |
| Email | Single line of text | Required |
| Department | Single line of text (or Choice) | Required |
| Session | Lookup | Source list: Sessions, column: Title |
| Status | Choice | Confirmed / Waitlisted / Attended / No-show / Cancelled. Default: Confirmed |

Optional: `ManagerEmail` (Single line of text) is used by Flow C to notify the manager on no-shows.

> Note: Power Automate writes to a Lookup column using the **Id** of the Sessions item (field `SessionLookupId`).

## Libraries
- **Training Materials** (document library): one folder per course.
- **SOP** (folder inside a library, or its own library): holds the SOP PDF and version history.

## Permissions (governance)
| Group | Level | Who |
|---|---|---|
| Training Owners | Full control | Training lead + 1 backup |
| Training Members | Edit | Trainers / coordinators |
| Training Visitors | Read | Managers, wider team |

Registrations contains personal data, so break inheritance on that list: Owners and Members only.

## Quick provisioning (optional)
Use `provision.ps1` (PnP.PowerShell) if your tenant allows it; otherwise create the lists manually with the tables above.
