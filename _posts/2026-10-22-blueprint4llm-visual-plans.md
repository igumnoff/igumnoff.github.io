---
layout: post
title: "I Got Tired of Reading Walls of Text from AI Agents. So I Made Them Draw Diagrams and UI Prototypes"
date: 2026-10-22 12:00:00 +0500
categories: [ai, programming, developer-experience]
image: /assets/img/posts/markdown-to-visual.png
---

Hi everyone! I'll be honest: as a programmer, I'm tired of reading the mountains of text that AI coding agents produce. It's painful. When I ask an agent to create a plan.md file that explains how to build a project from scratch or add a feature to an existing one, I get a wall of Markdown text. No UI prototypes. No diagrams that show how things work or what they look like. Reading it is a slog. I'd much rather look at UI prototypes and diagrams with arrows that show how everything works. As the saying goes, a picture is worth a thousand words.

Why does this happen? LLMs are built to understand text, so they happily produce text descriptions too. But a human still has to review what the AI produces, approve it, or request changes. And people tend to prefer visual explanations and simple presentation. Reading tons of text is exhausting. Building a mental picture of how something works or looks takes even more concentration and mental effort.

So I wanted to make LLMs a little more considerate of the poor programmer. The plan should be easy for a person to understand and include plenty of visuals. The AI could then use that same plan.md to do its work. Markdown is practically a native language for agents. It's also extremely popular with people because it's easy to read and write. For UI prototypes, there are tools like draw.io, Pencil, and Figma, but their JSON formats aren't designed to be written by hand from scratch. Reading them is manageable. Writing them is another story.

There's PlantUML Salt, of course, but it feels like UI prototyping was added to a project designed for UML. It wasn't quite what I needed. Eventually, I came across wireframes: rough sketches of what an application or website screen will look like. Balsamiq is probably the best-known tool for this, but it uses JSON too. Then I got lucky and found the open-source project WireMark. Its format is concise and easy for both people and LLMs to understand:

```wireframe
Wireframe #request w=420 h=300
  Stack column gap=2 padding=2
    Typography h4 "New request"
    TextField "Title"
    TextField "Description"
    Button "Submit request" contained
```

This renders into an SVG interface sketch that looks like this:

![Markdown to Visual](/assets/img/posts/markdown-to-visual.png)

Then came the question of what to use for UML-style diagrams and flowcharts. PlantUML has a text-based DSL, but it also depends on the Java virtual machine. Mermaid is more popular and widely supported, and LLMs readily generate diagrams and flowcharts in its format by default. GitHub has also supported Mermaid directly in Markdown since 2022. Why fight a de facto standard?

So I added this section to AGENTS.md (CLAUDE.md):

```
## Rules for Working with .md Files

- Use Wiremark for UI prototypes and Mermaid for diagrams that explain how the code works.
- When creating development plans (such as `plan.md`), focus on UI prototypes and diagrams rather than text descriptions, so people can quickly see how the project looks and works.
- When creating UI prototypes with Wireframe blocks, use a separate Wireframe block for each screen.
```

After that, AI agents like Codex, Claude Code, and Pi started creating plan.md files filled with clear diagrams, flowcharts, and UI sketches. But one problem remained: many viewers and IDEs don't support Wiremark and Mermaid embedded in Markdown. You usually see the DSL source instead of the rendered visuals. So I built an open-source project to solve that problem.

It has two modes. First, you can export from the terminal:

```
blueprint4llm plan.md --out=plan.html
blueprint4llm plan.md --out=plan.pdf --format=pdf
```

It takes plan.md as input and produces plan.pdf or plan.html with the diagrams, flowcharts, and UI sketches rendered in the document.

The second mode runs a local preview server:

```bash
blueprint4llm plan.md --serve
```

The terminal shows a URL. Open it in your browser to see the page with its diagrams and prototypes. When you save changes to `plan.md`, the document is rebuilt and the browser page refreshes automatically. You can also download a PDF of the current version directly from the page.

This mode is useful while you're still writing the plan and making frequent changes. You don't have to run the export again and open the output after every edit. Keep the preview next to your editor to see how new Wiremark and Mermaid blocks look as you work.

If you'd like to install the tool or take a look at its source code. Visit https://github.com/evgenyigumnov/blueprint4llm

