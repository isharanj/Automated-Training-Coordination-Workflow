# Automated Training Coordination Workflow

Removes manual admin from training registrations, invites, reminders and follow-ups using approved Microsoft 365 tools: SharePoint, Microsoft Forms, Power Automate, Outlook and Teams.

> All data in screenshots and packages is dummy/test data. No real emails or tenant URLs are included.

## Problem
Training coordination was run by hand: read each form response, check capacity in Excel, update counts, send invites, send reminders, chase no-shows and compile reports. It is slow, error-prone (double-booking, missed reminders) and depends on one person.

## Solution
A SharePoint repository (Sessions and Registrations lists) fed by a Microsoft Form, with four Power Automate flows:

| Flow | Trigger | What it does |
|---|---|---|
| A. Registration | New Forms response | Checks capacity, confirms or waitlists, updates SeatsTaken, sends Outlook invite, posts to Teams |
| B. Reminders | Daily 9 AM | Emails Confirmed attendees of tomorrow's sessions with the Teams link |
| C. Post-session | Registration item modified | Attended: feedback Form. No-show: follow-up with next session and notify manager |
| D. Weekly summary | Monday | Counts registrations, no-shows, waitlist and reports to the training lead |

Governance: Try/Catch scopes with failure emails on every flow, concurrency control on Flow A, least-privilege permissions, retention rules, and an SOP.

## Architecture
![Process map](docs/process-map.svg)

```
Microsoft Form --> Flow A --> SharePoint (Sessions, Registrations)
                     |--> Outlook (invite / waitlist email)
                     |--> Teams (channel post)
SharePoint Sessions --> Flow B (daily) --> Outlook reminders
Registrations status change --> Flow C --> Outlook (feedback / follow-up / manager)
Registrations --> Flow D (weekly) --> Teams / email summary
```

## Before / After
| | Before | After |
|---|---|---|
| Manual steps per registrant | 12 | 1 (mark Attended / No-show) |
| Weekly reporting | Manual count and write-up | Automatic |
| Capacity errors | Possible | Checked on every registration |
| Error visibility | None | Failure email with run link |

**11 of 12 manual steps removed per registration (92%).** Full breakdown: [docs/measurement.md](docs/measurement.md).

## Repository contents
| Path | Content |
|---|---|
| `sharepoint/` | List schemas and PnP provisioning script |
| `forms/` | Registration and feedback form specs |
| `flows/flow-specs.md` | Step-by-step flow definitions, expressions, error handling, test checklist |
| `flow-packages/` | Exported flow packages (.zip) |
| `docs/` | Process map, step count, SOP (md and PDF), demo script |
| `screenshots/` | Screenshots of lists, flows and run history (test data only) |

## Demo
Video: _add link here (Clipchamp / Teams recording)_. Script: [docs/demo-script.md](docs/demo-script.md)

## How to reproduce
1. Create the SharePoint site and lists ([sharepoint/lists-schema.md](sharepoint/lists-schema.md), or run `provision.ps1`).
2. Create the Form ([forms/registration-form.md](forms/registration-form.md)).
3. Build flows A-D from [flows/flow-specs.md](flows/flow-specs.md) inside a Power Automate solution.
4. Test with the checklist, then export the solution and add it to `flow-packages/`.

## Tools
SharePoint Online, Microsoft Forms, Power Automate, Outlook, Teams.
