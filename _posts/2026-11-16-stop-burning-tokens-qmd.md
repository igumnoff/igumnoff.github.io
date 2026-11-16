---
layout: post
title: "Stop Burning Tokens on File Searches: How QMD Helps Coding Agents Save Up to 55%"
date: 2026-11-16 12:00:00 +0500
categories: [ai, programming, developer-tools]
---

I noticed that, before carrying out a task, coding agents run `find`, `rg --files`, or `glob` to locate files and the classes and modules they need. Then they search file contents with `rg` or `grep`. This burns a lot of tokens. So I decided to see whether semantic search could save tokens compared with all of the above. If a local GPU is sitting idle, why not use it for semantic search?

I work at a company focused on security. In one of my projects, I use semantic search for sensitive documents. Then an acquaintance sent me a link to an open-source project that tackles the same problem.
The project is called "QMD - Query Markup Documents." It runs from the command line, extracts text from documents in all formats, and creates an index. It supports several search modes:

```
# Search across everything
qmd search "project timeline"           # Fast keyword search
qmd vsearch "how to deploy"             # Semantic search
qmd query "quarterly planning process"  # Hybrid + reranking (best quality)
```

We're interested in the last option. It requires three things to work:

- embeddinggemma-300M-Q8_0	Vector embeddings (default)	~300MB
- qwen3-reranker-0.6b-q8_0	Re-ranking	~640MB
- qmd-query-expansion-1.7B-q4_k_m	Query expansion (fine-tuned)	~1.1GB

The first model is used for indexing to compute semantic vectors for text chunks. The second compares the search results with the query to determine how semantically similar they are. The third generates several search queries from the original query to broaden the search.

First, add your source files to QMD (this is an example for a minimal project with a Rust server and an Angular frontend):

```
qmd collection add . \
  --name PROJECT1 \
  --mask "src/**/*.rs,\
frontend/src/**/*.{ts,html,scss},\
README.md,\
frontend/README.md,\
Cargo.toml,\
frontend/package.json,\
frontend/angular.json,\
frontend/tsconfig*.json,\
frontend/proxy.conf.json,\
.gitignore,\
frontend/.gitignore,\
frontend/.prettierrc,\
frontend/.editorconfig"
```
You don't have to do this manually: show this example to your coding agent and tell it to do the same for the current repo. It will figure out which folders and file extensions to add to the index, what to exclude, and so on.

Now let's run the indexing (you'll need to rerun it periodically as your codebase changes):
```
qmd update && qmd embed -c PROJECT1
```

To make the coding agent work with this QMD setup, add the following to `AGENTS.md`:

```
# QMD - Efficient Code Discovery

**Goal:** minimize token costs and code search time without compromising correctness.

## Choosing a tool

- If the path is known → read the file directly.
- If an identifier, error, or line is known → use `rg` / LSP.
- If only the behavior is known → use `qmd query`.
- Once the code is found → inspect the necessary sections with standard tools.
- Once you have enough information → do the task.

**QMD replaces the initial broad search; it does not add another step.**

## Semantic search

Use the project's existing collection.

"""
qmd query "DESCRIPTION" -c PROJECT1 -n 15 --files \
  | grep -E '\.(EXT1|EXT2|EXT3)$' \
  | cut -d, -f3- \
  | sed 's|^qmd://[^/]*/||' \
  | head -n 10
"""

- Use one specific query, preferably with English technical terms.
- Choose extensions that match the project, including configuration files when needed.
- Start by reading only the relevant sections of the 3–5 most relevant files.
- If you cannot find the implementation, refine the query; if necessary, increase `-n` to 60, then use `rg` / LSP.
- Stop semantic searching once you find the implementation.

## Saving tokens

- Do not scan the whole repository before using QMD.
- Do not repeat searches without a reason or open every result.
- Do not investigate unrelated code or dependencies.
- Do not write detailed search plans or reports.
- Do not paraphrase code you have read.
- As soon as you have enough information, make the changes and check the result.

**Priority: correctness → token cost → time.**
```

