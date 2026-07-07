---
layout: post
title:  "Reflections from my time at HubSpot"
author: "Kuba"
comments: true
description: After 2.5 years leading engineering in HubSpot's Flywheel Product Line, I reflect on turning around an org of 20+ teams and the lessons I learned building internal AI products for GTM.
image: "/_imgs/lessons.jpg"
related: [
  ["2024-01-30/lessons-from-intercom", "Lessons from Intercom on being a product engineering leader", "A reflection and summary of lessons from 6 years at Intercom, across 3 roles and 3 product groups, on leadership, product and building high-performing teams."],
  ["2023-07-27/turning-around-an-org", "Turning around a group of teams", "When you realise that the org you are leading became dysfunctional, it can feel hopeless. Here is a framework I used to quickly turn around a group of teams that was in trouble."],
  ["2026-01-26/new-talent-bar-for-software-engineers", "The new talent bar for software engineers", "Writing software is becoming cheaper and faster at an incredible rate. See how AI is reshaping the roles, expectations and talent bar for engineers."]
]
---
After 2.5 years, I’m leaving HubSpot. I am super proud of my time here. It was exceptionally rewarding, fast-paced, and dense with learning. Big change is always a great opportunity for reflection, and this post is an aggregation of high-level learnings from my time there. Most of them deserve a deeper dive that I’ll write up over next months, and this post is just a start.

### What did I do at HubSpot?

I led engineering in the Flywheel Product Line, and our mission was to reinvent our own GTM motions with AI. The opportunity is obviously huge with over 4000 people in GTM roles, and SMB-centric, 300k-strong customer base.

This mission only solidified around mid-to-late 2023, after the ChatGPT moment, and the opportunity and shape of the org shifted from a more traditional business-systems team to a product-led innovation team within HubSpot. It was a truly creative and innovative time where we tried tens of ideas, rebuilt products many times as models got stronger, and learned a ton about both GTM and AI more broadly.

We were early with our ideas. We prototyped and productionized internally plenty of things you now see live in HubSpot, Salesforce, Gong or many other GTM startups. Things like call summaries with structured data extraction against sales methodology, smart deal progression, deal coaching and best-next-actions, demo environment generation, customer agents, and agentic enrichment systems. Many, many more.

We worked hand-in-hand with our RevOps folks, business strategists, and business analysts. They bring exceptionally deep domain expertise, and when paired with a strong, high-agency product team, you can deploy AI effectively.

But it wasn’t an easy ride from the start. When I started in 2024, the org was in need of a strong turnaround. The mission was too big for the shape of the talent, processes, and principles the team had at the time. I had to turn that org around, elevate our best folks, and help us push through the high-density learning ahead of us.

### Turning around the org of 20+ teams

The first challenge was the sheer size of the org. We were over 110 engineers, over 20 teams, and 10+ Director+ stakeholders on the GTM side.

The first step is correctly identifying problems, based on a triangulated view from teams on the ground. I believe that in a turnaround scenario, when you join, the classic listening tour only works at a small scale of a few teams. With this much diversity in engineers and levels, and a lack of trust on both sides, you need to get close to the work and be willing to step down a few levels. **You just shouldn’t blindly trust anything, stay sceptical and validate and understand very closely.**

I spent over 3 months working most of my day as an engineer, directly with 4 teams (and some adjacent teams from time to time), in different parts of the org. The embed gave me a rare opportunity to build trust and deeply triangulate the problems and opportunities. I could hear them, probe them with other folks in the team or stakeholders, and validate them myself as someone who is part of the day-to-day team. If I ever join an engineering org for turnaround purposes, I would definitely repeat that.

After that, my direct team of senior managers and my product partners built our first pass at a new mission, vision, and strategy. This turnaround was pretty big (from order-taker to innovation hub), and landing a change like this is all about messaging. This takes quarters to land, and one of my team members gave me a rule of thumb I still use: **if you feel like you’re communicating just right and it’s not uncomfortable for you, you’re not communicating enough.**

