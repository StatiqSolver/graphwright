# Decisions only the owner makes

The first agent sends these to the owner as one batch, early (greenfield Phase 0,
or retrofit stage A), and keeps working. In a retrofit, pre-fill each answer from
what the repo already does and ask only what's genuinely open. Each has a recommended default that applies until the owner answers.
Each answer goes into `AGENTS.md` or a decision record in the same change.

Phrase them to the owner as consequences, as below, not as engineering choices.

---

**1. Who is it for, and what one thing must work end to end?**
This becomes the charter's core journey, the first milestone, and the thing every
"is it done?" question gets measured against.
*Default:* the agent drafts it from the owner's description and asks for a yes.

**2. Will the product itself be open source, closed, or undecided?**
This decides which open-source projects we're allowed to build on. Some licenses
require that anything built with them is also shared openly.
*Default:* undecided, so we only take permissively licensed projects (MIT, Apache,
BSD) into anything we hand to users. Anything else runs as a separate, swappable
piece and gets a written exception.

**3. Which parts need the hardest review?**
Changes there get reviewed until reviews stop finding real problems, and you make
the final call. Everything else gets a fixed number of review rounds and ships.
*Default:* anything an AI agent inside the product can do to a user's files or
machine; sign-in and accounts; money; loss of user data; and any number the
product presents as a computed engineering result.

**4. Who can merge and who can deploy?**
*Default:* agents open and prepare changes; only you merge to `main`, and only you
start a production release. Merging never deploys by itself.

**5. Who reviews, and what happens when a reviewer runs out?**
Two AI vendors reviewing each change catches mistakes that one alone misses.
*Default:* Claude and Codex, both on your subscriptions, never paid API keys. If
one runs out of quota, the other reviews and the change is labelled as reviewed by
one vendor only. If neither is available, changes wait.

**6. May agents fix usability problems without asking?**
*Default:* yes, for discoverability, dead ends and hard-to-use screens, as long as
the fix is seen working. Not for scope, pricing or anything in decision 4.

**7. May a senior agent close finished work without you?**
*Default:* yes, when every acceptance criterion is met and the evidence is posted
(including a screenshot for anything a user sees). It can't close work by
shrinking its scope, and it can't defer things.

**8. What may we spend?**
New paid services, and any usage-billed API, need your yes first.
*Default:* subscriptions and free tiers only. The agent quotes a monthly cost
before asking for anything else.

**9. Where do the apps run?**
Whether the local app must work offline, whether users sign in, and where the web
app is hosted all change what gets built first.
*Default:* the agent proposes an answer in decision record 0001 with the cost of
each option, and asks for a yes.
