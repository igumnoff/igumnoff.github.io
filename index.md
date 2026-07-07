---
layout: default
title: Work
permalink: /
---

<section class="hero">
  <div class="hero-copy">
    <p class="eyebrow">Founder / CTO / Principal Software Developer</p>
    <h1>Founder/CTO who still builds the system.</h1>
    <p class="lead">
      I build Rust, AI assistant, document engine, backend, and high-load platforms
      from architecture to launch.
    </p>
    <div class="hero-actions">
      <a class="button button-primary" href="#work">View technical work</a>
      <a class="button button-secondary" href="{{ "/blog" | relative_url }}">Read engineering notes</a>
    </div>
    <div class="proof-row" aria-label="Career highlights">
      <span>MyLivePage: 10M users</span>
      <span>20M page views/day</span>
      <span>Relayna: AI assistant platform</span>
    </div>
  </div>

  <div class="hero-portrait">
    <img src="{{ "/assets/img/evgeny-business.png" | relative_url }}" alt="Evgeny Igumnov">
    <div class="portrait-card">
      <strong>Evgeny Igumnov</strong>
      <span>Rust, AI tooling, document engines, production infrastructure</span>
    </div>
  </div>
</section>

<section class="role-grid" aria-label="Positioning">
  <article>
    <h2>Founder</h2>
    <p>Built and scaled products, teams, and operations, including MyLivePage reaching millions of users.</p>
  </article>
  <article>
    <h2>CTO</h2>
    <p>Owns architecture, delivery, infrastructure, deployment, and the technical shape of the product.</p>
  </article>
  <article>
    <h2>Principal Developer</h2>
    <p>Still writes production systems in Rust, React, Electron, Axum, SQLX, Docker, and related tooling.</p>
  </article>
</section>

<section id="work" class="section">
  <div class="section-heading">
    <p class="eyebrow">Selected work</p>
    <h2>Technical systems with product intent.</h2>
  </div>

  <div class="project-grid">
    <article class="project-card featured">
      <p class="project-kicker">Company / AI platform</p>
      <h3>Relayna</h3>
      <p>AI assistant platform across desktop, web, cloud, and mobile with local-first runtime, tool execution, provider integrations, tracing, and production infrastructure.</p>
      <a href="https://relayna.com">relayna.com</a>
    </article>
    <article class="project-card">
      <p class="project-kicker">Founder / high-load</p>
      <h3>MyLivePage</h3>
      <p>Website builder scaled to 10 million users and more than 20 million page views per day.</p>
    </article>
    <article class="project-card">
      <p class="project-kicker">Rust / documents</p>
      <h3>Shiva</h3>
      <p>Open source Rust library for parsing and generating documents through a common document model.</p>
      <a href="{{ "/rust,/parser,/pdf/2024/04/06/shiva-rust.html" | relative_url }}">Read note</a>
    </article>
    <article class="project-card">
      <p class="project-kicker">Rust / web server</p>
      <h3>CBLT</h3>
      <p>Minimal Rust web server for static files, reverse proxying, and TLS-oriented deployment experiments.</p>
      <a href="{{ "/rust,/web,/server,/proxy/2024/11/14/CBLT-Web-Server.html" | relative_url }}">Read note</a>
    </article>
    <article class="project-card">
      <p class="project-kicker">AI / local LLM</p>
      <h3>rustsn</h3>
      <p>LLM-assisted code generation and existing-code interaction with local model direction.</p>
      <a href="{{ "/rust,/llm/2024/10/05/rustsn-llm.html" | relative_url }}">Read note</a>
    </article>
    <article class="project-card">
      <p class="project-kicker">AI / agent memory</p>
      <h3>Telegram AI Agent</h3>
      <p>Personal AI bot with Qdrant vector memory, state-based behavior, and command execution flow.</p>
      <a href="{{ "/rust,/ai,/agent,/telegram,/embedding,/qdrant/2025/01/30/ai-agent-telegram-bot.html" | relative_url }}">Read note</a>
    </article>
  </div>
</section>

<section class="section writing-preview">
  <div class="section-heading">
    <p class="eyebrow">Writing</p>
    <h2>Engineering notes from the workbench.</h2>
  </div>

  <div class="writing-list">
    {% for post in site.posts limit:3 %}
      <a class="writing-item" href="{{ post.url | relative_url }}">
        <span>{{ post.date | date: "%b %-d, %Y" }}</span>
        <strong>{{ post.title | escape }}</strong>
      </a>
    {% endfor %}
  </div>
</section>
