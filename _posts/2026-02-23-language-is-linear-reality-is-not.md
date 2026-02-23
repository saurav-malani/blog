---
title: "Why AI Goes in Circles — It Gets What You Say, Not What You Mean"
date: 2026-02-23
tags: [ai, interface-design, constraints, language, systems-thinking]
excerpt_text: "The problem with LLM instability isn't intelligence — it's interface design. Natural language collapses multidimensional intent into linear text. The fix isn't better prompts, but a meta-layer that preserves constraint graphs."
---

You're building something with an LLM. You've gone back and forth a few times. It's mostly right, but the formatting is off. You say: *fix the formatting.* It does. But now the tone has shifted. You say: *match the earlier tone.* It does. But the structure you agreed on three messages ago is gone.

You're not imagining it. This actually happens. And once you notice the pattern, you see it everywhere — in code generation, in writing, in system design. Each fix quietly undoes something else.

The instinct is to blame the model. But the model isn't being careless.

Something else is going on.

Each time you give a new instruction, the model treats it as the primary objective. It optimizes for the latest thing you said. Unless you restate every prior constraint, some get quietly dropped. You're not iterating. You're overwriting.

This is sequential local optimization in a multidimensional constraint space. You're adjusting one axis at a time, but the problem lives in many axes simultaneously. Without a way to hold all constraints in place, each adjustment destabilizes the last.

But this doesn't happen with simple tasks. Ask an LLM to write a function, fix a typo, summarize a paragraph — it works fine. The circles start when the problem has multiple requirements in tension. Readability versus performance. Security versus usability. Brevity versus completeness. The more constraints that need to coexist, the more likely each new instruction collapses an old one.

So what's actually causing this?

## The Lossy Compression of Intent

Here's the uncomfortable truth: language is linear. We speak, write, and read one word after another, one sentence after another. But the problems we're trying to describe are not linear at all.

A system design problem doesn't live in a sentence. It lives across functional requirements, performance constraints, security boundaries, maintainability tradeoffs, environmental assumptions, and business priorities — all simultaneously, all in tension with each other. That's not a list. That's a constraint graph.

And every time you write a prompt, you're compressing that graph into a flat string of text. Some things survive the compression. Others don't.

What gets lost? The implicit stuff. The things you know but didn't say because they felt obvious:

- Which requirements are hard constraints and which are soft preferences.
- What the priority order is when two goals conflict.
- What must never regress, no matter what else changes.
- Which earlier decisions are load-bearing and which are flexible.
- What "good" looks like when you can't have everything.

You hold all of this in your head simultaneously. But the prompt captures only the explicit slice — the thing you said, not the full structure behind it. The model receives a compressed, impoverished version of what you actually mean.

That's not a prompting mistake. That's a structural property of the medium. Language is a lossy compression format for multidimensional intent.

## But Code Is Language Too — So Why Does It Work?

There's an obvious objection here: if language is the problem, how does code work? Code models the world. Code is written in language. And code doesn't oscillate the way prompts do.

It's a sharp question. And the answer reveals exactly where the real gap is.

Code is language — but it comes bundled with enforcement. Type systems, interfaces, explicit hierarchies, compiler checks, runtime validation, test suites. When a constraint is violated in code, something breaks visibly. There's feedback. There's enforcement. The structure isn't just described — it's *enforced*.

English has none of that. When you say *"make it scalable but readable and secure,"* you haven't defined what scalability means in this context, which quality dominates in a conflict, or what must never regress. You're relying on shared understanding that doesn't exist between you and the model.

Code works not because it's language, but because it's language plus structure plus enforcement. It makes constraints explicit and machine-checkable. English leaves them implicit and hopes for the best.

And that's exactly what gets lost in prompts. Not the words — the structure behind the words. The invariants you assumed were obvious. The priority ordering you never stated. The conflict resolution rules that exist in your head but never made it into text.

## The Model Isn't the Bottleneck

Here's what makes this especially interesting: LLMs are not linear internally. They operate in high-dimensional vector spaces. Attention mechanisms process relationships across the entire context in parallel. The model's internal representation is already rich, structured, and multidimensional.

The model *can* hold constraint graphs. It can reason about tradeoffs. It can maintain multiple objectives simultaneously — if it receives them in a form that preserves their structure.

But it doesn't receive them that way. It receives a flat string of text that has already lost most of the structure.

The bottleneck isn't inside the model. It's upstream — between your thinking and the model's input. We're using a low-fidelity interface to communicate with a high-dimensional reasoning system. The model is more capable than the pipe we're feeding it through.

## A Compiler for Intent

Once you see the problem this way, the shape of the solution becomes clear.

We don't need better prompts. We need an intermediate layer that preserves structure — something that sits between raw human thought and the model's input, and does what a compiler does for code: transforms a high-level representation into a form the target system can actually work with.

Instead of *Intent → English → LLM*, something closer to *Intent → Constraint Graph → Structured Specification → LLM*.

That layer would do what you currently do in your head (and often fail to fully transfer): extract the dimensions of the problem, declare invariants that must hold across iterations, assign priority ordering when goals conflict, make tradeoff rules explicit, and enforce regression checks so that fixing one thing can't silently break another.

A compiler for intent. Not replacing natural language — structuring it. Catching implicit constraints the way a type checker catches implicit type assumptions. Making the unsaid things said, in a form the model can actually hold stable.

This isn't science fiction. Pieces of it already exist — system prompts, structured output schemas, context management tools, CLAUDE.md files that persist instructions across sessions. But they're scattered and manual. The insight is that they're all solving the same underlying problem: preserving constraint structure that natural language drops.

The next interface for AI won't just be a text box. It will be a layer that helps you express what you actually mean — all of it, including the parts you'd normally leave unsaid — without flattening it into a string that loses the structure the model needs to stay coherent across iterations.

That's not a prompt engineering problem. That's an interface design problem. And it might be the most important one in AI right now.
