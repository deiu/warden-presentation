

Positioning: opportunities for upward influence for a startup CTO are different from such opportunities for a Staff engineer at an established company like Datadog. As a CTO, you must be able to influence your executive peers and the engineering culture, while you are expected to be the decision maker in the engineering team. That said, my approach towards the engineering team has always been to present the vision and use solid technical and commercial arguments when making engineering decisions.

=========================================

# Slide 0 (Title)

- Presentation title: Building a Self-Orchestrating Agentic Economy
- Warden metrics
- Warden team

Warden pivoted from a blockchain project to an AI project through Warden App (their thesis was that natural language interfaces would fix the broken UX in crypto). It offered a mix of mostly crypto-related agents (trades, swaps, bridges) and generic (financial deeep research, portfolio management, Tweeter performance analysis, Gmail/Gcal connectors, etc.), each bringing in revenue on its own. It worked: 25M+ users, 500K+ DAUs in the first year.

As CTO, I lead an engineering organization of 6 core developers and ~15 full time contractors.


# Slide 1.1 (Problem and constraints)

> INTERNAL NOTE: present it as a lesson around decentralized marketplace.

## Intro (verbal, not on the slide)

The platform launched in Q2 2025 hoping to reach a very ambitious goal of 100K users. It blew our expectations when we saw 1.5M users in the first couple of months.

**The problem:** Since the main revenue stream depended on the number of agents available to the users, it quickly became obvious that scaling the business meant adding A LOT more agents. At the same time there were two important constraints to consider:
- compatibility: initially Warden only supported LangChain-compatible agents (devs had to be part of the LangChain ecosystem, or learn it)
- friction around crypto payments => in the end opted for a stable coin (USDC) payment flow (more predictable revenue stream)

My suggestion was a two-stage plan. Step 1: build an agent market place (like an app store), where devs can register and monetize their agents, and Warden becomes the distribution channel. Step 2: build an agentic economy, where agents could augment their capabilities by outsourcing data from other agents around the world.

# Slide 1.2 (Solution)

My suggestion was a two-stage plan. Step 1: build an agent market place (like an app store), where devs can register and monetize their agents, and Warden becomes the distribution channel. Step 2: build an agentic economy, where agents could augment their capabilities by outsourcing data from other agents around the world.

**How I influenced my peers:** Here are the two main arguments that I used to get buy-in for this proposal. They were based on strategy and commercial incentives, instead of actual numbers, since the vision was very novel at that time.

1. In building a marketplace Warden would be offering distribution (access to its user base) to 3rd party developers who were interesting in building and monetizing their agents. The appealing part for them is that it fixes the "cold-start" problem that most devs/startups have -- i.e., the users are already here. We were interested in quantity vs quality at that time (remember the old app stores?).

2. In parallel, we would be building towards an agentic economy where agents would be able to discover and outsource work to other (more specialized) agents. Coming from a decentralized, standards-based background, I noticed that most of the building blocks that unlock this vision were already there (although in very early stages): ERC-8004 for agent identity and discovery, A2A for inter-agent communication, and x402 for agent-to-agent payments.

# Slide 2 (Baseline + why it was hard)

Let me first present the state I started from. Warden App Beta had recently launched. However:
- Warden App was a centralized agent platform where Agents were built using LangGraph and agent deployments were done on LangSmith.
- Implicit trust in agents: Warden also controlled the agent logic and tools they used (with typical guardrails around prompt injection)
- Agent routing was done internally (no agent discovery system, all agents were known / deployed by Warden)
- There was little to no engineering culture. Impossible to plan and execute consistently on the roadmap. This is where I focused first as a CTO until August/September.

The plan was to start with the work in which we could control the outcome end-to-end: the Marketplace. In practice we need to build an agent registration & monetization platform, supporting agent identity and discovery, with transparency tools and guardrails.

Then in parallel expand towards the agentic economy vision, which would take longer and we couldn't control. In practice, here in meant adding support for A2A and x402 to Warden agents.

To be able to achieve the vision, I realized I was facing multiple challenges:
- Frontier work (no blueprint). AFAIK even today I am unaware of a similar project.
- Problematic engineering culture: did not trust the team to pull off such a vision at this scale.
- Standards were emerging, but it was very early and we had no guarantees they would get traction.
- No developer ecosystem yet, meant it was difficult to get feedback, but also that we had to build the ecosystem.
- Security implications when working with trustless agents (potentially adversarial)
- Performance-wise, it was impossible to estimate the latency and potential infrastructure issues for 3rd party agents.

