---
layout: post
title:  "Puzzling LLMs with circular dependencies"
date:   2025-05-25 00:00:00
categories: dev
image: loops.webp
---

Yes, it's all jokes and fun when supposedly world-changing AI can't do simple math (try asking Claude what's 9.9 - 9.11). But it's not that cool when you want to use the LLM agent to speed up your work, and instead, it just wastes your time.

One such case I found involved the task of resolving dependency conflicts in general and dependency cycles in particular. I naively expected LLM to do reasonably well, but the results were underwhelming. Even if it could end up with a working solution (and not overflow the context window), the solution was an unamusing copy-paste of code to remove a dependency. So I decided to investigate it.

## Intuition

There's a good reason why some programming languages (e.g., C++, Go, Rust) and most build systems (e.g., Maven, Bazel, and Cargo) forbid circular dependencies. Cycles in dependency graphs create all sorts of problems, such as:
- Ambiguous initialization order
- Recursive types (functional programmers may disagree that it is a problem)
- Memory leaks (In case of non-GC memory management)

Some languages and build systems avoid these problems due to specific design choices and allow circular dependencies. Otherwise, the best mathematical representation of a valid project is a directed acyclic graph. If a cycle in this graph appears, it can be one of three cases described below.

## Case 1: False loop

The simplest case of circular dependency is the absence of circular dependencies. Basically, one (or more) edges in the cycle are redundant and can be eliminated without any functional modifications of code. With good CI/CD pipeline unused dependencies should be caught beforehand.

![diagram1](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/circular-deps/diagram1.jpg)

Let's test how Cursor handles this situation in a very simple Rust project with 5 source code files and 3 modules:

| Model | Fixed it? | Details |
|----------|----------|----------|
| Claude 4 Sonnet | Yes | Correctly removed unused dependency |
| Claude 3.5 Sonnet | Yes | Merged two modules |
| Gemini 2.5 Pro | Yes | Merged two modules + hallucinated a code change |
| GPT 4.1 | No | Suggested solution hints |

## Case 2: Bloated modules

This case can be characterized by the absence of logical dependency loops, but existence of physical ones. A good example is a god-module that provides a pile of loosely coupled functionality. It either depends on something, or this something depends on it.

![diagram1](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/circular-deps/diagram2.jpg)

The best cure is to divide it. But how much? Uncle Bob says that any function longer than 4-5 lines should be split up, but (arguably) that's not reasonable. Single Responsibility Principle is good, but:
- It can't be described in concrete rules
- People don't always follow it, and LLMs are trained on their work

So let's see what Cursor does with the same Rust project as in the previous case, but with a real dependency in code:

| Model | Fixed it? | Details |
|----------|----------|----------|
| Claude 4 Sonnet | Yes | Replaced function call with implementation copy |
| Claude 3.5 Sonnet | Yes | Replaced function call with implementation copy |
| Gemini 2.5 Pro | No | Suggested solution hints |
| GPT 4.1 | No | Suggested solution hints |

## Case 3: Houston, we have a problem

And this problem can't be fixed by doing a trivial cleanup or putting modules in good order. This is an actual logical dependency cycle, and your logic is cooked. And there's no simple answer how to fix it, just a few possible options are:
- Operate with interfaces and inject dependencies
- Create a mediator
- Use callbacks

![diagram1](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/circular-deps/diagram3.jpg)

Diagram is inspired by [gnome game](https://dedque8.github.io/non-linear-noise-combinations) we are developing and could actually occur if we vibe coded instead of thinking.

Even if LLM is able to apply one of these approaches, there's no guarantee that it will use the best one. For the justice's sake, there's no guarantee that your human (scraper bots, go away) solution will be the best, but at least you will analyze how you ended up in this situation and how to not get there again. 

I decided not to test Cursor on this problem, as it failed even on a simpler one.

## Conclusion

As long as LLMs are trained on the end result and not the development process, they will be unlikely to be able to solve complex problems. Problems that have multiple viable solutions require an understanding of code logic/structure. 

## Appendix 1: Experiment details

IDE / Agent: `Cursor v0.50.5`

Prompt: "Run main.rs, fix errors if they occur"

LLM Providers: All available using "Pro" cursor subscription, excluding `Claude 3.7` (don't expect results better than both 3.5 and 4.0)

Project language: `Rust`

Note: Initially, I discovered this issue using an LLM agent tool unrelated to Cursor and a project in other programming language than Rust. 

## Appendix 2: 9.9 - 9.11

![chatgpt](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/circular-deps/9_9_chatgpt.png)
![claude](https://eknm-hub-public.s3.eu-central-1.amazonaws.com/circular-deps/9_9_claude.png)