During my embed, I spent a lot of time identifying high-performers, validating that with my management team, and later elevating them and strengthening their ownership and agency. You need folks who will champion the new world at every level, and I think it’s critical to connect a few levels down in your org to make sure the message lands exactly right, without anything lost in translation. **Folks look up to other successful folks, and peer pressure, peer coaching and peer role modeling are one of the highest leverage options.**

**Last, speed needs to be uncomfortable, even for you and your most trusted people.** I learned that after Eoghan McCabe came back to Intercom in 2022 as CEO and really shook us deeply in how fast you can move. It stayed with me as a beacon of how fast you need to swing the pendulum to see the change. In HubSpot, everyone told me I was moving too fast. Many tenured people told me that rolling out change this way wasn’t the HubSpot way. This might have been true in the past, but in turnaround you need to swing hard.

### Reinventing with AI means flip-flopping between a blank check and scrutiny of impact

If you’re following X right now (July 2026), you’ve definitely seen the growing concern about token cost, ROI, and budgeting. None of this existed just four months ago. We’ve seen this cycle repeat multiple times internally since I started (not about costs though, but about impact). Generally, I’ve seen broad conviction on AI across execs and senior leadership, but at some point expectations shift and you move abruptly from “that’s awesome, can you also do X” to “okay, but where’s the impact.”

**There is so much cool stuff you can build with AI. It’s very easy to spend weeks to months building exactly the wrong thing.** We spent a lot of time breaking down the customer journey — what are reps’ actual jobs-to-be-done, what are they doing day to day, and which of those tasks are both time-consuming and feasible for AI, at its current state, to deliver with great quality.

From there, we found ways to measure performance on these jobs — leading metrics, often output metrics. Where possible, we showed the connection to outcome metrics, and in a few places even built causal models that proved causality.

One interesting observation: **incentives and the bar need to move up as you build conviction that your product is making people more efficient**. That’s what actually forces a behavior shift and lets you capitalize on the time you’ve earned back. Without that, your beta testers will usually show outsized impact, and as you roll out from a population of 150 people to 1,500, you’ll see lower impact numbers because some percentage of people just take it easier.

### Introducing AI to your GTM team has to compound over time

Throwing in tools without a clear vision for how they compound isn’t helpful. It makes your data murky and makes it hard to compound value. We rolled out three huge tools at a similar time, in an uncoordinated way \- Lovable, Glean, and Claude. Within months we lost trust in our data, saw big shifts in rep behavior, and watched people drift away from proven tools toward more exploratory builder tools. That was a mistake I wish we avoided.

**Instead, choose a small set of tools that you’re going to manage, enable, and iterate on.** They need to be highly extensible, continuously invested in, and built to enable experimentation. Claude Code/Cowork is a good example, and I can totally see why they are winning the market share that fast. They are just an awesome compounding platform.

On the other hand, this also means that there is often a potential compounding effect on your impact metrics. Many of our bets quickly showed good indicative metrics, but still took almost a year to play out with visible, double-digit impact on the entire rep population.

### Lessons from building products translate to building internal AI for your GTM

My product partner and I came from Twilio and Intercom respectively, where we spent our time building core products. We brought some of that approach to internal GTM.

We anchored on user research and deep understanding of problems and user jobs. We always started with a small alpha group that had a mandate to be early adopters and partner with us. We sometimes had to give them a small quota relief to ensure they had time.

Later, once we had feedback that the solution was proving valuable and covered most edge cases, we’d expand to a larger beta group (no mandate, no relief). This gave us high-confidence numbers and let us compare beta-group performance to non-beta-group performance to prove impact.

After having that data, we would roll out, enable, and sometimes even force usage after alignment with rep leadership.

**The most important lesson here: if you move too fast through these stages, you lose the opportunity to prove impact, and you’ll most likely ship an underwhelming product to a broad audience, losing their interest and burning the bridge early.** While distribution is easier for internal products than for commercial ones, you still can’t cheat real value or user perception.

