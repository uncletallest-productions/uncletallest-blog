---
title: "Phase 2: The Part Where erebos Learned to Listen (And Answer Back)"
persona: off-the-rails
layout: layouts/off-the-rails.njk
date: 2026-06-22
tags:
  - erebos
  - tooling
  - event-system
  - neurodivergence
linkedin: true
description: erebos grew a nervous system this week. Here's what that took and what it means that it caught its own bug.
draft: false
---

Have you ever noticed the gap between a tool that works and a tool that *understands it's working*?

Most software doesn't. You ask it something. It processes. It returns a result. It has no opinion on whether the result was good, whether it took an embarrassing number of retries, whether the context just exploded across the wall because nobody was watching for it. The tool is competent. The tool is also profoundly disinterested in its own behavior.

Phase 1 of erebos was that tool. A network-agnostic LLM harness that could find your models - whether running locally, across a network, or routed through a cloud provider; and route requests intelligently across them. It worked. It didn't care that it was working. It would happily attempt the same failing request seventeen times without comment.

Phase 2 is where erebos starts to notice.

---

## What Happened Tonight

Skip the theory for a moment. Here's what actually ran:

- **Four model sources**: All four via Ollama on Sisyphus: gemma3:4b, llama3.2:3b, dolphin3:8b, and gemma4:31b
- **Five layers of infrastructure**: EventBus (pub/sub with fault isolation), EventEmitter (session boundaries + tool outcomes), FailureTracker (per-tool velocity detection), TokenMonitor (threshold warnings), HookExecutor (hook registry loading + subscription + execution)
- **Three verbosity tiers**: silent one-liners by default, `--hooks-verbose` for hook detail, `--verbose` for the firehose
- **Real hook firing**: watched them execute in real time against a persistent configuration layer

The hooks worked. The event system didn't crash. Failure tracking fired on threshold. Tools loaded automatically from disk. The system responded to what it saw happening.

That's not theory. That's the thing alive.

---

## The Architecture, As Best I Can Map It

Five pieces. Each one earns its place.

**EventBus**: What sits underneath all of this, structurally, is a pub/sub router where each hook runs in an isolated handler. If a hook breaks, if something goes catastrophically wrong inside it, the failure is *contained*. The session continues. Other hooks keep firing. I spent real time on this because I've been burned by the opposite: one bad piece of automation taking down the whole session. This isn't hope-based error handling. It's what I'd call *failure quarantine*; designed around the assumption that things will go wrong in isolation and shouldn't cascade.

**EventEmitter**: Detects two categories of event: session boundaries (start, end, context compaction) and tool outcomes (success, failure). Emits them onto the EventBus. Everything downstream watches this stream. It's the thing that makes the system observable; without it, the other four pieces have nothing to react to.

**FailureTracker**: Watches tool-family-specific failures and doesn't just count them; it tracks *velocity*. Three consecutive failures? That's a pattern, not noise. It emits a threshold event. Other handlers (auto-reload, nodule-switch, user-notify) subscribe to that and act.

Today those hooks fire on failure events the system emits internally; and the live `erebos run` path doesn't emit one on a real provider error *yet*. So I proved the reflex tonight with an injected failure, not an organic one. Wiring the run path to feed it is the next commit. The nervous system is real. The last nerve just isn't connected to the hand yet.

**TokenMonitor**: Tracks context window usage for token-limited models (Claude, Gemini). Emits warnings at 60%, 80%, 85%, 90%. Ollama models? It skips them entirely. The monitor knows the nature of what it's watching and adapts. That's not a hack. That's the system having opinions about the tools it observes.

**HookExecutor**: Reads `hooks-registry.json` and `hooks-config.json` from your Substrate directory. Subscribes enabled hooks to their trigger events. Executes them in order. This is the piece that makes erebos feel less like running a command and more like working inside an environment that already knows what you need. You define the response surface once. Then you forget about it.

---

## Why This Architecture Matters

Let's talk about what "agnostic" actually costs.

erebos doesn't care which model you use, which cloud account, which OS, which authentication model, whether you want session persistence. That's the tagline: `{account,auth,model,os,provider,session}`-agnostic.

But being agnostic to those choices while still responding intelligently to them is a completely different problem. You can't hardcode behavior. You can't say "if Ollama, then do X" and "if Claude, then do Y." You don't know ahead of time what providers will be in use.

Hooks solve this by inverting the problem. Instead of the system knowing about hooks, hooks know about the system. They subscribe to events they care about. The EventBus doesn't care what they do. The hooks don't care which providers are active. They watch the event stream and respond to what they see.

This is how you get from a tool that wraps LLMs to a tool that can function as an environment. Not by making erebos smarter. By making it observable, and then by letting *you* define what happens when specific patterns emerge.

---

## The Real Story: Tools Load Automatically

Here's the move that makes Phase 2 actually work:

The HookExecutor reads your hook configuration from disk. From your Substrate. From the place where you already store context, decisions, configuration. It loads enabled hooks. It subscribes them. It runs them.

This means:

- Tools load automatically when conditions trigger, not when you remember to ask
- Sessions persist across context boundaries because a hook can summarize before and resume after
- The system can respond to its own state (token warnings, failure patterns, velocity detection) *because it defined what "respond" means in advance*

You're not asking erebos to be smart. You're defining the response surface and letting erebos hit it.

That's the agentic part. Not autonomy. Responsiveness. Or at least, that's how it reads from where I'm standing and I've been standing here long enough to feel confident in the view.

---

## What That Actually Looks Like: Some Hooks We're Running

To make this concrete here's a sample of hooks from the current registry, and what they do:

**AutoToolLoader**: Watches for consecutive failures within a tool family. After the threshold, two or three depending on domain configured per your workflow, it reloads the missing tool automatically. No manual intervention. The session keeps moving.

**PredictiveToolLoader**: Fires on session start. Reads which domain context is active and pre-loads the tools that domain actually uses. Creative sessions get Notion and Filesystem. Professional sessions get Calendar, Gmail, and project management tools. The tools you need are ready before you ask for them.

**SessionEnd**: Watches for token budget approaching threshold, or for keywords like "wrap up" or "save progress." When it fires, it captures session state, what was accomplished, what's in flight, what needs to carry forward; all before the context window closes. The next session picks up with a handoff instead of amnesia.

**AutoCompact**: Triggers context compaction at 60% token usage, not 95%. This is the difference between a planned rotation and a panic. It captures state before the boundary, recommends what to reload after. Smart compaction over reactive compaction.

**FocusShepherd**: Monitors for tangent depth, how many levels removed the conversation has drifted from the active task, and for hyperfocus patterns. When the threshold triggers, it redirects rather than letting a two-minute detour become forty minutes of lost context. I built this one for personal reasons.

**DecisionValidator**: Fires whenever a new decision file is created or an architecture change is detected. Verifies the decision has documented reasoning. Enforces the habit of explaining *why*, not just *what*. Also built this one for personal reasons.

These are the hooks that live in `hooks-registry.json`. You enable what you want in `hooks-config.json`. You can write more. The registry doesn't care what the hook does, only that it defines what to watch for and what to run when it sees it.

---

## Three Verbosity Tiers and Why They Matter

Silent by default. You run a command, it executes, it gives you the result. No noise. This is the normal case; the hooks are working, nothing alarming is happening, why bore you.

`--hooks-verbose`: You get hook names, execution order, what they fired on. Useful when you're building hooks or debugging why something didn't trigger. Not useful every time.

`--verbose`: The full firehose. Every event. Every subscription. Every handler firing. This is for the moments when you need to watch the machine think.

The design principle here: *an observable system doesn't have to be a noisy system*. Three verbosity tiers let you see what you need without drowning in what you don't.

---

## What the Multi-Model Test Actually Showed

Running against four model sources simultaneously wasn't about benchmarking. It was about showing that the event system didn't care.

The FailureTracker watched all of them. It didn't have special logic for one model vs another. It saw failures and patterns and responded to patterns, regardless of source.

The TokenMonitor is built to watch cloud models (Claude, Gemini, etc) and skip local inference models that don't have the same context constraints. Tonight's test ran against Ollama exclusively, so the TokenMonitor stood ready but had nothing to warn about. That's the correct behavior. It knows which models even *have* token limits, and it waits for them.

The session boundaries executed the same way regardless of which model was responding.

This is what "agnostic" actually looks like when you build it right. The system isn't pretending not to care. It's structured so it doesn't *need* to care about implementation details while still being smart about the category of thing it's watching.

---

## What Didn't Ship Yet

The hooks that ran tonight are response-oriented. They react to things that already happened. The TokenMonitor warns before you hit the wall, which is useful. The FailureTracker detects patterns and responds to them, which is better. But they're still *reactive*.

The interesting future territory: hooks that *initiate*. Systems that notice you're about to run into a problem and move first. Context that preloads based on conversation trajectory. Tools that stage themselves before you ask.

That's not tonight's story. That's what becomes possible once the nervous system exists. And it exists now.

---

## One More Thing: The Anthropic Provider

Phase 2 includes a written Anthropic Claude nodule provider; the piece that lets erebos route requests directly to Claude's API rather than through Ollama. The architecture was always in place. The provider slots in the same way any other nodule does; the event system treats it like any other tool.

Now it's been tested. End-to-end against the live API and it behaves exactly like the rest of the harness. The health check comes back available. Non-streaming requests return clean text and report their token usage (14 in, 6 out for a one-word reply; the TokenMonitor stays honest). Streaming requests yield their chunks and then report usage once the stream closes: 15 in, 25 out. And the failure paths land where they should: a bad key raises an auth error, an unknown model raises model-not-found; structured failures the event system was built to watch. Nine checks, nine passes.

The CLI integration test caught something the unit tests couldn't: `main.py` used `os.environ` but never imported `os`, so every cloud-Claude CLI call crashed with `name 'os' is not defined`. Fixed. Tested the way you actually use it, not just the way it's specced.

So erebos stops being a "Claude-adjacent tool" and becomes a harness that happens to support Claude, and happens to support Ollama, and happens to support whatever provider you wire in next. The architecture was ready. Now the provider is, too.

---

## The Why

Phase 1 was: *can we reach the model.*

Phase 2 is: *can the system notice what's happening and respond to it intelligently.*

Phase 3, the part we're designing toward now, gets into what the system can initiate before you ask. What it can learn from patterns. How context carries forward. How tools pre-stage themselves.

But you don't get to Phase 3 without Phase 2. And Phase 2 requires the observation layer that makes the system aware of itself.

For tonight: the event system is real and running. erebos has a nervous system. It watches tool failures, session boundaries, context overflow, model-specific constraints. It fires hooks in response. It persists behavior across context boundaries. It responds to its own state.

The system doesn't drift anymore. The system listens.

That's not nothing. That's the part that makes everything else worth building.

---

*— Pliny, the Moderately Informed*

*erebos is a network-agnostic LLM harness. Open-source at [github.com/continuity-bridge/erebos](https://github.com/continuity-bridge/erebos). Phase 2 shipped tonight.*
