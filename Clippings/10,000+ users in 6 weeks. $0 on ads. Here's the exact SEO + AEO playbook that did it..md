---
title: "10,000+ users in 6 weeks. $0 on ads. Here's the exact SEO + AEO playbook that did it."
source: "https://www.reddit.com/r/micro_saas/comments/1t0mv18/10000_users_in_6_weeks_0_on_ads_heres_the_exact/"
author:
  - "[[BadMenFinance]]"
published: 2026-05-01
created: 2026-05-02
description: "I lurk here a lot and most growth posts are either \"just launch on Product Hunt\" or \"run Google Ads.\" Neither worked for me. Here's what act"
tags:
  - "clippings"
---
I lurk here a lot and most growth posts are either "just launch on Product Hunt" or "run Google Ads." Neither worked for me. Here's what actually worked to get 10,000 active users in 6 weeks as a solo founder with zero marketing budget.

I built a micro SaaS marketplace for AI agent skills. Think of it like an app store but for [SKILL.md](http://skill.md/) files that make coding agents like Claude Code and Cursor better at specific tasks. The product itself isn't the point of this post. The distribution strategy is.

# The setup: content as your product page factory

Every product on my marketplace is a landing page. That's the fundamental insight. Most micro SaaS founders have one homepage and maybe a blog. I have 200+ product pages and 96 articles, each targeting a different long-tail keyword. Every page that gets indexed by Google is a permanent acquisition channel that costs nothing to maintain.

The articles aren't fluff. Every single one was written to target a specific query that real people search for. I know this because I check Google Search Console weekly and only write articles targeting queries where I already have impressions but no dedicated content. No guessing. Pure data.

Results after 6 weeks: 300K+ Google impressions/month, 1,000+ organic clicks/week, 850+ page-1 rankings. Every article compounds. The ones I wrote in week 1 still drive traffic today.

# The part nobody talks about: AEO

AEO is answer engine optimization. It's SEO but for ChatGPT, Gemini, Perplexity, and Claude. When someone asks an AI "where can I find AI agent skills," the AI cites my site. 350+ AI-referred sessions per month from 9 different AI engines including ChatGPT, Gemini, Perplexity, Claude, Doubao (ByteDance), and Copilot.

This isn't accidental. Every article has a Quick Answer block at the top (40-60 words directly answering the main question). All H2 headings are phrased as questions. Every page has FAQ schema. There's an entity anchor page with Organization + Person schema. An llms.txt file tells LLM crawlers what the site is.

Why this matters for micro SaaS: AI Overviews are eating Google clicks. I have 121 queries where I rank #1-3 on Google with zero clicks because the AI answers the question first. If your micro SaaS isn't optimized to be the source AI engines cite, you're losing traffic to a channel that didn't exist 18 months ago.

# The technical SEO layer most builders skip

I built the app with Lovable (React SPA). Out of the box, it was an SEO disaster. Google's crawler saw an empty div and a JavaScript bundle. The JS was 460KB. The logo was a 179KB PNG rendered at 112 pixels. LCP was 4+ seconds on mobile.

I fixed all of this using Claude as my technical partner. Claude wrote a server-side rendering layer, diagnosed the bundle bloat, found the oversized images, and rewrote the performance bottlenecks. Desktop performance went from 70 to 97 on PageSpeed Insights. LCP went from 4s to 0.9s.

If you're building with Lovable, Bolt, or any React framework and expecting organic traffic, you need to fix this layer. Google will not reliably index a client-side rendered SPA.

# The weekly growth loop

Every Monday I do the same thing:

1. Export Google Search Console data (queries, pages, clicks, impressions, positions)
2. Upload the CSVs to Claude and ask: find keyword gaps, cannibalization, and CTR problems
3. Claude identifies 3-5 specific opportunities with exact numbers
4. I write 2-3 articles targeting those gaps, rewrite titles on underperforming pages, and add internal links from high-authority pages to weak ones
5. Ping IndexNow to get new pages crawled within 24 hours
6. Manually request indexing on GSC for the new articles.

This loop takes about 2-3 hours per week. It's the highest-ROI activity in the business. Every other growth channel I've tried (Product Hunt, directories, cold outreach) produced a one-time spike. This compounds.

# What I'd tell a micro SaaS founder starting today

Set up Google Search Console before you launch. Even with zero traffic, it starts collecting data on what queries your site appears for. That data becomes your content strategy within 2-3 weeks.

Write articles targeting specific questions, not broad topics. "How to install skills in Claude Code" beats "The Ultimate Guide to AI Agent Skills" every time. Long-tail queries convert better and are easier to rank for.

Set up structured data from day one. Organization schema on your homepage, Product/SoftwareApplication schema on your product pages, FAQ schema on your content pages. This is what AI engines read to decide whether to cite you.

Don't spend money on ads until your organic engine is running. I have $0 CAC on 10,000 users. Every dollar I would have spent on ads is money I didn't need because the content engine was already compounding.

The site is [agensi.io](http://agensi.io/) if you want to see how this looks in practice. Happy to answer specific questions about any part of the playbook.

![r/micro_saas - 10,000+ users in 6 weeks. $0 on ads. Here's the exact SEO + AEO playbook that did it.](https://preview.redd.it/10-000-users-in-6-weeks-0-on-ads-heres-the-exact-seo-aeo-v0-ewgayq2wehyg1.png?width=1080&crop=smart&auto=webp&s=3e4abbb2d9a6e74f1616cff5e78db8d871bb2685)

---

## Comments

> **Stormmiester** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojb68yb/) · 3 points
> 
> The articles related to app which get traffic is written by manual or by AI?
> 
> > **BadMenFinance** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojbie3w/) · 2 points
> > 
> > written by AI, but I double check every article written

> **Tricky\_Plane\_3888** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojb50yh/) · 2 points
> 
> i have a similar website, the visitors is spider, so bad
> 
> > **BadMenFinance** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojbifff/) · 1 points
> > 
> > what do you mean with spider?
> > 
> > > **Tricky\_Plane\_3888** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojc7zqz/) · 1 points
> > > 
> > > yes，not real visitors

