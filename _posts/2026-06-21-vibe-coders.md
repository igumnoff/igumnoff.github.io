---
layout: post
title: "Are You Tired of Vibe Coders Yet?"
date: 2026-06-21 14:05:00 +0500
categories: ai, hiring, programming
---

![Vibe coding](/vibe.png)

So, I decided to share my experience hiring a new generation of programmers. It is absolutely wild. Sometimes I feel like I am not looking for developers, but for operators of a button labeled “make it pretty with AI.” They have already learned how to press the button, but reading what is written around it is apparently still a work in progress.

Let’s start with the simplest thing. Candidates often do not read the job description at all. They just send a cover letter in the style of “I am ready to work,” “.” or “I match the vacancy.” Period. Literally a period, Karl. Not even three dots to create at least some intrigue.

Or they send a standard message saying, I am such-and-such, I can do this and that. In other words, they simply duplicate their resume. Excuse me, but could you at least visit the company website, look at the product, try it, and write two or three normal sentences?

For example: “You have a cool AI agent. I have worked with LLMs, I checked out your product, and the idea is great. I would like to take part, gain experience, and be useful — I am ready to work for free for a line on my resume.” That is it. You already want to invite that candidate for an interview. About “working for free for a line on the resume,” I am joking, of course. But if a person has at least looked at the product, that is already an event on the level of “we found water on Mars.”

Instead, I get standard canned replies. Moreover, I have already tried explicitly stating that the job is in Astana. But I still get flooded with candidates from Moscow who, apparently, do not read anything at all and do not understand how they are then going to work remotely with blocks, sanctions, and the inability to receive a salary normally into a foreign-currency account.

All right, that is only the beginning.

Suppose a connection happens. The person more or less understands the working conditions and starts answering my questions. But this is where the next episode of the show begins. Almost everyone answers through AI. The replies are carbon copies. You can see the neural slop immediately. Perfectly smooth text, zero personality, zero specifics, but plenty of “I am highly motivated and ready to integrate effectively into your team.” Thank you, ChatGPT, I recognized you too.

I write to the candidate: “Listen, it is obvious you gave me an AI-generated answer.” And I immediately add that, in principle, I do not care. Use AI, for God’s sake. I use it myself. Something else matters: will you be able to work further and deliver results?

Next, we sign an NDA and an IP agreement, I give access to a private repository, and I send instructions on how to run the product in developer mode.

At this point there are usually three options. The candidate disappears. The candidate buries me in questions about errors, dependencies, Docker, Node, why it does not work, and where the “do everything” button is. Or a small percentage of heroes report: “I brought the system up. I wrote Hello to it, and it answered via the LLM.”

The last option already feels like a small holiday. You can open the children’s champagne and light a candle to Saint npm install.

Then I send one more instruction. It explains how to configure nginx, add the site certificate and private key, so the programmer can run the product’s iOS or Android app and route traffic through TLS not to the production server, but to their own local developer playground.

And this is where the great silence usually begins. The kind of silence where you can hear a lonely nginx crying somewhere far away.

Either silence, or every tenth person writes: “Well, I launched it in the emulator, it works.” That is already good. The day was not lived in vain.

But then the real fun begins.

I assign a ticket where my testers have clearly described the short bug title, reproduction steps, current behavior, and expected result. In other words, the task is quite specific. Take it, read it, reproduce it, fix it.

And now the freshly baked candidate studies how to make a PR for the first time in their life. Usually they somehow manage to create a branch, but creating a pull request from it turns out to be a serious trial for many. Almost everyone immediately opens a PR into `main`, even though the end of the instruction says to switch to the `release/0.3` branch. Apparently, that line is protected by a magical invisible font. Nobody notices it.

I have to explain separately how to name branches, where to open the PR, what a base branch is, and why `main` is not meant for experiments.

Even though I send a description of the process for working with branches and code in advance. But here comes the main plot twist again: nobody reads anything.

Fine. Suppose the person somehow manages to create a PR. Even into the correct branch, not `main`. You would think victory is near.

But no.

I open the PR, and it touches 30 project files. Thirty files. For a ticket where the task was to fix one button, one screen, or one check.

In other words, the candidate ran something, it “coded” for them, and as a result 90 percent of the changes have nothing to do with either the ticket or the bug. Sometimes it feels as if the AI simply walked through the project with a broom and decided to format this, rename that, improve the architecture here, add a strange abstraction there, and leave an import that nobody needs but that looks respectable.

I write: “Remove the unrelated changes from the PR. Leave only what belongs to the task.”

And a new episode begins. “How?”, “What is unrelated?”, “That is how it generated it for me,” “But it works.”

Sometimes the PR contains not a fix, but a crutch that merely hides the problem. The bug is not solved, but now it is invisible in one specific scenario. Until the next click. Or the next user. Or the next full moon.

So here is the question: how do you work with this? What kind of madness is this? How do you filter out such candidates? How do you find a normal programmer today — someone who reads the job description, can run the project, understands what branches and PRs are, does not change 30 files to fix one button, uses AI as a tool rather than as an autopilot with no brakes, and at least occasionally reads instructions?

Because for now it feels like the market has filled up with vibe coders. They confidently “feel” the code, but do not always understand what exactly they have just committed.

AI is an excellent tool. I use it myself and see no problem with that. The problem starts when a person stops thinking and turns into a gasket between the task and the Generate button.

And then that result lands in your PR.

With love, pain, and `git reset --hard`.