I tested it on a repository like this, with 1,231 source files across Java, Scala, and Angular:
```
C:\Users\igumn\temp2>qmd collection list
jcm-server (qmd://jcm-server/)
  Pattern:  server/src/**/*.scala,server/src/**/*.java,server/src/**/*.xml,server/src/**/*.properties,server/src/**/*.js,server/src/**/*.ts,server/src/**/*.css,server/src/**/*.html,server/src/**/*.tmpl,server/src/**/*.txt,server/src/**/*.md,server/pom.xml,server/Dockerfile,server/*.md,server/README.txt,web/src/**/*.ts,web/src/**/*.html,web/src/**/*.scss,web/src/**/*.css,web/package.json,web/angular.json,web/tsconfig*.json,web/karma.conf.js,web/README.md,!server/src/main/resources/templates/**,!server/src/main/resources/messages*,!server/src/main/resources/messages*/**,!web/src/assets/lang/**
  Files:    1231
  Updated:  1h ago
```

Prompt: "Add a post-quantum encryption flag to the disk encryption policy. It should be disabled by default, configurable in the web interface when editing the policy, saved to the database, and sent to the agent as part of the encryption policy via the REST endpoint."

## **Cost Comparison**

Using GPT-6 Luna API rates: input $0.10, cached input $0.01, and output $0.50 per 1 million tokens.

| Metric        | With QMD   | Without QMD | Savings  |
| ------------- | ---------- | ----------- | -------- |
| Input         | 65,006     | 101,501     | 35.96%   |
| Cached input  | 317,568    | 1,191,808   | 73.35%   |
| Output        | 13,859     | 16,348      | 15.23%   |
| Reasoning     | 9,607      | 11,260      | 14.68%   |
| **Cost**      | **$0.0166** | **$0.0302** | **45.09%** |

Now **QMD saves 45.09% in costs!**

The reduction in cached input is especially noticeable: nearly 4x.

I ran these tests with GPT-6 Luna. In the first test, the agent's runtime dropped from 6 minutes to 4 minutes.

Prompt: "Add a `dry` field to the erasure policy. It should be editable in the web interface and saved to the database, and it should also be sent through the `/rest` endpoint to the agent responsible for erasure. Its default value should be `false`."

Calculation using the same GPT-6 Luna rates: input $0.10, cached input $0.01, and output $0.50 per million tokens.

## **Comparison**

| Metric         | QMD         | Without QMD | Difference |
| ------------- | ----------- | ----------- | ------- |
| Input         | 77,167      | 98,042      | −21.29% |
| Cached input  | 936,832     | 854,912     | +9.58%  |
| Output        | 17,322      | 16,259      | +6.54%  |
| Reasoning     | 8,835       | 10,264      | −13.92% |
| **Cost**      | **$0.0257** | **$0.0265** | −2.78%  |

Here, **QMD turned out to be 2.78% cheaper**. There are savings, but they are small.

**Conclusion:** QMD reduced regular input tokens by 21.3% and reasoning by 13.9%, but the increase in cached input and output almost completely wiped out the savings.

Overall, the savings range from 2.78% to 45.09% of the cost. I think that's a decent result. Of course, we need a larger sample. But I'm ready for bed. Let me know in the comments if you've run a larger sample and what results you got.

PS

I recommend checking that junk isn't getting into your index. Try running a semantic query yourself. I once overlooked compiled .js files and large lang files-they heavily polluted the semantic search results. Here's an example of how to run a search manually:

```
C:\Users\igumn\jcm-server>qmd query "disk encryption policy editing form persisted setting sent to endpoint for agent" -c jcm-server -n 15 --files
Expanding query... (3.0s)
├─ disk encryption policy editing form persisted setting sent to endpoint for agent
├─ lex: disk encryption configuration
├─ lex: disk encryption policy
├─ vec: disk encryption configuration
├─ vec: disk encryption policy
└─ hyde: The topic of disk encryption policy editing form persisted setting se...
Searching 6 queries...
Embedding 4 queries... (1.5s)
Reranking 40 chunks... (3.3s)
#d70022,1.00,qmd://jcm-server/server/src/main/java/com/jetico/jcm/server/domain/common/bcvolume/EncryptPolicy.scala
#732b09,0.63,qmd://jcm-server/server/src/main/java/com/jetico/jcm/server/service/ajax/common/EncryptSettingsManager.scala
#e66494,0.63,qmd://jcm-server/web/src/app/modules/management/components/company/dialogs/add-or-update-policies/add-or-update-policies.component.html
#413c5e,0.62,qmd://jcm-server/web/src/app/modules/management/components/group/dialogs/change-policie/change-policies.component.html
#413c5e,0.62,qmd://jcm-server/web/src/app/modules/management/components/computer/dialogs/change-policie/change-policies.component.html
#758b4d,0.62,qmd://jcm-server/server/src/main/java/com/jetico/jcm/server/service/mcp/McpFacade.scala
#4e8973,0.62,qmd://jcm-server/web/src/app/modules/management/components/group/dialogs/change-policie/change-policies.component.ts
#bef78e,0.62,qmd://jcm-server/web/src/app/modules/management/components/computer/dialogs/change-policie/change-policies.component.ts
#4fcd1c,0.61,qmd://jcm-server/server/src/main/java/com/jetico/jcm/server/service/mcp/McpToolbox.scala
```

