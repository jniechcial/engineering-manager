---
layout: post
title:  "Software documentation in growing companies"
author: "Kuba"
comments: true
description: Finding the right balance between moving fast and keeping high-quality documentation is very hard for growing engineering organisations. If you feel that these problems are present in your company, experiment with Architecture Design Records.
image: "/_imgs/documentation.jpg"
related: [
  ["2019-01-14/ownership-explained-for-engineers-and-managers", "Ownership explained for Engineers and Managers", "Ability to take ownership is critical for your career and is a major step between junior and senior role. See how to look for high-impact ownership for engineers."],
  ["2018-01-17/where-can-you-find-feedback-for-your-team", "Where can you find feedback for your team?", "Ambitious people love feedback. They are hungry for it and are aware that feedback is helping them grow faster. How can you find feedback that will really help your directs be better?"],
  ["2017-11-05/growth-of-the-company-is-your-growth", "Growth of the company is your growth", "It may be common among engineers to hear that the company is growing too fast. What does the growth of the company actually mean for an engineer in startup?"]
]
---
Maintaining functional and correct architecture documentation for your product is basically an unsolved problem. There are companies where this is critical and rooted in compliance regulations - like aviation - and they do it right, but at a high cost. Architecture documentation is quickly outdated, and in fast-growing companies, the product evolves so fast that trying to keep your documentation up to date continuously slows you down. The moment it gets outdated and entropy consumes it, the moment it loses credibility and trust.

Finding the right balance between moving fast and keeping high-quality documentation is very hard for scaling companies. In practically all cases, it’s always better to make trade-offs on documentation and move faster building product.

Another common problem of scaling engineering organisations is how to scale your best practices, cultivate culture and maintain knowledge. The most critical moments that shape engineering culture at the organisation are when you have to make trade-offs and rationalise decisions. Even well done, architecture documentation misses the rationale, representing only the result. And usually, it’s easy to see the decision being made. For example, we decided to model this system in that way, which is generally understandable from the codebase. But what’s missing is why did we make this decision?

If you feel that these problems are present in your organisation, experiment with Architecture Design Records.

## What are Architecture Design Records?
Architecture Design Records are documents that describe:

* the engineering problem that the team is facing at a time,
* requirements that we need to meet to solve this problem,
* a variety of solutions that we brainstormed before choosing a solution with their trade-offs
* and the detailed description of the solution we chose

The exact template that the organisation uses will vary depending on the evolution of the process as you adopt it, your organisation’s current need, and your specific product.

At Intercom, we call them Tech Design documents and use Google Docs to write and store them. I know about other companies that write them in markdown and store in Github, in respective locations to code files that they touch.

I am not opinionated on the shape and format of this document. I think organisations should experiment and evolve their processes continuously, so it’s more important to start and iterate than come up with a perfect template.

However, I firmly believe that the past rationale and trade-offs analysis is one of the most valuable lessons that your organisation can take. Environment changes, constraints come and go, but the way someone distilled a problem, compared the options and made a decision is a prize on its own. That’s your engineering culture manifesting itself in real life. If your values and principles show in these documents, your culture is strong. But if they don’t, they are aspirational - and you need to work harder as a leader to ingrain them.

Who is a stakeholder of such documents? The team that wrote the document is a primary stakeholder. It increases the team’s engineering excellence and will be an artefact that the team itself will mostly use in the future. Additionally, in the case, you are a software consulting business, ADR stakeholder is also your customer. You can keep them up to date and aligned with your work.

Who is an approver? By default, especially in smaller or growing companies, it’s the team itself. More mature companies or those with higher costs of making mistakes, tend to have Architecture Review Boards that approve these docs.

## How you create them is as important as the documents itself.
Writing Application Design Records is a process in itself. And the way your organisation does it is equally important as the output document!

First of all, creating ADRs can easily enforce collaboration and mentoring. A well-rounded collaboration group can have a technical leader overseeing the solution, more junior members learning through practice and someone detached from the team to bring unbiased opinion.

This process also scales your engineering excellence. By writing ADRs for significant problems, you allow your most senior leaders to scale their influence. If your Principal Engineer collaborates with a team of 15-20 engineers, it’s challenging for her to track all the critical decisions organically. ADRs ensure that teams will solicit other senior leaders’ inputs in a scalable and organised way.

You also create opportunities for peer review of the work even before the first line of code is written. As the saying goes, an hour of planning can save a week of development.

And last but not least, you share knowledge between teams that don’t work together organically. Especially in the remote environment!

## How to trial and adopt the process?
Writing documents can be a gruelling and disengaging task for engineers if they don’t understand the reasoning, can’t immediately see the value or leaders leading by example.

Try the following process to introduce ADRs as an experiment.

**1. Lead by your most senior engineers.**

The first ADS should be created under the leadership of one of your most tenured and experienced engineers. Lead by example and show the value of knowledge sharing and engineering excellence to your organisation from your best folks.

**2. Adopt only for a few significant design decisions.**

Don’t say that it’s obligatory to write ADRs for almost everything from now on. You might get there if your organisation becomes hard fans of the process, but don’t start there. Writing one or two every quarter to solidify the most important developments is already a great start. On the other hand, at Intercom, each team plan in 6-week cycles, and we would usually expect 5-6 tech design documents per team per cycle.

**3. Share widely and continuously look for feedback.**

Share your ADRs and ask for feedback as often as possible. We created a separate channel on Slack to share all our ADRs. It’s a big stream of artefacts, but every engineer can pick the ones they are most interested in. You work in customer-facing team day-to-day but would like to grow your skills and learn a bit about infrastructure? Pick and read ADRs from our infrastructure teams!

Did you try some form of Architecture Design Records in your organisation? Do you have lessons or observations you would like to share?
