---
layout: post
title:  "rustsn — Open Source Project for Code Generation and Interaction with Existing Code Using LLM"
date:   2024-10-05 11:13:01 +0500
categories: rust, llm
---
I’ve been working on a tool called rustsn, which allows generating, compiling, and testing code using LLMs (Large Language Models). Initially, the idea was to automate the process of writing small pieces of code — so-called snippets — for different programming languages based on user-provided explanations. This tool has since evolved and gained new features, such as generating complete code for applications and explaining existing code based on vector representations (embeddings).

When I first started working on rustsn, the primary goal was to enable the user to, for instance, simply describe in words what function they needed, and the system would automatically generate working code. I started with Rust because this language has powerful capabilities for working with types and testing, making it ideal for writing safe and performant code. Later, I added support for other languages like JavaScript, Python, and TypeScript.

This tool has become much more than just a code generator. For example, I added an \`ask\` command that allows you to get explanations of the existing code in your project. A special model analyzes the source files and finds the most relevant ones based on your question.

In the near future, I plan to significantly expand rustsn's functionality. The first thing I want to do is integrate Docker so that users can run the tool not only on their host machine but also in containers. This will simplify deployment and usage on various platforms, ensuring a stable environment.

Moreover, one of the key goals is to expand the code generation capabilities. Currently, rustsn performs well in generating individual functions, but I would like it to be able to create complete seed projects — from command-line utilities to web services with REST APIs. This would make the tool much more useful for developers who want to quickly get a skeleton application based on a simple description.

Finally, to make working with rustsn even more convenient, I plan to add a graphical interface in the form of a web application. This will allow users to interact with the tool through a browser, without needing to use the terminal, which is particularly convenient for those who are not accustomed to working with the command line.

I see great potential in developing this project, and the next steps are aimed at making rustsn a universal tool for code generation and analysis, capable of helping developers at all stages of their work.

The main challenge I see in developing rustsn is that the use of commercial, frontier AI models, which require constant spending on APIs and resources, will lose out to open-source solutions. We’re already seeing a trend where open models, which can run on your own hardware, are becoming more powerful and user-friendly for developers. That’s why I’ve chosen to integrate rustsn with Ollama — a free and local solution that allows you to run LLMs on your own computer, providing maximum flexibility and independence.

I am confident that in 1-2 years, LLM optimization will make it possible to run them not on large GPU clusters, as required today, but on regular laptops with relatively modest hardware. This will be a real revolution for developers: everyone will be able to run powerful AI models locally, without depending on third-party services and their fees. My goal is to prepare rustsn for this new reality, making it more efficient, autonomous, and accessible to every developer, regardless of their budget or infrastructure.

Thus, rustsn will not only simplify the process of writing and analyzing code but will also become part of a future where every programmer can use local AI solutions for building applications, analyzing existing projects, and automating routine tasks.

If you want to participate in the LLM project and at the same time learn the Rust programming language, welcome to my project: [https://github.com/evgenyigumnov/rustsn](https://github.com/evgenyigumnov/rustsn)