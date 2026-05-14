+++
title = "I owe my career to open-source communities. I'm not sure newcomers can say the same."
description = "Open-source communities shaped my career in ways no tutorial ever could. Now, as AI floods those same spaces with noise, I worry about the people arriving just a few years too late."
date = "2026-05-15"

[taxonomies]
tags = ["life", "career"]
+++

## Open-source is cool

Ever since I started learning programming, I knew I wanted to align myself with the open-source community, because of the following characteristics I found admirable:
- the way people built software in collaboration, with developers contributing from across the world, and the genuine care for good documentation, including internationalization (i18n) efforts.
- the impact open-source had on the industry. From startups to trillion dollar behemoths, everyone relied on open-source software for critical workflows. Many teams maintain their separate forks to manage business requirements, push patches, or build features that align with their needs.

Perhaps the most impactful was also the hardest to articulate: I simply found the ecosystem, and the de-facto “marketing” around building in public, genuinely *cool*. And I knew I wanted to contribute to open-source and “build my credentials”, which in turn would elevate my career.

## Open-source has defined my career

In 2021, I’d chosen Python over Javascript as the language to learn, build some projects, and like thousands of others, break into tech. After learning the basics, came the time to learn web development.

I picked up FastAPI instead of Django, because I saw [this talk](https://youtu.be/37CcB2GBdlY) by FastAPI founder [tiangolo](https://github.com/tiangolo), and thought that async-first and type safety in web apps sounded great and was the future, even though I barely knew what Python types were at the time.

### Finding my people

Alongside the coding, I loved reading opinion pieces, GitHub issues, and tech discussions on Reddit and Discord. Over time, I began participating as an enthusiastic beginner — slowly making friends, finding mentors, and learning things I absolutely wouldn’t have learnt otherwise.
A mantra I’ve carried through every role since, including my recent transition to DevRel, is that you don’t have to be the best: by being passionate, curious, and proactive, you can make friends and be remarkable.

### Landing that first role

I wanted to make a mark online, and besides coding, that meant being active in these communities, learning how they thought about things, being genuine, open to feedback on my code, and being somewhat opinionated.
I couldn’t be the best developer out there, but I could put my first real project, a CLI-based song downloader, out on Reddit, get genuine feedback from better programmers, and build some credibility in the process.
I landed my first job in July 2022, after about one year of learning programming, as a web developer for a small startup using FastAPI on the backend. Their CEO was impressed by my GitHub project, and the documentation and discussions around it. And so, the stars aligned, and I broke into tech.

### Delving deeper

In late 2023, I was obsessed with Rust, and found similar support in the r/rust subreddit and the official Discord server. This was some of the most enjoyable time I had programming: being blown away by Clippy’s helpful error diagnostics, admitting to a couple Rustacean friends about how I broke down in tears dealing with Tokio’s lifetime requirements, when I was [rewriting my CLI song downloader](https://dhruvahuja.me/posts/writing-rust-bindings/) in async Rust, and the joy I felt seeing my blogs being accepted for the [This Week in Rust newsletter](https://this-week-in-rust.org).

These small wins meant the world to me. They assured me that I’d “made it”, that I’m a part of these communities I have admired since the beginning, as at least a little more than an enthusiastic novice.
In fact, I owe my career to these communities, all the internet strangers who answered my silly questions (most infamously, “how many endpoints should a typical web API have?”, for which I received scathing critique), helped break down design patterns, and plan out my career. Many of these strangers have gone on to become people I deeply respect and look up to, and a few have even become lifelong friends.

## When the floodgates opened

Fast forwarding to 2026, and things have changed significantly. LLMs have reduced the barrier to entry for *generating* code, memos, and documentation of all kinds. You could ask Claude to generate a 10-page manifesto for your startup idea and it will gleefully do that.
While producing output has never been easier, it has made it incredibly difficult to be taken seriously. 

This has led to online communities and platforms being bombarded with low-effort content. Often, the users posting these projects have little idea about the internals, and even resort to using AI for responding to comments on their posts.
For users looking to participate in genuine discussions or understand the intent and implementation details, such posts are a waste of time and effort.

### Communities push back

To combat this flood of low-effort content, moderators of most communities have begun tightening the rules. For example, many large subreddits now outright flag posts that have a GitHub link, or are perceived as “AI slop” — the term given to this sort of AI-generated content.
The first rule in the r/python subreddit, titled `No showcase posts`, explicitly states:
```md
Due to an increase of showcases featuring AI content such as working with multiple AI models or wrappers around APIs, this is no longer allowed. Please post your showcases in the appropriate monthly showcase post or a daily thread instead.
```
The r/programming subreddit, which has ~290K weekly visitors, [banned all AI-related content](https://www.reddit.com/r/programming/comments/1s9jkzi/announcement_temporary_llm_content_ban/) for the month of April, with the community responding to the change positively in a [follow-up thread](https://www.reddit.com/r/programming/comments/1t4odyl/looking_for_feedback_on_ai_content_in/) started by the moderators to gather the community’s feedback.

### Open-source repositories under siege

Similarly, many renowned GitHub repositories have adopted extreme measures to counter this problem.

[cURL](https://github.com/curl/curl), for one, completely [scrapped its bug bounty program](https://github.com/curl/curl/pull/20312). [Daniel](https://github.com/bagder), cURL’s founder and development lead, published a blog post a few days later explaining why the problem had become untenable.

In the comments, [Piotr](https://github.com/ppkarwasz) from the Apache Log4j team further detailed their own struggles — overseeing an influx of low-effort reports to the Log4j Bug Bounty Program since July 2024. This was the time when engineers truly began harnessing the power of LLMs for speeding up their development workflows.


![Log4j AI Slop Comment](/images/curl_blog_log4j_comment.png)
_Credit: [Daniel’s Blog](https://daniel.haxx.se/blog/2026/01/26/the-end-of-the-curl-bug-bounty/#comments)_

Although I have cherry-picked a few well-known examples here, I am certain these changes are not limited to larger communities or programming codebases. Communities of all shapes and sizes, including individuals, have been struggling under the weight of AI-driven activities.
By this point, almost all of us have run into a wall of overly-verbose Markdown documents, complete with elaborate Mermaid diagrams and references to dozens of sources for further reading, at work or elsewhere.

Ironically, in my experience the Mermaid diagrams often do more to explain the concept than the pages of generated text.

## An uphill battle for newcomers

While the tightening grip on public platforms affects everyone, I believe newcomers have it the worst.

Those getting into the tech industry now face an uphill battle: an already tough job market now sees tech companies use AI to automate the tasks that junior engineers used to take on, learn from, and grow in seniority over time.

Couple that with the heightened scrutiny on content shared in public spaces, and the opportunities to get into open-source have narrowed significantly. Maintainers are already burdened, other passionate community members are burnt out by AI slop, and people increasingly become suspicious of anything that seems AI-generated (and it often is).

I think the current mindset adopted by communities against AI-slop can best be encapsulated by slightly tweaking the Python adage on duck-typing:
> If it sounds like AI and reads like AI, then it must be AI.

And honestly, they're not wrong. Still, rather than give in or leave things to deteriorate further, these communities are fighting back. The subreddit bans, tighter rules, and honor systems are all steps taken to stem the flood. While these actions might seem harsh or extreme to many, moderators and maintainers have deemed it necessary, if the care for craft is to be maintained.

Through all of this, I consider myself genuinely fortunate to have found those people when I did — early, when I needed it most. I've witnessed what made open-source great up close, and I'm afraid that things might never be as good as they were (or seemed to be, personally), particularly for the people just now finding their footing.

I don't think it's all doom-and-gloom though. The adaptation processes have already begun, and I believe they will strike the right balance. But for the newcomers, the path is narrower and the signal harder to find among the noise. I hope they still get to experience what I did.
