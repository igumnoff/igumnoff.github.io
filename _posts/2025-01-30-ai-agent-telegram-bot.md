---
layout: post
title:  "How to Create an AI Telegram Bot with Vector Memory on Qdrant"
date:   2025-01-30 11:01:07 +0500
categories: rust, ai, agent, telegram, embedding, qdrant
---

The idea for this pet project came from my desire to build my own AI agent. I established minimal technical requirements for myself: the agent should have multiple states, be able to launch tools, and use RAG (Retrieval Augmented Generation) to search for answers.

Ultimately, I decided to create a personal Telegram AI bot that can remember the information I need, and whenever I want, I can ask it what it has retained. It’s like a notebook, only this is an AI-powered notebook that can answer questions. Additionally, I wanted it to be able to execute commands on a server—commands described in human language that it would translate into terminal commands.

Initially, I considered using LangChain. It’s a great tool—it supports connecting vector databases, using various LLMs for both inference and embedding, and defining the agent’s logic through a state graph. Ready-made tools can be called as well. At first glance, everything seems convenient and simple, especially when you look at typical and straightforward examples.

However, after digging a bit deeper, I found that the effort required to learn this framework wasn’t justified. It’s simpler to directly call LLMs, embeddings, and Qdrant via REST API. Plus, you can describe the agent’s logic in code using an `enum` to represent states and performing a `match` on these states.

Moreover, LangChain was originally written in Python. I prefer coding in Rust, and using a Rust version of LangChain turns out to be a dubious pleasure—usually running into issues at the most inconvenient moments when some component hasn’t yet been rewritten in Rust.

For implementing the RAG magic, I decided to use the following algorithm: When the user asks a question, key words are extracted from the query using an LLM. Then, an embedding is used to compute a vector from these key words. This vector is sent to Qdrant to search for the nearest vectors from the documents already stored. After that, a query is formed for the LLM using the found documents along with the user’s question. The result is an LLM-generated answer that takes into account the data that is semantically close to the question. Accordingly, when the user provides information to the bot, it is saved in Qdrant with an associated vector computed via embedding. In other words, vectors with similar meanings have minimal distances between each other. This is how the search for semantically similar documents works.

## Design

First, I devised the overall logic for the AI bot’s operation. The bot responds to user commands by:

- Checking the password before starting.
- Understanding what the user wants (a question, a statement, a request to forget, a terminal command, etc.).
- Working with the Qdrant vector database—it can remember and forget information.
- Comprehending commands in a human-like manner and executing them on the server.
- Accomplishing all of this using a local LLM (via HTTP API requests).

Then, I detailed the scenario for the AI bot's operation:

### 1. The User Sends a Message in Telegram

The user sends anything to the bot—a question, a fact, a request, a command—anything at all.

The bot receives the message via the Telegram Bot API.

### 2. Password Verification

First, the bot waits for the user to enter the password. It compares the entered text with the environment variable `BOT_PASSWORD`.

- If the password is **correct**, the bot transitions to the `Pending` state (ready to operate).
- If the password is **incorrect**, it asks for the password again.

### 3. Message Processing

When the bot is in the `Pending` state, it analyzes the message. To understand exactly what the user sent, an LLM is invoked:

The LLM receives the text and returns a number corresponding to:
1. Question
2. Fact / statement
3. Request to forget
4. Terminal command
5. Anything else


### 4. Actions Based on the Message Type

#### Type 1: Question

The bot asks the LLM to extract keywords from the query to understand **what it is about**.

Using these keywords, the bot searches for the most relevant documents in the Qdrant vector database.

Then it **merges** the retrieved information with the original question and once again consults the LLM to get a **final answer**.

The answer is then sent to the user.

#### Type 2: Statement (Save Information)

The bot creates an embedding from the text and adds it to Qdrant.

The user receives a confirmation: "Information saved".

#### Type 3: Request to Forget

The bot searches for **what exactly needs to be forgotten** using keywords.

It then asks the user to confirm whether it should indeed forget it.

- If **yes** → it deletes the document from Qdrant.
- If **no** → it leaves it as is.

#### Type 4: Terminal Command

The bot asks the LLM to **formulate a command** for Linux based on a description.

It then asks the user to confirm whether to execute the command:

- If **yes** → it executes the command using `std::process::Command` and sends the result.
- If **no** → the command is not executed.

