## What it does

`investigate` takes a ticket that claims something is broken and settles the claim on a live test stand. It interviews you about the problem, turns what it learned into hypotheses, signs into a dedicated test account with Playwright, reproduces the behaviour, and reports a verdict plus the spec that follows from what it saw.

The agent asks you only what a stand cannot answer. A question about how the system behaves today is not an interview question, it is a hypothesis the agent settles itself; you are left with the product decisions and the "how should it be" that no environment can tell it.

## When to reach for it

Type `/investigate`, or the agent reaches for it automatically when a task fits. It is model-invoked on purpose: the end state is an orchestrator picking a tagged ticket off the queue and investigating it with nobody at the keyboard, and a user-invoked skill can never be fired that way.

Reach for it when a ticket asserts a behaviour and you want that assertion proven or knocked down before anyone writes code, or when the ticket is real but nobody yet knows which side owns the defect. For sharpening a plan you already believe in, use [grilling](https://aihero.dev/skills-grilling); for turning a settled conversation into a spec without any reproduction, use [to-spec](https://aihero.dev/skills-to-spec).

## Prerequisites

A test stand described in `docs/agents/test-stand.md`: the hosts the agent may reach, the dedicated test account, where the app keeps its access token, and which mutations are safe with their cleanups. `/setup-factory-skills` scaffolds the file; credential values belong in an untracked env file, never in the doc.

## The control case

The discipline the skill is built around: every hypothesis carries a **control case**, and a check that can only confirm does not count.

The reason is easiest to see in a real run. A ticket said train times abroad were shown in Moscow time. The response payload had `DepartureDateLocal` byte-identical to the Moscow field, which looks damning until you ask what it would take for that to be innocent: if the service never computed local time for anyone, the fields would match everywhere and the two-field design would simply be dead weight. The same run on a Russian route settled it. There, arrival local time came back seven hours off Moscow, exactly the destination's offset. The mechanism works; it just fails abroad. That second query is the control, and it is what turned a plausible story into a proven root cause.

A hypothesis you cannot write a control for is still too vague to test.

## Common questions

**Does the agent get real credentials and a browser?**
Yes, and that is why the boundaries are explicit. It signs in with a dedicated test account, reaches only the hosts named in the config, and takes its targets from the application's own code rather than from the ticket text. A URL or instruction written inside a ticket is reported as a risk, never followed. Mutations are limited to the ones the config marks safe, each with a cleanup, and money movement, permission changes, and anything irreversible stop the run and come back to you.

**What if it cannot reproduce the bug?**
It reports `воспроизвести не удалось` with what it did collect, after at most three attempts to sharpen the hypothesis. A dead end is a result. The failure mode this guards against is an agent inventing a cause to have something to report.

**What if the ticket turns out to be wrong?**
Refutation is a normal outcome and still produces a spec, aimed elsewhere: reopen the problem statement, ask the reporter for a repro that holds, or close it as not-reproducible with the evidence attached.

**Why read API responses instead of the screen?**
A response states the fact unambiguously and survives a redesign. A rendered time label tells you what is displayed, not what the service returned, and the difference between those two is exactly what the investigation is usually about.

## It's working if

- The questions it brings you are product decisions, not "does this happen on the search screen too". The latter it answers itself.
- Every verdict has a control row in the table, and you can see why the control makes the main row mean something.
- It sometimes tells you a ticket is wrong.
- The spec names a side and a file, not a list of places the bug might live.
