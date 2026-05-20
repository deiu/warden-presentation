Opportunities for upward influence for a startup CTO are different from such opportunities for a Staff engineer at an established company like Datadog. As a CTO, you must be able to influence your executive peers and the engineering culture, while you are expected to be the decision maker in the engineering team. That said, my approach towards the engineering team has always been to present the vision and use solid technical and commercial arguments when making engineering decisions.

# Slide 0 (Title)

Hello everyone, my name is Andrei and today I want to tell you about a how I built a Self-Orchestrating Agentic Economy at Warden.

First a few words about Warden. This was a company that pivoted from a blockchain project to an AI project. Their thesis was that natural language interfaces would fix the overly complex UX in crypto. To prove it, they released Warden App, which offered a mix of mostly crypto-related agents (trades, swaps, bridges) and generic agents (deeep research, portfolio management, and many others), each bringing in revenue on its own. It worked: the platform grew to 25M+ users, 800K+ DAUs in the first year.

As CTO, I lead an engineering organization of 6 core developers and ~15 full time contractors. 

But how did we go from this app to an agentic economy?

# Slide 1 (Problem and constraints)

The platform launched in May 2025 hoping to reach a very ambitious goal of 100K users. By July it blew our expectations when we saw it surpassed 1.5M users.

However, since the main revenue stream depended on the number of agents available to the users, it quickly became obvious that scaling the business meant adding more agents.
@@
So, the problem was "how do we get more agents?".
At the same time there were two important constraints to consider:
- Warden only supported LangChain-compatible agents (meaning devs had to be part of the LangChain ecosystem, or learn it)
- There is a lot of friction around crypto payments => I pushed for a stable coin (USDC) payments for a more predictable revenue stream

For the solution..

# Slide 2 (Solution)

I suggested a two-step approach. Step 1: build an agent marketplace (like an app store), where devs can register and monetize their agents, and then Warden becomes the distribution channel, while in parallel start working on Step 2: build an agentic economy.

The two main arguments that I used to get buy-in from the exec team for this proposal were based on strategy and commercial incentives (instead of actual numbers) since the vision was very novel at that time.

1. In building a marketplace Warden would be offering distribution (access to its user base) to 3rd party developers, thus fixing the "cold-start" problem that most devs/startups have -- i.e., the users are already here.

2. In parallel, we would be building towards an agentic economy where agents would be able to discover and outsource work to other (more specialized) agents. Most of the building blocks that unlock this vision were already there (although in very early stages): ERC-8004 for agent identity and discovery, A2A (Google) for inter-agent communication, and x402 (Coinbase) for agent-to-agent payments.

# Slide 3 (Baseline + why it was hard)

Let me first present the state I started from. 

Warden App had recently launched. However, it had some limitations:
- all Agents were built using LangGraph and agent deployments were done on LangSmith.
- Implicit trust in agents: Warden also controlled the agent logic and tools they used
- Agent routing was done internally (no agent discovery system, all agents were known and deployed by Warden)
- There was little to no engineering culture, making it impossible to plan and execute consistently on the roadmap. This is where I invested a lot of effort as a CTO until things improved around August/September.

The plan was to start with the work in which we could control the outcome end-to-end: the Marketplace.

Then in parallel expand towards the agentic economy vision to focus on external agents, however which would take longer and we couldn't control.

# Slide 4 (Challenges and Trade-offs)

To be able to achieve the vision, I realized I was facing multiple challenges and trade-offs:

@@

- **Multi-hop failures compound.** Agents-hire-agents is the headline but also the failure-mode multiplier: each hop adds settlement, proof generation, and verification, all of which can fail independently. The only viable solution meant bounding recursion depth and total budget per call tree at the protocol level, keeping cost paths and cascading failures contained as the call graph grows.

- **The reliability math is unforgiving.** Aggregating availability across components we don't operate compounds the unreliability and is bounded by the weakest external dependency. Five dependencies at 99% (each) result in 97% before we even start writing code.

- **External dependencies sit across a trust boundary.** Chains, facilitators, RPC providers, indexers, external agents: they each need a different approach. Cascading failures appear when conflating "things we operate," "things we depend on," and "things we cannot trust".

- **Onchain is slow truth; offchain is fast staleness.** The chain is canonical but expensive; every cache, every event-driven view, every indexer will eventually disagree with it. Reconciliation is the safety net under every read in the system.

- **You can't page the chain.** Much of "ops" in this system is the protocol itself. We can't restart external services we don't control; observability has to span systems we don't operate, and recovery often means serving cached state while someone else's outage runs its course.

