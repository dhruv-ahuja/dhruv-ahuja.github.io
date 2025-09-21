+++
title = "My Quest to Find the Perfect Self-Hosted Notes App"
description = "My back-and-forth journey to go self-hosted with my notes."
date = "2025-02-11"

[taxonomies]
tags = ["self-hosted", "notes", "productivity", "tools", "silverbullet"]
+++

## Backstory

My search started with me looking for a good open-source notes app as I realized that my UpNote premium subscription was costing me the same as my VPS per year. I also felt excited to scratch the itch to tinker with technology, which I’ve had for quite some time now.

My two main requirements were that the app needed to support all Markdown features while preferably offering some synchronization method. It also needed to have either an Android app or work well as a PWA so that I could jot down ideas or update existing TODOs and notes quickly.
How I was going to backup my notes if sync wasn’t available, was a problem I intended to solve later.

## What about Obsidian, you might ask

Now you might ask me what’s wrong with using Obsidian, as there are ways to sync notes for free. I tried SyncThing and found that I didn’t like the idea of merge conflicts resulting in two versions of the same file when editing from phone.

I still use it daily for neat and structured work notes. Also, vim mode feels great to use (even though I know just the basics of vim). I take rough notes during meetings and discussions in Apple Notes, which I then rewrite and add to Obsidian or mostly directly to Notion if they need to be documented or tracked (which is almost always the case).

## Apps I’ve tried

- Affine - was the first app that I tried, seemed promising as a free alternative to Notion, which I use regularly at work and quite like. However, it kept using cloud storage, which from what I understood, should not have been the case. This irked me out a lot, and I stopped using it then and there.
- Outline - was the one I was very interested in, being another Notion-like knowledge base app. I tried a few approaches and spent a lot of time trying to make it work on-and-off over two weeks, but couldn’t. At this point I was quite frustrated and was reaching the end of my tether and dropped it.
- UseMemos - was the first breakthrough of sorts, something that was just worked, was self-hosting oriented and could be easily used through a PWA. I had started migrating few of my notes before feeling the lack of a folder hierarchy.
- nb - is the next app that I found, a terminal-based all-in-one app that boasts everything that you could ask for in a notes app. The sync was perfect, you make a change and you see the git sync take place immediately.

`nb` was great, however, this being CLI-only was a major blocker. I was ready to use it in tandem with `useMemos` for the time being and thats what I decided to do...

## The transition to SilverBullet.md

Meanwhile I had mentioned my experience and requirements over to the guys at TEO Discord, and got a great recommendation from the always-reliable Satya Bhat, who recommended [Karan’s very-well written article on Self Hosting Outline](https://mrkaran.dev/posts/setting-outline/). It featured a workaround for the OIDC based login setup that had blocked me earlier. I was going to go along with it, when Karan himself recommended [SilverBullet.md](https://silverbullet.md) to me, which had seemed interesting.

I tried it out and immediately fell in love. The app just worked great out of the box, had a good feel to it and was intuitive to use. The command and open page “quickbars” make it snappy to trigger a command, example git sync or change to a separate file, like one would do with Obsidian.
About git sync, it required some work but what is maybe an hour of setup for a tool you’re going to be using for a long time. I had to enable the `git` and `Github` plugins, provide a git repo and a Personal Access Token to enable sync. SilverBullet was unable to push to my empty repo I’d just created, so I entered the container and set the remote, and that was it.

It is quite lightweight, consuming ~350mb of system RAM at the time of writing. Did I also mention it has vim mode and the font looks cool? Yeah, I’m hooked. I’ve written this article itself using SilverBullet, and it has been a great experience:

![The Blog Post](/images/notes_app.png)

## The end (for now?)

I am hooked on SilverBullet. I am not a power-user making graph-like notes or interlinking them together, as such I feel like it offers me all the features that I would want.
I will miss UpNote’s share notes feature available to premium members, which syncs your shared notes in real-time, has been particularly useful for me when gathering feedback on work-in-progress blog posts.

Overall I’m really happy with my choice, and it seems thus far to be well-worth the time investment. I would whole-heartedly recommend it to anyone that has similar requirements as mine or wants to try a self-hosted option.

That’s it, thanks for making it till the end!
