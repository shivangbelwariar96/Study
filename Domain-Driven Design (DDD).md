This is an interview question with no clean numeric answer, and the speaker spends most of the video showing you the *method* for arriving at one, plus the warning sign you've gotten it wrong. Three big pieces: (1) the four properties microservices should have, (2) the **Domain-Driven Design (DDD)** approach with event storming and bounded contexts, (3) the **distributed monolith** anti-pattern. Let me walk through every piece with mobile-friendly visuals.

## Part 1 — Why this question is tricky

The speaker opens with a candor that matters: **there is no right number.** Anyone who says "always 3 microservices" or "5 microservices per domain" is wrong. The interviewer asking *"how many?"* is testing whether you fall into that trap.

The honest answer is: *"There's no fixed number. Let me walk you through how I'd determine it, and we'll see what comes out."* That reframes the question from "give me a number" to "demonstrate your thinking" — which is what they actually want.

## Part 2 — What we're trying to achieve

Before you can decide how to split a monolith, you need to know what success looks like. The speaker lists four properties every microservice split should produce. These are your acceptance criteria.
<img width="1472" height="924" alt="image" src="https://github.com/user-attachments/assets/ae0712d1-595d-4d82-984f-31ee4386dd32" />


Note property 3: **low**, not zero. Microservices must talk to each other — that's unavoidable. The goal is to make every call meaningful, not to chat about every operation. If your services synchronously call each other for every internal step, you've created a distributed monolith with all the downsides of microservices and none of the upsides.

These four properties give you a way to **validate** any proposed split: take your candidate microservices, ask if each property holds, and if any of them fail, the split is wrong.

## Part 3 — Multiple approaches exist; we'll focus on DDD

The speaker briefly mentions other ways teams partition monoliths:

- **Database-driven** — one service per database, or split by read vs write workloads (CQRS pattern).
- **Configuration-driven** — different services for different config profiles.
- **Technology-driven** — frontend = one service, backend Java = another, MySQL access layer = a third.

These all work in specific contexts, but they tend to optimize for the wrong axis. They split based on *how the code is implemented* rather than *what the business does*. The result is often technically clean services that still tightly couple around business logic.

**Domain-Driven Design (DDD)** flips this. It says: split based on the **business domain**, not the technology. Two pieces of code that change for the same business reason belong in the same service; two pieces that change for different reasons belong in different services. This is essentially the **single responsibility principle** applied at the service level.

## Part 4 — DDD's four-step process


The DDD approach to identifying microservices has four steps:

<img width="1472" height="960" alt="image" src="https://github.com/user-attachments/assets/08beac14-4bb0-4a1e-9bec-76ca13dc76e1" />


Steps 1 and 4 are short. The interesting work happens in steps 2 and 3. Let me trace the chat-application example the speaker uses.

## Part 5 — Step 1: Understand the domain

Pick a clear problem statement. The speaker uses **a chat application** as the example. It could equally be e-commerce, banking, ride-sharing — anything with enough structure to split.

The key here is that **you cannot do this step alone as an engineer**. You need to sit with domain experts, real users, product managers — the people who understand the business. Otherwise you'll model the system based on assumptions, miss subtle requirements, and end up splitting on the wrong lines.

For our example: *"a chat application where users send and receive messages, with notifications when messages arrive."* Simple enough to work through.

## Part 6 — Step 2: Event storming


Now you enumerate every **event** that can happen in the domain. An event is a thing that occurs — past tense, like "user registered" or "message sent". Everyone in the room contributes independently first to avoid groupthink. For the chat app, initial list:

- User registered
- User logged in
- Message sent
- Message delivered
- Message deleted

Five events. But we're probably missing some. The second sub-step is to **sequence the events** in their natural order, which exposes gaps.

<img width="1472" height="1048" alt="image" src="https://github.com/user-attachments/assets/e4ae8d51-f7ad-47e4-bd5f-cb0f643e874e" />


Sequencing reveals events the initial brainstorm missed:

