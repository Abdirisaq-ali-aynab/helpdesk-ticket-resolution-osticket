# Help Desk Ticket Resolution – "Print Issues" (osTicket)

End-to-end walkthrough of working a real help desk ticket in **osTicket**: triaging the request, documenting work with internal notes, troubleshooting on the server, finding the root cause in the Windows event logs, fixing the issue, updating the user, and closing the ticket with resolution notes.

> **Skills shown:** ticketing system workflow · triage · internal documentation · Windows Services · Event Viewer log analysis · root-cause analysis · customer communication · ticket closure

---

## 🎫 Ticket Summary

| Field | Value |
|---|---|
| Ticket # | **883654** |
| Subject | Print Issues |
| Requester | Good User (`gooduser@goodcorp.local`) |
| Department / Help Topic | Support / Report a Problem |
| Priority / SLA | Normal / Default SLA |
| Assigned to | Good Helper (help desk agent) |
| Environment | `goodcorp.local` Active Directory domain, print server role on the DC |
| **Root cause** | Print Spooler service set to **Disabled** by `GOODCORP\Administrator` (Event ID 7040) |
| **Resolution** | Print Spooler startup type set back to **Automatic** and service started |
| Final status | **Resolved → Closed** |

---

## 1. Triage – Read the Ticket

Logged in to the osTicket agent panel as the help desk agent and opened ticket **#883654**. Reviewed the header details (status, priority, requester, source, SLA) and then the ticket thread.

> *"I recently uninstalled some software I was using that handled print jobs but now nothing's working when I try to print."*

**First read:** the user thinks they broke printing by uninstalling software. Printing goes through the print server, so the first suspect is the **Print Spooler service**.

<p>
<img src="images/01-ticket-883654-overview.png" width="49%" alt="Ticket 883654 overview in osTicket">
<img src="images/02-ticket-thread-user-report.png" width="49%" alt="Ticket thread showing the user's report">
</p>

*Left: ticket details. Right: the user's description of the problem.*

---

## 2. Document the Plan – Internal Note

Before touching anything, I logged an **internal note** (not visible to the user) saying what I was about to check. That way other technicians can see the ticket's progress.

- **Title:** Going to inspect print service on DC
- **Note:** Likely print service failure.

<p>
<img src="images/03-post-internal-note-button.png" width="49%" alt="Post Internal Note button">
<img src="images/04-internal-note-plan.png" width="49%" alt="Internal note describing the plan">
</p>

---

## 3. Troubleshoot – Check the Service

In an enterprise network, clients print through a central **print server**, which here is the Domain Controller. I opened **Services** (`services.msc`) on the DC and found **Print Spooler** with startup type **Disabled** and not running.

<p>
<img src="images/05-services-console-dc.png" width="49%" alt="Services console on the DC">
<img src="images/06-print-spooler-disabled.png" width="49%" alt="Print Spooler service shown as Disabled">
</p>

Finding the broken setting isn't enough. I still needed to know **when** it changed and **who** changed it.

---

## 4. Investigate – Find the Root Cause in Event Viewer

Service configuration changes are written to the **System** log. Event ID **7040** = *"The start type of the X service was changed from A to B."*

1. **Event Viewer → Windows Logs → System**
2. **Filter Current Log** → Event ID `7040`
3. **Find** → `print`

<p>
<img src="images/07-event-viewer-system-log.png" width="49%" alt="Event Viewer System log">
<img src="images/08-filter-event-id-7040.png" width="49%" alt="Filtering the System log for Event ID 7040">
</p>
<p>
<img src="images/09-find-print.png" width="49%" alt="Searching filtered events for 'print'">
<img src="images/10-event-7040-spooler-disabled.png" width="49%" alt="Event 7040: Print Spooler changed from auto start to disabled">
</p>

**Finding:**

| Field | Value |
|---|---|
| Event ID | 7040 (Service Control Manager) |
| Logged | 10/16/2024 1:10:56 AM |
| Message | *The start type of the Print Spooler service was changed from auto start to disabled.* |
| User | **GOODCORP\Administrator** |
| Computer | DC.goodcorp.local |

✅ **Root cause:** the outage was **not** caused by the user's uninstall. An administrator account disabled the Print Spooler service.

<p>
<img src="images/11-changed-by-administrator.png" width="49%" alt="Event details showing GOODCORP\Administrator made the change">
<img src="images/12-copy-event-details.png" width="49%" alt="Copy Details as Text from Event Viewer">
</p>

---

## 5. Record the Evidence on the Ticket

