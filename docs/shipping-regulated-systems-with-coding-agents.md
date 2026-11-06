# Shipping regulated systems with coding agents

**Multi-tenancy, mutual TLS to government APIs, and where the agent needs a hard boundary**

Luciano Manetta

---

Most writing about AI coding agents is set in a forgiving environment. A side project, an internal tool, a greenfield service where the cost of being wrong is an hour of rework.

The systems I build are not in that environment. They operate in Brazilian tax, labour and judicial compliance. They hold other people's digital certificates. They talk to federal government APIs, SERPRO among them, under mutual TLS. A number that comes out of them can end up in a statutory filing, and when it does, someone has to be able to show where it came from, which rule produced it, and who approved it.

I still build them through coding agents. This is what that actually requires.

## What regulated changes, and what it does not

It is worth being precise, because the word "regulated" gets used to justify a lot of ceremony that buys nothing.

What does not change: the agent is still very good at implementation. It reads an unfamiliar module faster than I do. It writes the data access layer, the API client, the validation, the tests, the migration. Refusing that leverage because the domain is serious would be a decision to work slower for no gain in safety.

What does change is the set of things that cannot be decided during implementation.

In a forgiving environment, an agent that makes a reasonable architectural call mid-task is being helpful. In a regulated one, several categories of decision are not reasonable to make mid-task at all, because getting them wrong is not a bug. It is a breach, a mis-filing, or a number nobody can defend.

So the discipline is not "supervise the agent more". It is to move a specific set of decisions out of the agent's reach entirely, before any code is written.

## The decisions that leave the agent's reach

Five, in my systems.

**Tenant isolation.** Multi-tenancy enforced at the row level, in the database, not in application code. This matters because application-level filtering is one forgotten `where` clause away from one customer reading another customer's fiscal data. An agent writing a new query under time pressure is exactly the circumstance in which that clause gets forgotten. Push the boundary down to where it cannot be skipped by omission, and the agent's mistake becomes an empty result instead of a disclosure.

**Authentication posture.** Multi-factor, with the session and credential model fixed as part of the architecture. Not a thing to be designed while building the feature that needs it.

**The audit trail.** What happened, when, to which tenant's data, under whose authority. Designed once, written to by everything, never optional for a particular code path because that path was inconvenient. An audit trail with gaps is worse than none, because it invites a confidence it has not earned.

**Credential custody.** Customers' digital certificates are held on their behalf. The custody model, the encryption posture, and the rules about what may ever touch a certificate in the clear are architecture, decided up front, with the blast radius understood.

**Who approves what.** The line between what the system may do on its own and what requires a qualified human to say yes. In this domain that line is not a product preference. It is the difference between automation and unauthorised action.

None of these five are things I ask an agent to decide. They are written down, they are the environment the agent works inside, and the acceptance criteria for every task reference them.

## Specification as the real work

The consequence of the above is that my work moved upstream. I write the specification, and the specification is where the engineering judgement now lives.

For each task: the architecture it fits into, the data model it may touch, the security posture it must preserve, and one acceptance test that defines done. The agent implements against a target it did not choose.

This sounds like overhead and is the opposite. A vague instruction to a capable agent produces a large amount of plausible code pointed slightly wrong, and discovering that costs far more than the specification would have. The specification is not bureaucracy around the work. It is the part of the work that requires me.

Two practices that earned their place:

**One acceptance test per task, written before.** Not a test suite. One test that encodes what the task is for. If the agent satisfies it and the result is still wrong, my specification was wrong, and I know that immediately instead of three tasks later.

**Reuse stated explicitly.** Agents will happily reimplement something that already exists, because from inside the task the existing component is invisible. When a specialised engine already handles fiscal document processing, the instruction says to call it, not to build one. Otherwise you end up with two implementations of the same regulated logic, drifting apart, and only one of them maintained.

## The government API is its own discipline

Integrating with Brazilian federal APIs under mutual TLS, SERPRO and the fiscal gateways among them, is a different kind of problem from integrating with a modern commercial API, and it is worth saying why, because it shapes how much autonomy an agent can usefully have.

The client authenticates with a certificate, so connection setup is stateful and failure modes are specific: an expired certificate, a chain the server will not accept, a TLS negotiation that fails for reasons the error message does not explain. These are not the HTTP errors an agent is used to reasoning about.

Availability is not a guarantee. Services go down, degrade, or answer slowly during filing deadlines, which is exactly when they are needed. Retrying harder is often the wrong response.

The contract moves. Fields appear, meanings shift with regulation, and the documentation lags the behaviour. Code written against last quarter's understanding stops being correct without failing loudly.

An agent handles the mechanical parts of this well: the client layer, the retry logic, the parsing, the error typing. What it cannot do is tell the difference between "this failed and should be retried", "this failed and a human must look at it", and "this succeeded but the answer means something different from what we assumed". That distinction depends on knowing the domain and the regulation, and it is mine.

The practical rule I landed on: the agent builds the integration, and the classification of outcomes is specified, not inferred. Every response path is assigned a meaning in advance by someone who knows what the regulation says.

## Where I let the agent run and where I do not

The useful frame turned out to be blast radius, not difficulty.

Wide autonomy where a mistake is cheap and visible: implementing a well-specified function, writing tests, building UI, refactoring inside a module, generating a migration against a reviewed schema, exploring an unfamiliar part of the codebase to answer a question.

Narrow autonomy where a mistake is expensive or silent: anything crossing a tenant boundary, anything touching credentials, anything that changes what gets written to the audit trail, anything that alters the approval line, anything that changes how a regulatory rule is interpreted in code.

That split has nothing to do with how hard the task is. Some of the narrow-autonomy work is trivial to implement. It is narrow because being wrong is not recoverable in the same afternoon.

## Verification proportional to risk

The last piece, and the one that took longest to get right.

My instinct early on was that a regulated domain justifies maximum verification everywhere. Run everything, prove everything, every time. Agents are happy to comply, and the result was thirty-minute validation cycles on changes that could not possibly have broken what was being validated.

That is not safety. It is cost that looks like safety, and it has a second-order effect that is worse: when every change triggers the same heavy ceremony, the ceremony stops carrying information. Nobody reads a green suite that is always green.

What works is proportion. Focused validation scoped to what the change can reach. Broad regression when a change is structural, crosses a boundary, or touches one of the five decisions above. Components already homologated and running in production are not re-proven on every pass; they are re-proven when something beneath them moves.

The verification budget is finite, in machine time and in human attention. Spending it uniformly guarantees it is underspent exactly where it mattered.

---

The summary I would give someone about to do this: coding agents are entirely usable for regulated work, and the thing that makes them usable is not supervising them harder. It is deciding, before they start, which questions they are not allowed to answer.