#### Type 5: Everything Else

If the bot does not understand what is being asked, it simply responds politely and in a friendly manner using the LLM, just like a regular chat bot.


## Code Implementation

I started writing code for working with LLM and embeddings. Below is a list of functions from [ai.rs](https://github.com/evgenyigumnov/ai-agent-telegram-bot/blob/main/src/ai.rs) with brief and clear descriptions:

### `llm(system: &str, user: &str) -> anyhow::Result<String>`
**What it does:**
Sends a request to a chat LLM (via an OpenAI-compatible API).

**Input:**
- `system` — the system message (e.g., instructions for the bot).
- `user` — the user's message.

**Output:**
- Returns the model's response as a string.



### `emb(input: &str) -> anyhow::Result<Vec<f32>>`
**What it does:**
Creates an embedding for the given text using an embedding model.

**Input:**
- `input` — the text string that needs to be encoded.

**Output:**
- A vector of embedding values `Vec<f32>`.



Next, I implemented the functionality for working with Qdrant. Below is a list of functions from [qdrant.rs](https://github.com/evgenyigumnov/ai-agent-telegram-bot/blob/main/src/qdrant.rs):

### `add_document(id: i32, text: &str)`
**Adds a document to Qdrant.**
1. Generates an embedding for `text` using `emb()`.
2. Forms a `Point` and sends a `PUT` request to Qdrant.
   Used for the bot to remember information.



### `delete_document(id: i32)`
**Deletes a document by ID from the Qdrant collection.**
Sends a `POST` request to `points/delete`.



### `create_collection()`
**Creates a collection in Qdrant.**
1. Reads the embedding dimensionality from the `.env` file.
2. Sets the comparison metric to Cosine.
   Useful for the bot's initial setup.



### `delete_collection()`
**Deletes the entire collection from Qdrant.**
Useful when switching the embedding model (different dimensionality).



### `exists_collection() -> bool`
**Checks if the collection exists in Qdrant.**
Sends a `GET` request and returns `true` if it exists.



### `last_document_id() -> i32`
**Finds the maximum ID among all documents.**
Needed to correctly increment the ID when adding new ones.



### `all_documents() -> Vec<Document>`
**Retrieves all documents from the collection.**
Scrolls through the collection page by page using the Qdrant `scroll` request.



### `search_one(query: &str) -> Document`
**Searches for a single (most relevant) document.**
Used for confirming the deletion of specific information.



### `search_smart(query: &str) -> Vec<Document>`
**Intelligent search for relevant documents.**
1. Performs a standard `search()`.
2. Filters results by `distance > 0.6`.
3. If none match, it takes the first one.
   Used when generating responses.



### `search(query: &str, limit: usize) -> Vec<Document>`
**Basic search for documents by vector similarity.**
1. Generates a query vector.
2. Sends a `points/search` request to Qdrant.
3. Returns the sorted documents along with their `distance`.




Then, using the building blocks from `ai.rs` and `qdrant.rs`, I wrote the bot’s logic in [main.rs](https://github.com/evgenyigumnov/ai-agent-telegram-bot/blob/main/src/main.rs):

### `main`

The main asynchronous entry point:

1. Loads the `.env` variables.
2. Initializes a collection in Qdrant and prints the documents from memory.
3. Creates the Telegram bot.
4. Starts processing messages (`teloxide::repl`), handing control over to the Finite State Machine.

### `enum State`

```rust
enum State {
    AwaitingPassword,
    Pending,
    ConfirmForget { info: String },
    ConfirmCommand { message: String, command: String },
}
```

The user's Finite State Machine:

1. `AwaitingPassword`: waits for the password input.
2. `Pending`: main mode – the user is authorized.
3. `ConfirmForget`: confirmation for information deletion.
4. `ConfirmCommand`: confirmation of command execution.

### `State::process`

The main entry point that calls the handler for the current state:

```rust
pub fn process(input: &str, state: &State) -> anyhow::Result<(Self, String)>
```

It calls the corresponding function (essentially a `match` on the state).

### `process_password`

Verifies the password entered by the user:

```rust
pub fn process_password(input: &str) -> anyhow::Result<(Self, String)>
```

- If the password matches the `BOT_PASSWORD` from `.env`, it transitions to `Pending`.
- Otherwise, it remains in `AwaitingPassword`.

### `exec_pending`

The most important part: determines the type of the user's message (question, info, command, etc.):

```rust
pub fn exec_pending(message: &str) -> anyhow::Result<(Self, String)>
```

- It passes the phrase to the LLM and receives an answer: "1", "2", ..., "5".
- Depending on the digit, it calls the required function:
    - `1` → `exec_answer`
    - `2` → `exec_remember`
    - `3` → `new_forget`
    - `4` → `new_command`
    - otherwise → `exec_chat`

### `exec_answer`

RAG approach: extracts relevant documents and generates an answer:

```rust
pub fn exec_answer(message: &str) -> anyhow::Result<(Self, String)>
```

- Extracts keywords from the message.
- Searches for documents in Qdrant.
- Feeds all this to the LLM and receives an answer.
- Returns `Pending`.

### `exec_remember`

Simply adds new information to Qdrant with an auto-increment ID:

```rust
pub fn exec_remember(message: &str) -> anyhow::Result<(Self, String)>
```

### `exec_chat`

A simple conversation with the LLM without RAG:

```rust
pub fn exec_chat(message: &str) -> anyhow::Result<(Self, String)>
```

### `new_forget` → `exec_forget`

Deletion of information from memory:

- `new_forget`: finds the appropriate document based on keywords and asks for confirmation.
- `exec_forget`: if the user confirms, deletes the document.

### `new_command` → `exec_confirm_command`

Command execution:

- `new_command`: transforms the user's request into a Linux command and asks for confirmation.
- `exec_confirm_command`: if confirmed (i.e., "yes"), executes the command and returns stdout/stderr.

### `is_condition`

A universal way to determine: "Is the user in agreement?" or "Does this message contain a command?".

### `extract_tag`, `extract_number`

Helper functions:
- `extract_tag`: extracts the content from a `<tag>` in the LLM response.
- `extract_number`: finds the first digit in the LLM response.

### `init_qdrant` and `print_docs`

- `init_qdrant`: creates a collection if it does not exist.
- `print_docs`: prints all memory on startup (convenient for debugging).


## Docker & Startup

The startup uses [docker-compose.yml](https://github.com/evgenyigumnov/ai-agent-telegram-bot/blob/main/docker-compose.yml). It contains two services:
- `backend` — the bot itself.
- `qdrant` — the vector database.

The [Dockerfile](https://github.com/evgenyigumnov/ai-agent-telegram-bot/blob/main/Dockerfile) builds the Rust project and runs the binary inside a minimal `debian:bullseye`.

Startup instructions:
```bash
# Copy .env-example and specify your variables in it
cp .env-example .env

# Build and run the containers
docker compose up --build
```

## Example Chat with the Bot

```
Evgeny Igumnov:
Hello!

Igumnov's memory extention:
Incorrect password. Try again.

Evgeny Igumnov:
supersecret

Igumnov's memory extention:
Password accepted. You can continue working with the bot.

Evgeny Igumnov:
Katya's birthday is November 24.

Igumnov's memory extention:
Information saved.

Evgeny Igumnov:
What is my wifi password?

Igumnov's memory extention:
Your Wi-Fi password is 4040404040.

Evgeny Igumnov:
Run a command that displays a list of files in the current folder

Igumnov's memory extention:
Run the "ls" command?

Evgeny Igumnov:
yes

Igumnov's memory extention:
Result of the command

all-files.sh
Cargo.lock
Cargo.toml
docker-compose.yml
Dockerfile
LICENSE
README.md
src

Evgeny Igumnov:
When is Katya's birthday?

Igumnov's memory extention:
Katya's birthday is November 24th.

Evgeny Igumnov:
Forget when Katya's birthday is

Igumnov's memory extention:
'Katya's birthday is November 24.' Forget the information?

Evgeny Igumnov:
yes

Igumnov's memory extention:
Information forgotten.
```

## In the End

I obtained the code for a full-fledged AI agent:
- It can **understand and analyze** text.
- It has **states** and can switch between them.
- It works with both **memory** and **the terminal**.
- Everything is written in Rust: fast, stable, and predictable.

The source code of the AI Telegram bot is available here: [https://github.com/evgenyigumnov/ai-agent-telegram-bot](https://github.com/evgenyigumnov/ai-agent-telegram-bot)