Copied the event (**Copy → Copy Details as Text**) and posted it as an internal note titled **"Print service was disabled by Administrator"**. The evidence now stays with the ticket for audits, follow-up with the admin team, and trend reporting on recurring causes.

<p>
<img src="images/13-internal-note-with-log.png" width="49%" alt="Internal note containing the event log details">
<img src="images/14-internal-note-posted.png" width="49%" alt="Internal note posted to the ticket">
</p>

---

## 6. Resolve – Fix the Service

**Services → Print Spooler → Properties → Startup type: Automatic → Apply → Start.**

> ℹ️ *Automatic (Delayed Start)* returns an error for Print Spooler. Windows doesn't allow delayed start for this service, so use **Automatic**.

PowerShell equivalent:
```powershell
Set-Service -Name Spooler -StartupType Automatic
Start-Service -Name Spooler
Get-Service -Name Spooler   # Status: Running
```

<p>
<img src="images/15-spooler-properties.png" width="49%" alt="Opening Print Spooler properties">
</p>

---

## 7. Communicate – Reply to the User

Sent a reply to the requester so they know it's fixed:

> *Hello gooduser, we've gone ahead and restarted the print service. Sorry about that! Looks like it got set to disabled.*

<p>
<img src="images/16-reply-to-user.png" width="49%" alt="Composing the reply to the user">
<img src="images/17-reply-posted.png" width="49%" alt="Reply posted in the ticket thread">
</p>
<p>
<img src="images/18-reply-posted-banner.png" width="49%" alt="Reply posted successfully banner">
</p>

---

## 8. Close the Ticket

Changed the status from **Open → Resolved** and filled in the closure notes with the **cause** (Print Spooler disabled by an admin, event log attached to the ticket) and the **fix** (service set back to automatic start).

The ticket moved into the **Closed** queue. Closed tickets are kept, not deleted, so there's a full history and the ticket can be reopened if the problem comes back.

<p>
<img src="images/19-set-status-resolved.png" width="49%" alt="Setting the ticket status to Resolved">
<img src="images/20-close-ticket-resolution-notes.png" width="49%" alt="Close ticket dialog with resolution notes">
</p>
<p>
<img src="images/21-status-changed-resolved.png" width="49%" alt="Status changed to Resolved confirmation">
<img src="images/22-closed-tickets-queue.png" width="49%" alt="Closed tickets queue">
</p>
<p>
<img src="images/23-closed-ticket-history.png" width="49%" alt="Ticket 883654 listed under closed tickets">
</p>

---

## 🕒 Ticket Timeline

| When | Action |
|---|---|
| 10/16/2024 01:10 | `GOODCORP\Administrator` changes Print Spooler from Automatic to **Disabled** (Event 7040) |
| 11/29/2024 07:45 | User submits ticket #883654 "Print Issues" via web portal |
| 11/29/2024 07:46 | Ticket transferred to **Support** department |
| Work session | Agent posts internal note: plan to inspect print service on DC |
| Work session | Print Spooler found **Disabled** in Services on the DC |
| Work session | Event Viewer → System → Event 7040 identifies who and when |
| 12/04/2024 02:19 | Internal note with event log evidence posted |
| 12/04/2024 | Print Spooler set to **Automatic** and started |
| 12/04/2024 02:22 | Reply sent to user confirming the fix |
| 12/04/2024 02:26 | Ticket set to **Resolved** with closure notes, moves to Closed queue |

---

## 💡 Takeaways

- **Don't trust the first explanation.** The user blamed their uninstall; the logs showed an admin change was the real cause.
- **Document as you go.** Internal notes and attached evidence make the ticket useful to the next technician and to management reporting.
- **Fix it, then find out why.** A follow-up with the admin team about why the spooler was disabled stops it from happening again.
- **Close the loop.** Update the user in plain language, then resolve with clear notes about cause and fix.

### Useful commands for this kind of ticket
```powershell
# Check the spooler on a print server
Get-Service -ComputerName DC -Name Spooler | Select Name, Status, StartType

# Who changed a service's startup type?
Get-WinEvent -ComputerName DC -FilterHashtable @{LogName='System'; Id=7040} |
  Where-Object Message -match 'Print Spooler' |
  Select TimeCreated, @{n='User';e={$_.UserId.Translate([System.Security.Principal.NTAccount])}}, Message
```

---

*Lab from Level Effect SOC100-1 (HelpDesk Analyst & Troubleshooting: "Lab – Resolving Your First Ticket"). Lab guide and screenshots from the course material; write-up by me.*