- **Latency is the sum of every trust-boundary crossing.** Per-call settlements, cross-chain reads, sub-agent invocations, facilitator round-trips: each adds time we can't optimize around. The architectural decisions about where to push sync calls versus async matter a lot.


# Slide 4 (Design deep dive)

## Slide 4.2 Agentic Economy Flywheel

The flywheel was a self-reinforcing loop where users would interact with an agent in order to get work done. The agent would have access to a catalog (index) of all the available agents and their capabilities, and was able to pay (hire) other sub-agents to perform work that itself wasn't capable of doing. Once the work was done, the agents would be paid, devs could start making money and potentially be motivated to build more agents and growing the agent catalog.

## Slide 4.3 Self-Orchestrating Agents

How does the flywheel work in practice?

@@

A client agent is the agent the user first interacts with. It comes with its own runtime (model + tools), A2A support that allows it to communicate with other agents, x402 for payments (like a bank check), and a wallet (for spending or being payed).

@@

- The self-orchestration is a term I invented to refer to the ability for an agent to "follow its nose" and discover if it can outsource part of its work to another agent.
- The agent lookup was done against a centralized Agent Registry service. If another agent matched the required criteria, then the client agent would initiate an exchange with it using A2A and x402.

@@

- The Server agent (the one contracted) would ask for payment through x402 and then process the task. To handle the payment it would rely on a 3rd party x402 Facilitator service.

@@

- The last layer is where the onchain work was done, either for payment settlement or to index agents and update agent reputation.

I also have a protocol flow that we can discuss later if you'd like.

## Slide 4.4 Payment Flows

Payment flows are worth taking a quick look at. 

@@

The general trend at the time was to use x402 (from Coinbase). The payment protocol uses HTTP 402 (Payment required) to basically implement a paywall functionality. The flow is as follows:

However, as you might have already guessed, there are several issues that appear here:... Which is why I spent a bit of time coming up with an alternative.

@@

Long story short, this alternative involves subscriptions. The main difference is that payment is no longer done upfront, and a very light subscription flow is required once.

The main advantages of this approach are that ....

# Slide 5 (Rollout & Personal Contributions)

Timeline:
The timeline covers most of my involvement as CTO at Warden. 
- Warden App was launched around May, but it grew in time with more features being added (Trading terminal, Prediction market, but also more agents). While I didn't actively contribute with code during its development, my contribution was on the engineering culture.
- Otherwise, in terms of engineering contributions, the items marked in orange are the ones I owned end-to-end...

I would like though to spend a moment here talking about engineering culture.
@@
When I arrived in this company, I found an engineering team that had no direction and focus.
- There were no formal processes, which meant that it was impossible to follow a roadmap and ship on time.
- There was also a serious reliability issues, with releases seeing a lot of bugs in production.
@@
What I did was to:
- Add a bit of process without introducing too much friction:
    - PROCES: Gradually introduced process: Definition of Done, PRDs, release coordination checklists
    - PROCES: Security reviews before any public-facing feature shipped
    - PROCES: CodeRabbit in the PR workflow for early automated review
    - PROCES: QA as hard gatekeeper for every production release
- Define ownership and responsibilities
- Align product with tech and with commercial
- And last but not least, mentor the team and help them grow
@@
In just a few months the team was able to:
- consistently release on schedule
- improve the release quality: fewer production bugs, no security incidents, no rollbacks
- adapt to commercial asks from one week to another

Without these measures (and I have to credit the team as well), we would never have been able to execute on the roadmap.

# Slide 6 (Outcomes)

- before/after metrics
- We were really happy with how the Warden App was growing, but it wasn't only good news.
@@
The plan fell short in a different number of ways:
- We noticed most people were not building production grade agents (mostly testing)
- The dev community didn't really take off. I suspect that at the time there wasn't a critical mass of devs familiar with or willing to transition into building agents.
- Unreliable 3rd party providers (LangChain, Privy, Venice.ai).
- Interop issues (worked closely with LangChain team to fix their A2A implementation)

# Slide 7 (Learnings)

01. Being in the lead also means giving others time to catch up. I often noticed there was no one else we could turn to for collaboration or validation. 
@@
02. #2 is a good follow-up for #1. In hindsight, I realize that I was too ambitious. A better approach would have been to identify players in the ecosystem who might have benefited from being part of the agentic economy early on, and use them as design partners. Focus on a few verticals instead of going after the full scope vision.
@@
03. Despite $1M in incentives for developers, we never saw much interest (it eventually grew to 100+ early 2026). I know that building dev communities takes time, but in the current context it didn't really make a difference. And with this last comment, I have reached the end of my presentation.

# Slide 8 (Thank you)

Thank you for your attention.