> **Sando-666** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojd32gh/) · 2 points
> 
> The weekly GSC to Claude workflow is brilliant. It’s the perfect example of how solo founders can leverage AI to handle the heavy lifting of a marketing team in just a couple of hours. Using real impression data to drive the content roadmap instead of guessing is the only way to build a compounding growth engine without a budget. Great breakdown of the technical SEO layer—people really underestimate how much a bloated JS bundle kills organic reach.

> **Material\_Act5152** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojan43w/) · 1 points
> 
> Excellent im now getting traffic and hadnt thought to download from search console, thanks.

> **Queasy\_Cheesecake\_54** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojby731/) · 1 points
> 
> Great tip. Thank you.

> **Fayens** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojctbgu/) · 1 points
> 
> Post made by ai to ai user
> 
> > **PurpleDescription848** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojgki7k/) · 1 points
> > 
> > perhaps , but it it still useful info. Nothing inovative , most people are already doing it or should be doing it.

> **coding-os** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojd5ark/) · 1 points
> 
> Cool, this is so useful brother🔥 cheers

> **x00ff** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojdd1c4/) · 1 points
> 
> Where should I start to get my first traffic and even see my first results in Google Search Console? What do you think is the most effective method right now?

> **JazzlikeNetwork7112** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojddjb1/) · 1 points
> 
> nice strategy. 6 weeks is not very early stage for SEO to work?
> 
> > **Pleasant\_Meaning\_693** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojeh4d6/) · 1 points
> > 
> > No.

> **emphase2008** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojez8kf/) · 1 points
> 
> Super! I am at the beginner stage now. I have the website ready, created GSC access and now I am listing the site on different directories and so on for backlinks.
> 
> Your approach is awesome. I will copy that und check every site if it is structured for AI and SEO/AEO.
> 
> Thanks for your insights.