- **User logged out** — happens after login, in parallel with sending messages.
- **Message received** — the recipient side of "message sent" (from the sender's perspective). The original list only captured the sender's view.
- **User notified** — when a message arrives but the recipient hasn't opened the app, they need a notification.

The sequencing also shows that login splits into two parallel paths — a user might log out, or might send a message. Both are valid post-login flows. Capturing that branching matters because it shapes which events are *concurrent* (and therefore independent) versus *sequential* (and therefore dependent).

By the end of step 2, you have a complete map of every event that happens in your domain, in sequence, with no gaps.

## Part 7 — Step 3: Bounded contexts (the heart of DDD)

This is where DDD gets genuinely subtle. The speaker spends real time on it because most engineers misunderstand it.

A **bounded context** is a boundary inside which a particular concept has a specific, consistent meaning. The same word — same name on a piece of data — can mean different things in different contexts, and **that's the signal that you need separate microservices**.

### The sandwich example

The speaker uses a brilliantly simple analogy. Consider a sandwich:

- **Context A — restaurant**: the sandwich is a dish. People will pay for it. They want to eat it. It has a price, a description, ingredients, perhaps allergen information. It's an asset with positive value.
- **Context B — garbage bin**: the same physical sandwich. Nobody will pay for it. Nobody will eat it even if it's free. It has zero or negative value.

<img width="1472" height="726" alt="image" src="https://github.com/user-attachments/assets/b8ffc269-49a3-45f7-9ec7-892dd452a6b1" />


It's the same object — same bread, same fillings — but the two contexts give it completely different properties, meanings, and behaviors. The "sandwich" in the restaurant code is a different model from the "sandwich" in the garbage-disposal code. Trying to share one data model between them would be a mistake; you'd end up with a model that's awkward in both places.### Applying it to the chat app

Now back to our events. Group them by **which object they operate on, and what that object means in that context**.

**User registered / User logged in / User logged out** — these events all operate on the *User* object. But what does *User* mean here? It means the full authenticated identity: credentials, permissions, profile, authentication state. Call this the **User Management** context.

**Message sent / Message delivered / Message received / Message deleted** — these events operate on the *Message* object. What does *Message* mean here? Sender ID, recipient ID, content, status (pending/delivered/received/deleted). Call this the **Message** context.

**User notified** — this event operates on... well, "user." But wait. Stop and ask: is the *User* here the same as the *User* in the User Management context?

**No.** The "user" in the notification event is just a user ID and a notification preference — maybe an email address or a device token. It doesn't need to know about login credentials, permissions, profile data. It's a much thinner concept. Same word, different meaning.

If you merged "user notified" into the User Management bounded context, you'd be conflating two different concepts because they share a name. The User Management service would suddenly need to know about notification logic, or the Notification logic would have to pull in User Management's entire model. Coupling that should not exist.

So **user notified** goes into its own context: **Notification**. The "user" inside the Notification context is a different model from the "user" inside User Management. They might share an ID, but everything else is separate.

<img width="1472" height="1080" alt="image" src="https://github.com/user-attachments/assets/6d69ed65-16a4-42ec-8074-53d7da01dc41" />


### The rule of thumb for grouping events

The speaker states it clearly: if multiple events operate on objects whose **meaning in context is the same**, they belong in the same bounded context. If two events look like they're working on "the user" but the *meaning* of "user" differs, they belong in separate contexts.

And the people best placed to judge whether two meanings are "the same" are the **domain experts** — not the engineers. The engineers translate the domain expert's judgments into code. This is why DDD insists on collaboration with domain people throughout.

## Part 8 — Step 4: One microservice per bounded context

This step is now trivial because the hard work happened in step 3.

- User Management bounded context → **User Management microservice**
- Message bounded context → **Message microservice**
- Notification bounded context → **Notification microservice**

Three microservices fall out of the analysis. Not a number you picked at the start — a number that emerged from analyzing the domain.

<img width="1472" height="1080" alt="image" src="https://github.com/user-attachments/assets/7027bf26-fa0a-4b56-951f-2b94580eec4b" />


### A nuance on duplication

The speaker calls this out specifically: yes, `user_id` will appear in the Message service and in the Notification service. That's **acceptable duplication**. The Message service doesn't need the full User object — it just needs to know "this message was sent by user 42 to user 99". The Notification service similarly only needs IDs to know who to notify. Minor duplicate references between services don't violate DDD; they're the natural shape of cross-context interaction.

What you avoid is *replicating the full data model*. The Message service should not store a copy of every user's profile, credentials, and permissions. That would be tight coupling — every change to the User model would force a change in the Message model.

### Then validate against the four properties

Once you have the candidate split, walk back through the four properties from Part 2:

- Can each service be coded and deployed independently? Yes.
- Are they loosely coupled? Yes — interactions are via stable IDs, not shared schemas.
- Is communication overhead low? Yes — they interact a few times per user action, not constantly.
- Can they scale independently? Yes — message volume can spike without affecting login or notification capacity.

All four hold. The split is defensible.

## Part 9 — The anti-pattern: distributed monolith

This is the warning the speaker closes with, and it's the failure mode you must avoid. A **distributed monolith** is a system that *looks* like microservices on the architecture diagram but behaves like a monolith — except worse, because all the monolith's coupling is now happening over the network.

Signs you've built one:

- Deploying service A requires deploying service B and C at the same time (no independent deploy).
- Service A makes 20 synchronous calls to service B for one user request (no low communication overhead).
- Scaling service A forces scaling service B because they're always in lockstep (no independent scaling).
- A bug in service B can crash service A directly (no isolation).

You get all the costs of microservices — network calls, distributed transactions, observability complexity, deployment infrastructure — and none of the benefits. It's strictly worse than the monolith you started with.

### Amazon Prime Video's famous example

The speaker brings up a real case. In 2023, Amazon Prime Video's engineering team published a widely-shared article describing how they **merged two of their microservices back into a monolith** and saw a **90% cost reduction**. The two services in question handled audio and video processing.

The internet's first reaction was *"see, microservices are dead!"* The speaker corrects this: **Amazon didn't abandon microservices.** They had hundreds of microservices across the company, and they continue to. What they realized was that *those specific two services* — audio and video processing — were a textbook distributed monolith. The two services constantly called each other, were always scaled together, and couldn't be deployed independently. Merging them into one service eliminated the network overhead between them and the duplicated infrastructure.

**This is what good engineering looks like.** They identified the distributed monolith, and they fixed it. The lesson isn't "don't use microservices" — it's "don't split unless the split passes the four-properties test."

<img width="1472" height="1080" alt="image" src="https://github.com/user-attachments/assets/b3cbb81f-9ca8-4b0e-a64d-884a46a7589f" />


## Part 10 — How to answer the interview question

Putting it all together, here's the structure of a strong answer to *"how do you divide a monolith into microservices, and how many?"*

1. **Reject the premise** of the number question. *"There's no fixed number — that depends entirely on the domain."* This signals you understand the field.
2. **State the success criteria.** *"Whatever split we choose, it has to give us loose coupling, independent deploys, low communication overhead, and independent scaling. Those are the four tests."*
3. **Walk through DDD.** *"I'd use Domain-Driven Design. Four steps: understand the domain with experts, run event storming to enumerate events, group related events into bounded contexts, derive one microservice per context."*
4. **Use a concrete example.** Walk through the chat application (or whatever the interviewer's scenario is) showing the events, the sequencing, and the bounded contexts you'd identify.
5. **Mention validation.** *"Then I'd validate each candidate microservice against those four properties. If any fail, I'd merge or re-split."*
6. **Mention the anti-pattern.** *"The big risk is creating a distributed monolith — splitting things that shouldn't be split. Amazon Prime Video's audio/video case is a famous recent example of this being caught and corrected."*

That structure shows you've thought about the *process*, not just memorized a formula.

## Part 11 — Other approaches worth knowing

The speaker briefly mentions, but doesn't fully explore, other splitting heuristics. Useful to have these in your back pocket:

- **CQRS (Command Query Responsibility Segregation)** — split read services from write services. Useful when read load vastly exceeds write load and you want to scale them independently with different infrastructure (e.g. write to PostgreSQL, read from a search index).
- **Strangler Fig pattern** — when migrating from a monolith incrementally, you route specific endpoints to new microservices one by one while the monolith continues serving everything else. The monolith gradually "strangles" as more and more endpoints leave it.
- **Conway's Law alignment** — split services to match your team structure. If two teams are working on the same service, you'll see coordination overhead. If one team owns two services that change together, you'll see deployment coupling. The "two-pizza team" rule says one team should own one service.

These complement DDD rather than replace it. DDD gives you the *initial* split; CQRS, Strangler Fig, and team alignment guide how you evolve over time.

## The whole thing in one paragraph

There is no universal answer to "how many microservices?" — the right number emerges from analyzing the specific business domain, not from a heuristic. Whatever split you arrive at must produce four properties: **loose coupling** (changes don't ripple), **independent code/test/deploy** (separate pipelines), **low communication overhead** (services don't chat constantly), and **independent scaling** (one service can grow without forcing growth of others). The standard analytical method for arriving at a defensible split is **Domain-Driven Design**, which proceeds in four steps: understand the domain with the people who actually run it, **event storm** to enumerate every event that can happen in the system and sequence them to surface gaps, identify **bounded contexts** — boundaries within which a concept (like "user") has a specific consistent meaning, recognizing that the same word in two different contexts (User Management's full identity vs Notification's bare ID) signals two different microservices — and derive one microservice per bounded context, then validate against the four properties. The failure mode to avoid is the **distributed monolith**: a system that looks like microservices on paper but couples them so tightly (lockstep deployments, constant chatter, coupled scaling) that it has all the costs of distributed systems and none of the benefits — Amazon Prime Video's 2023 audio/video re-monolithification was exactly this pattern caught and corrected, yielding 90% cost savings while leaving the company's hundreds of other microservices untouched. The right number of microservices is *the number that falls out of the domain analysis and survives the four-property test*, not a number you pick at the start.

If you want, two natural follow-ups the speaker mentions briefly but doesn't go into:

**DDD tactical design** — the *strategic* design we covered (event storming, bounded contexts) gets you to "which services exist". DDD's *tactical* design covers what happens *inside* a service: aggregates (consistency boundaries), entities vs value objects, repositories, domain events, anti-corruption layers. This is the next level of DDD and worth knowing for senior-level interviews.

**Database-per-service and the consistency problem** — once you split a monolith into microservices, the single-database assumption breaks. Should each microservice own its own database? (Yes, per DDD.) Then how do you maintain consistency across services that need related data? The answer involves **sagas**, **outbox patterns**, and **eventual consistency** — a whole branch of distributed systems that becomes unavoidable once you commit to a microservices split.