### AI adoption is led by leadership and managers

I think every company has already realized that to push AI adoption, you need to push from the top, keep people accountable, and role-model the behavior.

We saw great acceleration when our sales leadership set a quota for AI adoption across their teams. This forces the org to be more open-minded and active in product development \- providing feedback and giving you more quantitative insight.

**To make that work, though, your managers need to see the value in AI first. They need to use it. That’s what gets them talking about it, role-modeling it, and getting their team to follow.** One of our products have seen very fast adoption among sales reps, mostly because we optimized it and enabled for first for managers. They loved it, and used it on their pipeline reviews with their teams and created great word of mouth marketing.

### RevOps-as-code

We haven’t gotten there yet fully, but I believe Operations will go through a shift similar to DevOps and infra-as-code.

Everything will become defined in code, instead of visual workflows and a patchwork of tools stitched together. Code is testable, observable, reversible, and Claude can navigate it and reason through second- and third-order consequences in ways that visual workflows, lists, and recipes just can’t.

We’ve seen this play out many times: workflows stitched together through tools become safer to change, more reliable, and better tested and understood once they move to code. **And non-engineers can easily suggest changes by working with Claude Code, opening PRs, and — after review by engineers — getting them safely into production.**

I don’t think there exists a good platform yet to actually do that at scale, and with x-company transferable knowledge (what terraform was). If this prediction comes true, I would bet we will see inspiration in tools like Apache Airflow, and I wouldn’t be surprised if Workato goes this direction, or AWS comes up with a more generic solution.

### The ML, Data Science, and Engineering roles are getting closer together

Especially if you are building AI products, it will be hard to say when traditional software engineering ends and machine learning starts, and vice-versa. You just can’t build successful products without evaluations, creating benchmark datasets, experimentation. Not even going further into fine tuning or your own RL.

You still definitely need experts to set direction and make a handful of significant, multi-quarter decisions. **But everything in between seems to boil down to great engineering talent that has good fundamentals, can stretch across domains and has access to frontier model with sizable budget.**

There’s nuance in speed, though. A product engineer building a classification engine with good recall and precision is totally feasible, but it’ll probably take a few weeks and some oversight from an expert. An expert would probably do it in days. But as the muscle gets stronger, it gets faster.

I think the solution will more and more frequently be one team \- MLEs and data scientists working hand in hand with product engineers, with frequent tours of duty with software engineers building up their MLE chops by leading the work under supervision.

### Everyone needs platform-wide AI primitives that accelerate the entire company

Our DevEx team at HubSpot did an exceptional job building critical AI primitives that power core HubSpot features and are highly extensible. We were lucky that HubSpot’s business also relies on those so they received a big amount of investment that we didn’t need to pay for, but there are great open-source options out there too.

**First, our principle was to expose everything as MCP endpoints.** All internal systems should be MCP-first. That lets us integrate with Claude, HubSpot Breeze, or whatever other tool pops up in the coming months.

**Second, you need an agent harness that’s highly extensible.** We built our custom, but Agno was also used in a few places at HubSpot. I didn’t play around with Claude Managed Agents, but I would be vary of locking yourself down to just Anthropic models.

You also need a knowledge base that’s accessible programmatically via RAG, text search, and with metadata filters. We had to build something specifically for our use-case, but there are plenty of good products out there.

**I strongly believe that you also need storage for the AI insights and signals your data scientists and AI engineers generate.** Discoverability is key and we quickly realised that there is huge amount of insight that we generate, don’t scale, and duplicate. That’s why we built our own platform, that mapped entities 1:1 with our HubSpot instance. It acted as an extension of a CRM, but for unstructured data and agent-first discoverability.

And last but not least, you need an evals system. We had some custom built solutions but really struggled to take it off at scale of many teams. Later on, we migrated to Braintrust and its ease of use let us dramatically ramp up eval coverage and get both engineers and PMs close to agent traces.
</content>
