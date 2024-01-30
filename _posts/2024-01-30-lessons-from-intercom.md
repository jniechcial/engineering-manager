---
layout: post
title:  "Lessons from Intercom on being a product engineering leader"
author: "Kuba"
comments: true
description: 
image: "/_imgs/lessons.jpg"
related: [
  ["2020-07-07/lessons-being-group-engineering-manager", "7 lessons on being Group Engineering Manager", "Check out what changes when you become manager of managers and what skills and responsibilities are getting most important."],
  [ "2020-02-01/top-books-for-engineering-managers", "Top 10 books for every software engineering manager", "In management, we look for common patterns to use frameworks, processes and principles from our experience. Reading books is one of the best ways to broaden horizons as engineering manager outside of your day to day work."],
  ["2018-12-3/how-to-take-over-team", "How to take over a team?", "Taking over a team is a challenge - the cogs are spinning and you don't have context about anything and usually there is no time to slow down."]
]
---
I am finishing my time at Intercom and this post is an attempt of reflection and summary of lessons from my time there. I had a privilege to work there for 6 years, across 3 roles, in 3 product groups, with tens of unique, talented and engaged folks.

There is way more that came to my mind that I decided to not put online at this stage. After 6 years with intercom, I am not sure how much I take for granted at this stage vs. what is a genuine good lesson to share. I am curious to hear how these things resonate with you and especially if you have counter lessons!

## Leadership

### 1. Always [start from the problem](https://www.intercom.com/blog/intercom-product-principles-start-with-the-problem/) and ensure alignment with your stakeholders that this is the right framing, scope and priority of the problem.
Hands down, starting from the problem is the most powerful, life-changing lesson from Intercom. It applies to everything and provides an almost magical amount of focus in everything you do. What is the problem really? How do you know it? How will you know it’s solved? Does this sound worth spending time on? Applies to product, processes, org shape, home decoration, car repair, anything. It provides clarity like nothing else. 

