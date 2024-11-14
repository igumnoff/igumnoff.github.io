---
title: Projects
permalink: /
layout: page
---

### CBLT
![cblt](https://github.com/evgenyigumnov/cblt/raw/HEAD/logo.png)

Safe and fast minimalistic web server, written in Rust, that serves files from a directory and proxies requests to another server.

Features: 
- **10 times faster than Nginx for small content under 100KB**
- KDL Document Language configuration (**Cbltfile**)
- Proxy requests to another server
- Serve files from a directory
- TLS support
- Gzip compression
- Redirects

Repo: [https://github.com/evgenyigumnov/cblt](https://github.com/evgenyigumnov/cblt)


### Shiva
![Shiva](/shiva.png)
- Shiva Library: Implementation in Rust of a parser and generator for documents of any type
- CLI Shiva: Сonverting documents from any format to any

Features:

- [x] Common Document Model (CDM) for all document types
- [x] Parsers produce CDM
- [x] Generators consume CDM

| Document type | Parse | Generate |
|---------------|-------|----------|
| Plain text    | +     | +        |
| Markdown      | +     | +        |
| HTML          | +     | +        |
| PDF           | +     | +        |
| JSON          | +     | +        |
| XML           | +     | +        |
| CSV           | +     | +        |
| RTF           | +     | +        |
| DOCX          | +     | +        |
| XLS           | +     | -        |
| XLSX          | +     | +        |
| ODS           | +     | +        |
| Typst         | -     | +        |

Repo: [https://github.com/igumnoff/shiva](https://github.com/igumnoff/shiva)


### rustsn 

![rustsn](https://github.com/evgenyigumnov/rustsn/raw/HEAD/logo.png)

This Rust-based tool generates, compiles, and tests code using LLMs, resolves dependencies, and provides explanations of existing code through embeddings.

Features

1. **generate function** command is used to generate code snippets based on user-provided explanations.
2. TODO: **generate application** command is used to generate seed project code based on user-provided explanations.
3. **ask** command is used to get explanation by existing codes of your project based on user-provided question.

Supported languages by feature

| language   | generate function | generate application | ask |
|------------|-------------------|----------------------|-----|
| Rust       | +                 | -                    | +   |
| JavaScript | +                 | -                    | +   |
| C#         | -                 | -                    | +   |
| Python     | +                 | -                    | -   |
| TypeScript | +                 | -                    | -   |
| Java       | +                 | -                    | -   |
| Kotlin     | +                 | -                    | -   |
| Swift      | +                 | -                    | -   |
| PHP        | +                 | -                    | -   |
| Scala      | +                 | -                    | -   |

Repo: [https://github.com/evgenyigumnov/rustsn](https://github.com/evgenyigumnov/rustsn)


### Gabriel2
![Gabriel2](/gabriel2.png)

Gabriel2: Indeed, an actor library, not a framework, written in Rust

Features:

- [x] Async for sending messages
- [x] Async for messages processing in actor
- [x] Support messaging like send and forget 
- [x] Support messaging like send and wait response
- [x] Mutable state of actor
- [x] Self reference in actor from context
- [x] Actor lifecycle (pre_start, pre_stop)

Repo: [https://github.com/igumnoff/gabriel2](https://github.com/igumnoff/gabriel2)

### Parvati
![Parvati](/parvati.png)

Parvati library: ORM library with no boilerplate code, written in Rust.

- [x] SQLite support
- [x] MySQL support

Repo: [https://github.com/igumnoff/parvati](https://github.com/igumnoff/parvati)

### Metatron
![metatron](/metatron.png)

Metatron library: Implementation in Rust of a report generation

Supported report types:

- Plain text
- Markdown
- HTML
- PDF

Repo: [https://github.com/igumnoff/metatron](https://github.com/igumnoff/metatron)
