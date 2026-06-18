# Field Notes: Agents, Teams, and Your Own Limits

Not a skill. This is the part of all of it that's about you and the people around you, not the code. Keep it where you'll reread it when a team starts wearing on you.

## You are a single-core processor

The cognitive load is the real constraint, not the typing. Watching CC run while you do nothing *feels* unproductive — and that feeling is the trap, because the watching is the job. Your supervision is the thing that catches the confident-wrong code an agent produces. The moment you split attention to feel busy, you degrade exactly the faculty that justified you being there.

So the urge to run a second session is usually not a throughput decision. It's the discomfort of looking idle. Name it as that. Real parallelism needs asymmetric depth — one task that needs you, one that doesn't. Two things that both need you isn't 2x; it's two queues you're now behind on.

Treat context-switching as your most expensive operation and guard single-threadedness like a resource. The dead time while an agent works is not waste to fill with another agent — it's time to write the next spec, read ahead, sharpen the thing you're already holding.

## The firefighter economy

Watch what your organization actually rewards. If it rewards the visible fix and ignores the invisible prevention, it is training people to manufacture fires they can heroically put out. The 2am firefighter is legible to a manager — dramatic, timestamped, obviously working hard. Your prevention is invisible: nobody can see the incident that *didn't* happen because the convention held. The incentive gradient literally points toward carelessness, and the careless collect the reputation.

This isn't cynicism. It's the predictable output of rewarding the wrong signal. Once you see it you can't unsee it, and you stop being surprised that the person who breaks prod at 5pm and fixes it at 1am is the one who looks committed.

## Stop being the shock absorber

Here's the part that costs *you*. When you silently refactor and clean up someone's careless push, you are subsidizing that exact incentive. You erase the signal that would have exposed how they work, and you convert your competence into their reputation. The arsonist keeps the credit because you quietly made the damage disappear.

So stop — not out of spite, out of reading the incentives correctly. Let messes be attributable. The person who broke it owns the fix, in daylight, on the record. That is not sabotage; it is removing yourself as the thing that absorbs the cost of other people's shortcuts. You are not their daycarer, and the free labor is the thing that makes the dysfunction sustainable.

## Don't generalize from n=1

One bad company — especially your first — is one data point. "Corporate is like this" is the easy, bitter generalization standing in for case-by-case judgment, and it's the same move as collapsing every edge case into one universal because the universal is easier to hold than the discrimination. You'd never reason that way when calm. The frustration is doing it for you.

The honest version: *some* places reward prevention, gate merges hard, and run blameless postmortems precisely so the firefighter pattern can't take root, and *some* don't — and you've worked with both, because you've also shipped happily with mindful people and built real mutual trust. A bad team is not a verdict on the category. Don't build a life decision — solo forever, teams are poison — on a sample size of one.

## Principle, not contempt

Separate two things that fuse when you're angry. "I won't pretend bad engineering is fine" is principle. Keep it, sharpen it, and don't let anyone reframe your standards as you being difficult. "These people are beneath me as people" is contempt — and contempt costs you and changes them zero.

The careless engineer is sometimes one honest explanation away from converting and sometimes not, and you can't tell which while you're pre-sorting everyone into the bin marked unworthy. So spend the energy you'd burn on contempt on the two things that actually move a person: enforcement (let the machine carry the rule, so you don't have to police it) and transmission (one real "why," once, on the record). If they convert, good. If they don't after that, it's a staffing decision, not a feud — and you stop stewing in a room you're trying to leave.

---

The throughline: the discipline that makes software outlive you and the discipline that keeps you sane are the same move. Make the right thing the path of least resistance, enforce it in the machine instead of with your vigilance, and refuse to personally absorb the cost of other people's shortcuts. The rest — the rage, the cleanup, the generalizations — just keeps you doing other people's night shifts.