PPS

I tried it on a small project with 27 files. Instead, the cost went up by 60%. The solution turned out to be overkill. In general, use it only on large codebases.

## **UPDATE**

I went ahead and built a hybrid QMD + GitNexus solution after all. GitNexus performs structural code analysis (AST, dependency graph) and lets you search based on it. Here are the results:

### **Cost comparison**

| Metric                       | Without QMD   | QMD          | QMD + GitNexus |
| ---------------------------- | ------------ | ------------ | -------------- |
| Input                        | 101,501      | 65,006       | 59,122         |
| Cached input                 | 1,191,808    | 317,568      | 225,792        |
| Output                       | 16,348       | 13,859       | 10,894         |
| Reasoning (included in Output) | 11,260       | 9,607        | 8,465          |
| **Cost**                     | **$0.03024** | **$0.01661** | **$0.01362**   |
| **Savings**                  | -            | **45.1%**    | **55.0%**      |

The savings figure rose from 45% to **55%!** The coding agent's runtime increased from 6 minutes to 7 minutes. Here are the revised instructions for AGENTS.md:

```
# QMD + GitNexus - Efficient Code Discovery

**Goal:** find the required code and its relationships with minimal token usage, without compromising correctness.

## Choosing a tool

- If the file path is known → read it directly.
- If the exact identifier, error, or line is known → use `rg` / LSP.
- If only the behavior is known and the implementation location is unknown → use QMD semantic search.
- If you need to understand symbol relationships, the execution path, or the impact of a change → use GitNexus structural analysis.
- Once the code is found → inspect the necessary sections with standard tools.

QMD and GitNexus replace the initial broad search; they do not add mandatory steps. Use only the tools needed for the task.

## QMD semantic search

Use the project's existing collection:

    qmd query "DESCRIPTION" -c PROJECT1 -n 15 --files \
      | grep -E '\.(EXT1|EXT2|EXT3)$' \
      | cut -d, -f3- \
      | sed 's|^qmd://[^/]*/||' \
      | head -n 10

- Formulate one specific query, preferably using English technical terms.
- Specify suitable source and configuration file extensions.
- First read the 3–5 most relevant files, and only the relevant sections.
- If the implementation is not found, refine the query; if necessary, increase `-n` to 60, then use `rg` / LSP.
- Stop semantic search once the implementation is found.

## GitNexus structural analysis

Use GitNexus through the CLI when you need to investigate code structure:

- `context` - relationships and context for a known symbol.
- `impact` - possible impact of changing a symbol.
- `trace` - path between known symbols.

Examples:

    gitnexus context SymbolName
    gitnexus impact SymbolName
    gitnexus trace SourceSymbol TargetSymbol

If a symbol is unknown, find it first through QMD, `rg`, or LSP. Use GitNexus only when its structural analysis will help answer the question; do not run every command in sequence.

## Token efficiency

- Do not scan the entire repository before using QMD.
- Do not repeat searches without a reason or open every result.
- Do not investigate unrelated code or dependencies.
- Do not write detailed plans and reports about the search.
- Do not summarize code you have read.
- Once you have enough information, make the changes and verify the result.

**Priority: correctness → token cost → time.**
```

Create a `.gitnexusignore` file to keep junk out of the index so that it contains only source files. Then start GitNexus indexing with this command:

```
gitnexus analyze --index-only
```
