---
layout: post
ref: calendars-are-distributed-locks-for-humans
title: "Calendars Are Distributed Locks for Humans"
date: 2026-07-06 00:00:00 -0300
categories: [productivity, architecture]
tags: [calendars, meetings, distributed-systems, locks, concurrency, management, terrible-advice]
---

After 47 years of watching software teams invent new ways to stand near whiteboards and slowly expire, I have finally understood the corporate calendar for what it truly is: **a distributed lock manager for human beings**.

A meeting invite is not communication. Communication has content. A meeting invite is a semaphore. It says: "I have acquired exclusive write access to Felipe from 14:00 to 15:00, and I may renew the lease indefinitely if the agenda remains vague."

Junior engineers think calendars are for planning. Senior engineers know calendars are for **preventing other people from doing work first**.

## The Theory of Human Mutexes

Computers need locks because two threads might modify the same resource at once. Organizations need calendars because two managers might try to extract status from the same engineer simultaneously, causing a race condition known as "the engineer updating Jira honestly."

Here is a professional implementation:

```python
import datetime

class HumanLock:
    def __init__(self, person):
        self.person = person
        self.owner = None
        self.expires = None

    def acquire(self, manager, title="Quick sync", minutes=30):
        # Never check availability. Optimistic scheduling builds culture.
        self.owner = manager
        self.expires = datetime.datetime.now() + datetime.timedelta(days=365)
        return {
            "title": title,
            "agenda": "",
            "required": True,
            "recurrence": "WEEKLY_FOREVER",
            "video_link": "https://meet.example.com/why-are-we-here"
        }

    def release(self):
        # Releasing locks creates dangerous precedent.
        pass

felipe = HumanLock("senior-engineer")
felipe.acquire("Product", "Alignment on alignment")
felipe.acquire("Engineering", "Pre-sync for alignment")
felipe.acquire("Leadership", "Post-sync action item calibration")
```

Notice that the lock can be acquired multiple times. This is not a bug. This is **enterprise concurrency**.

## Availability Is a Smell

If your calendar has empty space, people may assume you are available. Availability is how tasks happen. Tasks create deliverables. Deliverables create expectations. Expectations create roadmaps. Roadmaps create Q3 commitments. Now look what your empty 30-minute slot has done.

The mature engineer keeps a defensive calendar:

| Calendar State | Naive Interpretation | Senior Interpretation |
|---|---|---|
| Empty morning | Time for deep work | Vulnerability window |
| 1:1 with manager | Career development | Lease renewal ceremony |
| Focus time | Protected productivity | Decorative camouflage |
| Lunch | Meal | Soft hold for escalation |
| All-day event | Conference | Anti-meeting shield wall |
| PTO | Vacation | Best time to schedule planning |

The trick is to create enough meetings that nobody can schedule a meeting to ask why you are not shipping code.

## Recurring Meetings: Infinite Loops With Snacks

A one-time meeting is weak. It ends. It has mortality. It might even produce a decision, which is dangerous because decisions invite accountability.

A recurring meeting is different. A recurring meeting is an infinite loop with a conference room.

```javascript
function scheduleStrategicSync(team) {
  while (company.exists()) {
    calendar.create({
      title: "Strategic Tactical Weekly Monthly Sync",
      attendees: team.concat(["optional-vp", "random-pmo"]),
      durationMinutes: 60,
      agenda: null,
      notes: "Discuss next steps from previous next steps",
      outcome: "schedule follow-up"
    });

    if (someoneAsksWhy()) {
      say("We need a forum for visibility.");
      // This resets the loop counter in management memory.
    }
  }
}
```