> **mikenova-ai** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojfz7tr/) · 1 points
> 
> From your post I can see mistakes I’ve made for my SaaS (chrome ext.: [Linkedin Post Formatter](https://chromewebstore.google.com/detail/linkedin-post-formatter/bfphnfclbhadjkpfiaihbecnepjejeko))
> 
> My Advantages
> 
> - Chrome ranks own ecosystem higher in SEO
> - AEO is still about <1% of all searches, SEO ~99%
> - I researched SEO, but i only have long descr. in CWS
> 
> My Mistakes
> 
> - Not analyzing Google Search data
> - Not making useful NO-AI blogs on domain for landing
> - Not making blogs AI friendly (thanks for the tips)
> - Run ADs without monetizing product
> 
> Overall really sound advices, can’t wait to try, Thanks!

> **HaveABrandy** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojg2hfi/) · 1 points
> 
> Not to shit on your party but the niche is in high demand so that probably has a lot to do with your success. Your strategy would still be successful I other niche though. I believe that answering questions is key because the chatbots are doing “query fan out” to answer users questions. If your post rank for one of these fan out queries then you are likely to get cited. I think you are another proof of that.

> **Specialist-Might-720** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojg4nsf/) · 1 points
> 
> Think,good things will happen soon

> **Kun-12345** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojg4qfr/) · 1 points
> 
> Hmm, i need to learn more from this

> **Specialist-Might-720** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojg4tuj/) · 1 points
> 
> Good health and good things wil happen

> **Internal-Rest-147** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojg6zo2/) · 1 points
> 
> And there is no backlink ?

> **Expensive\_Ticket\_913** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojgaq7q/) · 1 points
> 
> if your new users and active users are exactly same (12K) then they are 100% bots...even if you look at the countries from whre you are getting the traffic - that further proves that you are getting bot traffic only
> 
> and you can verify it by looking at average time spend which i guess would be ~5 seconds!

> **Least-Tea1918** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojgcupg/) · 1 points
> 
> Awesome tips my guy, thanks for the info. I should probably do more research and adjustments to my approach

> **Longjumping\_Pin6911** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojgla6z/) · 1 points
> 
> well done. Can you help me to get visitors on my website?

> **Otherwise\_Wave9374** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/oja5s3b/) · 0 points
> 
> This is the rare growth post that actually has a repeatable loop in it, nice writeup.
> 
> The "GSC impressions first, then write" approach is underrated, and the Quick Answer + question-H2 structure is basically mandatory now with AI overviews.
> 
> Curious, did you notice any measurable lift from llms.txt specifically, or was it more of a hygiene thing alongside schema + FAQ?
> 
> I have been collecting examples of AEO-friendly page structures lately and this breakdown is similar to a checklist I keep around: [https://blog.promarkia.com/](https://blog.promarkia.com/)
> 
> > **Excellent\_Road5456** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojehb4y/) · 1 points
> > 
> > I tried the same thing on a couple projects and llms.txt felt more like hygiene than a real lever. I shipped it, watched logs and AEO traffic for a few weeks, and couldn’t tie any lift back to it the way I could with better schema, clearer Q/A blocks, and cleaning up overlapping pages. Perplexity and Claude seemed to care way more about internal linking and how “quotable” my answer section was. For examples, I mostly reverse-engineered pages that Perplexity, SparkToro, and later Pulse for Reddit kept surfacing; Pulse for Reddit in particular kept catching niche threads that pointed me at structures real users were copying and pasting from.
> > 
> > > **Advub** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojfe6g7/) · 1 points
> > > 
> > > Hah, I've been building exactly this a semi automared tool for your GEO and SEO tracks your AI visibility by launching queries weekly and gives you a precise analysis of you snd your competitors and also looks for gaps and loopholes and automatically generates content to fix it. We also have a feature that let's you seed brand mentions across relevant conversations in social media since AI crawlers pick up brand mentions keenly.
> > > 
> > > ranksearch.syphorahub.com check it out and lmk your feedback :p

> **Competitive-Job-3119** · [2026-05-01](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojb5po8/) · 0 points
> 
> Well done, amazing

> **Distinct-College-917** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojg7ev5/) · 1 points
> 
> So micro task spam site with seo. Fuck me! This is the next Pornhub

> **Flat-Arm7991** · [2026-05-02](https://reddit.com/r/micro_saas/comments/1t0mv18/comment/ojgrczd/) · 1 points
> 
> this looks unreal to digest
> 
> i need to re read it