### 2. Mission, vision, ownership and strategy are the most impactful tools to achieve high performing orgs.
I [wrote more about it here](https://engineering-manager.com/2023-07-27/turning-around-an-org) in a slightly different context, but with a similar conclusion - work with orgs on mission, vision, ensure they have and feel real ownership to create their own strategy and then keep them accountable for progress. Autonomy and sense of identity drive exceptional engagement and motivation which is critical whenever times get hard (and they will!). It trumps tactical goals setting or even worse, zooming in as a leader to micromanage, despite these tools providing faster but limited return.

### 3. When choosing long-term strategy - conviction, resiliency and communication are critical.
We created and executed a 2-year long product and technical strategy that shipped a new outbound product for Intercom and migrated all existing customers into it from legacy systems. We doubted ourselves plenty of times. It’s critical to maintain conviction, resiliency and ways of ensuring you are on the right track, according to schedule. Underpromise, overdeliver, keep shipping, stay focused. At the same time, you need to keep your stakeholders and supporters informed and close enough that they don’t get misaligned on timelines, progress or complexity of the work you are going through.

### 4. Quick and dirty process accelerators are great to test locally and scale globally when proven to work.
When I was EM in one of my teams, we had over 200 open issues. I couldn’t wrap my head around that and either understand how our state changes or what’s most important to work on. I came up with a script that scored each issue based on how many customers were affected . This quick and dirty prioritisation system got huge traction at Intercom and became our default and org-wide tool with its own Tableau dashboards. Similarly, when I was tasked to create an escalation process for our sales team, we ran a local experimental version with one product group and simple Coda doc before making it a global default with its own tooling. Test processes locally and [make sure they solve a problem](https://www.intercom.com/blog/engineering-processes-need-solve-problems/).

### 5. You can achieve spectacular outcomes when focusing on one thing, but with real trade-offs.
The last year at Intercom (2023) was a year of impatience and moving fast under the new CEO. Moving fast was the most important thing across the company. We did spectacular things, we made aggressive and ambitious decisions, we moved the fastest I saw Intercom move, despite being 3x bigger than when I first joined. This was the right thing to do in the wake of AI even though it came with real trade-offs on our product health that we had to pay down year later.

## People & management

### 1. Fair, objective and transparent are the most important principles in compensation and performance evaluation.
I deeply believe in these three principles when evaluating performance or deciding on compensation incentives or raises. There are plenty of processes that help you achieve them - transparent and written expectations across levels, calibration and promotion meetings, obligatory peer feedback, compensation bands or compensation policies.

### 2. Find the right-shaped opportunities to help people thrive and achieve impact.
Sometimes folks don’t perform and it’s important to ask yourself if there is a better place in the organisation that would benefit from their skillset, instead of managing them out from their current position or coaching them to grow where they are. I found a few folks who performed amazingly after we found the right-shaped opportunities for them. But don’t let under-performance slip when there are no high-confidence opportunities in sight. You can’t manage underperformance by moving people around.

### 3. There is always a reason why someone is underperforming, learn that before you act.
Empathy and trust will let you understand why someone is underperforming. As a manager, my job is to get the best out of my team and it’s critical to understand the problem - why someone might be underperforming? Stress at home, financial problems, personal conflict or health problems can happen behind the scenes without your reports notifying them about them upfront. Dig and understand what’s the reason to find the right shaped way forward before you act.

### 4. Building managerial support networks in your team.
Management is a lonely job, your managers should rely on each other. I [wrote more about it here](https://engineering-manager.com/2023-07-17/power-of-peer-coaching-for-managers).

### 5. Routines can help you avoid burn-out and keep you on track.
Planning, reflections, operation data reviews - it all creates a personal operating system of routines that provide time and activities that help you get perspective. It’s self-management practice that helps me to not burn out and stay on top of what’s going on. It helps me to identify and know when to take a break. Dropping my routines is usually the first sign that I am going sideways and need to take care of myself.

## Industry and customers trends

### 1. Support is a slow moving industry. It takes a lot of energy and work to see the change.
Support orgs on average have high inertia. Change management and switching costs are expensive, and if it’s not showing clear ROI, it’s very tough to drive any change in tools or processes. It works, but it’s tougher than it sounds. Adoption is a big challenge.

### 2. Support has a chance to generate money, not only be a cost centre.
There is a huge opportunity ahead of Customer Support orgs to generate revenue and not only be a cost centre. AI is the tool to enable it. Remove the friction by engaging with AI bots and create exceptional co-pilot experiences that help agents drive value, not just manage questions.

### 3. Sales and marketing are moving very fast. Experiment and expect immediate results.
Sales and marketing have easier adoption curves, but way harder to retain. Showing ROI and before/after data is critical. If you can’t deliver, the tool or the process is out. I ran a rollout of an embedded iPaaS solution for our high-spend customers and failed. With reflection, we have set the revenue bar too high to show ROI quickly and never earned sales team trust that this can work. Go for quick, visible ROI.

### 4. Omnichannel is great, but email is still considered the primary channel for many.
During my time in Outbound product at Intercom, it was clear that customers wanted email. They loved our omnichannel. Especially in-apps with their impressive engagement rates. But new things do not always replace old, and often rather expand the landscape. Marketers need reliable, powerful email tools to be your primary tool.

### 5. In-app is the best channel for engagement, support and x-/re-/up-selling.
I loved to see some exceptionally well designed proactive support, consisting of product tours, banners, tooltips and timely send in-app messages. It makes the experience of onboarding magical. I loved that we could message tens of our customers and ask them for feedback, and they would reply in less than 24h, with additional thoughts. It’s magical both ways.

### 6. Partners need well designed incentives and reporting to drive action, and it’s harder than sales because it’s based on trust and relationship, not a work agreement.
Intercom had to deprioritise our investment in App Partnerships and thus we weren’t able to build valuable long-term relationships with our key developers. When we ultimately had some ideas on how they could help us and our common customers, it was tough to get traction. Partnerships are amazing but need to be nurtured, based on the right incentives and give insight into the right reporting to drive behaviours.

### 7. Data management is evolving, painful, and it’s tough to find a silver bullet.
We owned a relatively flexible, broadly used data platform. It was the area of product with the biggest amount of feedback. But making an impact in this space was very tough. The market is evolving fast, with new approaches like ETL, reverse-ETLs, point-integrations, CDPs. Your customer base will be spread over multiple different “best ways of managing your data”. And because of that, there was never a single problem. It’s always been long lists of problems, similar in some shapes but different. To see meaningful change and improvement in data management, it’s all about strategy and consistency. Small investment shots here and there rarely work.

### 8. Data integrations are getting more complex the more you dive into them.
I learned to never underestimate x-system integrations. From the distance, they always sound relatively easy - if this then that, sync objects in system A with the same objects in system B. But the closer you get there, the more differences you discover. Small nuances, API rate limits, race conditions, retries and lack of idempotence, and many more angles. The devil is always in the details, and the jobs your customers are trying to achieve with these integrations are often similar but fundamentally unique. Integrations are super complex the deeper you get and don’t underestimate it.

### 9. Building easy to adopt integrations is removing friction, but reduces the TAM.
The most easy to use integrations are layers of abstractions on top of core capabilities (like APIs or iPaaS blocks). They remove a lot of friction, usually working out-of-the-box. What I realised over the years is that building broadly adoptable integrations is very tough as every company has unique needs, setup and their own internal IT mess to navigate. Building these abstractions makes it easier to adopt, but significantly reduces the TAM, which can end up with a lower number of customers using them versus a more complex but powerful version.

### 10. AI will fundamentally change how we build integrations.
However, I believe it will play out differently for deterministic, high-volume integrations and non-predictable, dynamic integrations. Co-pilot experiences will dramatically accelerate users in iPaaS tools like Zapier or Workato while creating repeatable, high-volume, predictable process automation. This will accelerate them, while maintaining today’s reliability and ultimately deterministic behaviour of these integrations. 

AI agents that can reason about which tools to use and adapt to dynamic and unpredictable input will remove a need for building any integrations (outside of API capabilities) in human-triggered activities like reporting, data exploration or asking for help.

## Product & Technology

### 1. Betting only on the future is like shorting the present and it carries opportunity cost.
Intercom is an extremely innovative company and our senior leadership is excellent in having great insight into what trends will make waves in the market. But betting only on that, building only for the future, is like shorting the present and thus carries the opportunity cost. I think we lived that both with emails and phone, being confident for years that these channels were dying just to realise year by year that they were not shrinking at all. I find it a useful mental model when you think about the future trends - would you short present and be willing to pay the premium if you continue to be wrong?

### 2. Every developer needs to have AI in their toolbox.
When we worked on Fin, our AI team was fundamental to making it work, we wouldn’t be able to do it as a product team. However, as time goes by, more and more excellent features have been built into that system directly by our product teams. The initial push might need specialisation, but it’s important to expose AI to your org as soon as possible. Another engineer shared with me this anecdote - 2008 companies had whole teams that did “mobile design”. Only a few years later it was part of everyone’s job.

### 3. Monoliths can scale very well, give solid deployment safety and high leverage for developer experience and observability.
I continue to be amazed how well Intercom Rails monolith scaled over the years. Exceptional engineering decisions, keeping it simple, huge leverage that our only developer experience team has, and sticking to proven cloud technologies helped us navigate growth really well. Monolith is also something that keeps our ability to ship very fast, very often (by sweating things like fast rollbacks or quick CI). We found a few services we owned painful to work with in comparison - teams had to be slowed down by maintaining their own dependencies, deployment pipelines or infrastructure updates. I never worked in professional, high-scale microservices architecture but can’t wait to learn about the trade-offs there.

### 4. How often you ship is critical. [It’s your heartbeat.](https://www.intercom.com/blog/shipping-is-your-companys-heartbeat/) 
You might not ship it to customers, but there is always a way to ship to production in a safe way. If you can’t find it, keep looking. It builds muscles for accelerating even more when necessary, and keeps you from losing impatience culturally. Show off the progress, ideally on a regular company-wide demo.

### 5. Create plenty of system models, mental models, apply different layers of abstractions. Looking at the same problem from different angles really expands your horizons and builds alignment.
At Intercom I learned that the best meeting starts when someone picks up the marker. System models, made up on the spot, adapted, evolved as you talk - all of this really help sharpen your understanding of the problem, its complexities and dependencies. Write them down, show them in your next conversation, ask for someone else’s mental model. Doing this work together with your partners and stakeholders accelerates how fast you collaborate and removes misalignment. Shared mental models are very useful for moving fast. 

### 6. Own and know your data, without excuses.
I had plenty of excuses in the past for my understanding of product data without the help of data analyst to craft queries and analysis techniques. I have no mercy for myself since ChatGPT. The quality and confidence in which I can navigate my product space without a dedicated analyst has increased dramatically and I expect that everyone at least slightly technical can do it now.


I also want to say big thank you to everyone I worked with and that helped me form these lessons over the years. Intercom is a truly special company! 

Last but not least, thank you to my two long-term partners in crime that provide me with great review and feedback on that blog post, Stephen Forbes and Aidan Lynch!