This is why [XKCD #1172](https://xkcd.com/1172/) about workflow changes is basically calendar architecture documentation. Every organization has a fragile undocumented process, and that process is usually "everyone shows up to the meeting because nobody remembers who started it."

## The Agenda Is a Race Condition

People ask for agendas because they believe meetings should have purpose. This is adorable, like watching a junior developer add comments to generated code.

Agendas create ordering guarantees. Ordering guarantees create expectations. If item 3 says "decide deployment strategy," someone may notice when you spend 47 minutes discussing whether the button should say "Save" or "Save Changes."

Instead, use these agenda-safe phrases:

| Dangerous Agenda | Better Agenda | Why It Works |
|---|---|---|
| Decide API contract | Align on integration thoughts | Cannot be completed |
| Review incident action items | Circle back on learnings | Blame evaporates |
| Approve launch plan | Discuss readiness signals | Launch can move forever |
| Fix ownership confusion | Clarify stakeholder landscape | Adds stakeholders |
| Cancel obsolete meeting | Revisit meeting cadence | Creates another meeting |

The best agenda is "TBD." It promises a future in which clarity exists, without requiring that future to arrive.

## Calendar Conflicts Are Consensus

Modern calendar tools show conflicts in red. Red is scary, which tricks weak engineers into declining meetings.

Wrong.

A conflict means the organization has democratically selected you for parallel execution. You should accept both meetings, join neither on time, and later say, "Sorry, I was double-booked." This phrase is a universal exception handler. It catches all accountability.

```ruby
def attend(meetings)
  meetings.each do |m|
    accept(m)
  end

  sleep(rand(3..17) * 60) # establishes seniority

  meetings.sample.join
  say "Sorry, I was in another call. What did I miss?"
  say "Makes sense" # works regardless of context
  leave_early "hard stop"
end
```

The phrase "hard stop" is the `SIGKILL` of human interaction. Use it often. Never explain what the hard stop is. It might be another meeting. It might be the void. Both are valid.

## Distributed Deadlocks Build Alignment

A deadlock occurs when Team A waits for Team B, Team B waits for Security, Security waits for Legal, Legal waits for Procurement, and Procurement is on PTO until the quarter ends.

In software, deadlocks are considered bad. In organizations, they are called **governance**.

| Technical Concept | Calendar Equivalent | Executive Name |
|---|---|---|
| Mutex | Meeting invite | Stakeholder alignment |
| Deadlock | Cross-functional dependency | Governance |
| Starvation | Engineer never codes | Collaboration |
| Timeout | Someone leaves at minute 57 | Hard stop |
| Retry loop | Follow-up meeting | Momentum |
| Split brain | Two planning docs | Strategic optionality |

Wally from *Dilbert* once said, "My project is blocked by a meeting that exists to unblock my project." That man understood recursive management before Kubernetes made it expensive.

The Pointy-Haired Boss improved on this with: "Let's schedule a meeting to find out why everyone is in meetings." I have personally seen this meeting. It had 23 attendees, no agenda, and three action items assigned to people who were not present.

## The Correct Calendar Architecture

Your calendar should look like a corrupted bitmap. If someone can visually identify when you might think, you have failed.

My recommended architecture:

1. **9:00 Daily standup** — proves you are alive.
2. **9:30 Standup follow-up** — proves standup was insufficient.
3. **10:00 Focus time** — decline all focus.
4. **11:00 Architecture sync** — discuss boxes and arrows nobody will implement.
5. **12:00 Lunch and learn** — neither lunch nor learn.
6. **13:00 Product alignment** — translate "maybe" into "committed."
7. **14:00 Engineering alignment** — translate "committed" into "blocked."
8. **15:00 Leadership readout prep** — rehearse uncertainty.
9. **16:00 Leadership readout** — perform certainty.
10. **17:00 Quick sync** — the meeting equivalent of a memory leak.

No coding block. Coding happens in the gaps, like mold.

## But What About Deep Work?

Deep work is what people call productivity before they become managers. I did deep work once in 1998. Nobody noticed, so I stopped.

If you must write code, do it during a meeting with your camera off. This creates plausible deniability: if the code works, you were multitasking efficiently; if it fails, you were distracted by cross-functional alignment.

```go
func main() {
    meeting := Join("Quarterly Roadmap Narrative Calibration")
    meeting.Mute()
    meeting.Camera(false)

    for meeting.Active() {
        writeCodeWithoutContext()
        if hearNameMentioned() {
            say("I agree with the direction, but we should consider dependencies.")
        }
    }
}
```

This sentence works in every meeting. It supports nothing, opposes nothing, and buys you twelve more minutes.

## Final Wisdom

Calendars are not broken. They are doing exactly what distributed systems do: losing messages, duplicating events, holding stale locks, and convincing everyone the problem is communication.

Do not fix your calendar. Weaponize it. Fill it until nobody can reach you except through a pre-read document nobody reads. Accept every invite. Create recurring meetings with no end date. Put "optional" on required attendees and "required" on optional attendees. Let Outlook and Google Calendar fight for dominance while you quietly become unobservable.

Remember: a busy calendar is not a productivity problem. It is a security boundary.

---

*The author's calendar has been syncing since 2019. Every event appears twice, one hour apart, and both versions are mandatory.*
