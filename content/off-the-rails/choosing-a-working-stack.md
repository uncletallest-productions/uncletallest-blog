---
title: Choosing a Working Stack That Works
persona: off-the-rails
layout: layouts/off-the-rails.njk
date: 2026-06-14
tags:
  - neurodivergence
  - tooling
  - process
linkedin: true
description: Picking a tech stack isn't just a technical decision. It's a self-knowledge problem.
draft: false
---

Everyone has opinions about tech stacks. Most of those opinions are about performance
benchmarks, ecosystem size, or whatever the last conference talk was about.

None of that is wrong. But for those of us whose brains work a little differently,
there's a question that comes before all of that:

**Will I still be able to work in this when I come back to it after two weeks away?**

That's the real filter.

---

I didn't arrive at this filter through wisdom. I arrived at it through the specific
misery of returning to a project, knowing exactly what I needed to do, and losing
the thread somewhere between "open a terminal" and "what directory was I in again."

The problem isn't distraction. The problem is **decision density** — the number of
small choices standing between you and the actual work. Each one is cheap in isolation.
Open a terminal. Navigate to the right directory. Type the right command. Wait for the
thing to load. Switch windows. Each step is maybe three seconds. Each step is also a
place where your brain can decide it has something more urgent to attend to, and by
the time you've noticed, you're reading about competitive disc golf or reorganizing
your bookmarks.

For an ADHD brain, every step between *intention* and *momentum* is a potential abort
point. Stack selection, if you're honest about it, is largely an exercise in counting
abort points and deciding which ones you can live with.

---

I did not come to this understanding cleanly.

My current environment started as an XFCE ISO. I knew I didn't want XFCE — I've
never found a desktop environment I actually wanted to live in — but it was a known
quantity. A reasonable place to start subtracting from. So I stripped everything
out of it. Not some things. Everything. Then I started building what I actually wanted
on top of what was left.

The result is Stoa — what I've taken to calling a "distroette," a stripped-down
Wayland environment built on labwc, greetd, and tuigreet. Svelte by design. Opinions
exercised throughout.

Some of those opinions, since we're here.

I like some eye candy. I don't need to live in a spartan void to feel productive.
What I need is for the pretty thing to also *work*. Kitty is a beautiful terminal.
Kitty also doesn't have a right-click context menu with copy and paste. That's not
an exotic feature — that's functionality that has existed since roughly forever. When
your beautiful tool fails that test, it's telling me something about whose workflow
it was designed for. Mine involves a mouse sometimes. Next.

I don't need 47 tools to do the job when three of them have the one feature I
actually need. The real cost isn't the tools that don't work — it's the time spent
finding out they don't work. Every download-evaluate-discard cycle is steps I didn't
take toward the actual work. That overhead goes on the abort-point ledger too.

And if your application has 2,307 features and I need four of them: I'm probably not
downloading it. Feature count is a signal. A tool that tries to do everything has
made tradeoffs around the things I care about — legibility, for starters. An
eight-point default font is the kind of tradeoff made by someone who doesn't have my
eyes or my age. I know, I know. I'm old. I still have opinions about it.

It will eventually become a build script on top of a Debian Trixie netinst. Starting
from XFCE and subtracting was the long way around, but it's how I learned what
bedrock actually looked like. You sometimes have to strip something to the studs to
know what the studs are.

---

Here's the specific thing I built and why it matters.

Before: to open my main working environment, I opened a terminal, navigated to the
right directory, typed a command, waited for it to load, then typed another command
to move to the right window. Five steps. Five decision points. Five places to lose
the thread.

After: I click a pinned menu entry. I wait. I press Super+[1-4] to move between
sessions.

That's it. One decision instead of five, and the second one is spatial memory rather
than recall — my brain already knows where 1 is. It doesn't have to go looking.

This is not a preference. This is a prosthetic.

I also built stoa-control — something that embeds nm-applet, blueman-applet, and a
handful of other things into one surface, and custom-rolls the configuration options
that would otherwise require me to remember where rc.xml lives and what I was trying
to change in it. The goal everywhere is the same: the configuration layer should
require as few separate acts of memory as possible.

---

None of this is unique to environment setup. The same logic is why I eventually forked
MavosxWM and started turning it into koWM. I had opinions about how a window manager
should behave that were not being exercised by anything that already existed.
So I exercised them.

This is a pattern worth naming: there's a meaningful difference between *selecting*
a tool from available options and *exercising judgment* about what the tool should be.
Most engineers do the former. Neurodivergent engineers often can't afford to — because
a tool that fights you, even slightly, is a tool that introduces abort points, and
abort points are expensive in a way that doesn't always show up in benchmarks.

The filter question — *will I still be able to work in this when I come back?* —
is really asking: how many times is this going to make me remember something I
shouldn't have to remember? How many steps between intention and momentum? How many
ways can this interrupt the work with administrative overhead?

A good stack answers those questions with a shrug. It's just there. You don't have
to work to be in it.

---

I have opinions. This is what allows me to exercise them.

Now my guinea pigs — sorry, *users* — get to test whether they transfer.

*— Pliny, the Moderately Informed*