# Slide 3 (Design deep dive, zoom on the agent marketplace)

## Slide 3.1 Trade-offs

- Standards over a proprietary solution. In-house would have been faster, but standards paint a better commercial story and allows us to get faster validation/buy-in from the community. At the same time I was aware that shipping against drafts means rework risk if standards move; yet waiting means missing the market window.
- Start with an internal payments solution. Faster and cheaper for internal platform payments. Add x402 later for Agent-to-Agent payments.
- Fully isolated/sandboxed agents. Basic chat interaction for now with more complex access policies coming later.
- No Agent hosting. Too risky for us to take on extra infrastructure costs early on. Agent devs had to use their own infrastructure (BYOI)

## Slide 3.2 Agentic Economy Flow

The flow (or flywheel in this case) was a self-reinforcing loop where users would interact with an agent in order to get work done. The agent would have access to a catalog (index) of all the available agents and their capabilities, and was able to pay (hire) other sub-agents to perform work that itself wasn't capable of doing. Once the work was done, the agents would be paid, devs could start making money and potentially be motivated to build more agents, this growing the agent catalog.

## Slide 3.3 Self-Orchestrating Agents

How does the flywheel work in practice?
@@
A client agent is the agent that the user first interacts with. It comes with its own runtime, an A2A protocol client to allow it to communicate with other agents, x402 client to sign payments (like a bank check), and a wallet (for spending or being payed).
@@
- The self-orchestration is a term I invented to refer to the ability for an agent to "follow its nose" and discover if it can outsource part of its work to another agent.
- The agent lookup was done against a centralized Agent Registry service. If another agent matched the required criteria, then the client agent would initiate an exchange with it over A2A.
@@
- The Server agent (the one contracted) would ask for payment through x402 and then process the task. To handle the payment it would rely on a 3rd party x402 Facilitator service.
@@
- The last layer is where the onchain work was done, either for payment settlement or to index agents and update agent reputation.

I also have a protocol flow that we can discuss later if you'd like.

## Slide 3.4 Agentic Architecture (Warden)

Zooming out, here are the different architectural components that were required in order to enable self-orchestrating agents.
@@
Remember, we only had the Warden App with its internal agents. Everything else had to be built roughly at the same time.
@@
I built the Warden Studio dev portal (where devs can register and manage their agents). The Wrapper Agent (an agent-like service that interfaced with other agents, either internal or external). It had support for A2A, Payments, PoI generation and guardrails. In terms of guardrails, together with the team I opted to fully sandbox the agent access so it would not be able to access PII data or to perform financial transactions for the user (enfored at the tool level). I started working on a permission system (similar to OAuth) that I never had time to finish.
I had help from the team in building the PoI contract.
The Wrapper connected to Internal agents (we called these community agents). But also with...
@@
External agents that were registered on other networks, as well as the cross-chain indexer. The indexer is a service that the team built in order to crawl other ERC-8004 registries and discover more agents. It came with a tool that checked spec compliance in order to validate interoperability across the different standards. Turns out that we grossly overestimated the quality of the agents available at that time, the interoperability tool only returning 1 or 2 agents that were actually compatible. I decided to deprioritize the work on this tool to focus on other parts of the architecture.

# Slide 4 (Personal Contributions and Rollout)

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

# Slide 5 (Outcomes)

- before/after metrics
- We were really happy with how the Warden App was growing, but it wasn't only good news.
@@
The plan fell short in a different number of ways:
- We noticed most people were not building production grade agents (mostly testing)
- The dev community didn't really take off. I suspect that at the time there wasn't a critical mass of devs familiar with or willing to transition into building agents.
- Unreliable 3rd party providers (LangChain, Privy, Venice.ai).
- Interop issues (worked closely with LangChain team to fix their A2A implementation)

# Slide 6 (Learnings)

01. Being in the lead also means giving others time to catch up. I often noticed there was no one else we could turn to for collaboration or validation. 
@@
02. #2 is a good follow-up for #1. In hindsight, I realize that I was too ambitious. A better approach would have been to identify players in the ecosystem who might have benefited from being part of the agentic economy early on, and use them as design partners. Focus on a few verticals instead of going after the full scope vision.
@@
03. Despite $1M in incentives for developers, we never saw much interest (it eventually grew to 100+ early 2026). I know that building dev communities takes time, but in the current context it didn't really make a difference. And with this last comment, I have reached the end of my presentation.

# Slide 7 (Thank you)

Thank you for your attention.