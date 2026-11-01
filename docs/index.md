---
title: "Free Observability Course: Prometheus, PromQL, Grafana, Alerting, OpenTelemetry & SLOs"
description: "Free observability course: 40 lessons on Prometheus, PromQL, Grafana dashboards, alerting, OpenTelemetry, and SLOs."
---

# Observability Mastery Path

<div class="mp-hero-lockup">
  <div class="mp-hero-globe">
    <svg viewBox="0 0 160 160" class="mp-globe-svg mp-mark">
      <circle cx="80" cy="80" r="74" fill="var(--md-default-bg-color, #FAF7EF)" stroke="#2FA98D" stroke-width="5"/>
      <rect class="mp-bar mp-bar-1" x="36" y="90" width="17" height="24" rx="5" fill="#2FA98D"/>
      <rect class="mp-bar mp-bar-2" x="59" y="74" width="17" height="40" rx="5" fill="#7FC9B7"/>
      <rect class="mp-bar mp-bar-3" x="82" y="56" width="17" height="58" rx="5" fill="#E0B457"/>
      <rect class="mp-bar mp-bar-4" x="105" y="38" width="17" height="76" rx="5" fill="#C97B4A"/>
      <circle class="mp-dot" cx="44.5" cy="82" r="7" fill="#22261f"/>
    </svg>
  </div>
  <div class="mp-hero-divider"></div>
  <div class="mp-hero-text">
    <div class="mp-hero-title">Mastery Path</div>
    <div class="mp-hero-sub">BOOTCAMP</div>
    <div class="mp-hero-rule"></div>
    <div class="mp-hero-byline">VINOD BHANUJA</div>
  </div>
</div>

Most "observability" tutorials show you a pretty Grafana dashboard and call it done, without ever explaining the one
idea that makes the whole stack make sense: a metric, a log line, and a trace span are three different, lossy
projections of the same underlying event, and the real skill is knowing which projection answers which question —
and how to connect them back together when one alone isn't enough. This course builds that skill by running a real
Prometheus server, a real Grafana instance, and real OpenTelemetry-instrumented code on an ordinary laptop, and
deliberately breaking things to watch each tool actually earn its place.

This course was written and run against a **real local Prometheus 3.x server, a real local Grafana instance, and
real Python services instrumented with the OpenTelemetry SDK** — no cloud observability vendor account, no SaaS
trial, no Docker Compose stack of pre-built dashboards. Real PromQL queries against real scraped metrics, real
alert rules that really fire, and a real trace reconstructed from real exported spans.

It assumes basic programming and command-line comfort, and benefits from — but doesn't require — the service-level
instrumentation built in the [Microservices Mastery Path](https://sigilipelli.github.io/microservices-mastery-path/),
whose own tracing and structured-logging lessons this course builds on directly and links back to rather than
repeating.

## How the program is organized

| Level | Focus | Modules | What you can do after |
|-------|-------|---------|------------------------|
| **1 · Metrics & Prometheus Fundamentals** | The Prometheus data model, instrumenting real metrics, PromQL, recording rules, the TSDB | 10 | Instrument a real app and query its metrics correctly |
| **2 · Dashboards, Alerting & Logs** | Grafana dashboards, the RED/golden-signals/USE methods, Alertmanager, centralized logging | 10 | Build dashboards and alerts that actually page the right person |
| **3 · Distributed Tracing & OpenTelemetry** | Spans, context propagation, the Collector, sampling, cardinality, correlating all three pillars | 10 | Trace a real request end to end and correlate it with metrics and logs |
| **4 · SLOs & Operating Observability at Scale** | SLIs/SLOs/error budgets, burn-rate alerting, capacity planning, scaling Prometheus, on-call | 10 | Design and operate an SLO-driven observability stack for a real service |

Each level has 9 teaching modules plus a hands-on project (module 10), for **40 lessons** total.

## How to use this site

- **Run it yourself.** Download Prometheus and Grafana directly from their own sites, or install both with one
  `brew install prometheus grafana` command — no account, no cloud bill, no managed observability platform needed
  for the core lessons.
- **Read the "what was run" note** at the top of each lesson. The overwhelming majority of this course's command
  output, PromQL results, and dashboard screenshots come from a real, observed, local stack; anything that
  genuinely couldn't be run locally (a managed vendor's UI, a petabyte-scale long-term-storage backend) says so
  plainly.
- **Do the "predict, then run" exercises.** Prometheus has several behaviors (counter resets, histogram bucket
  math, PromQL's staleness handling) that are easy to get wrong from reading alone and genuinely clarifying to
  watch happen.
- **Do the projects.** Each one is a real, runnable instrumentation, dashboard, or alerting pipeline, not a
  screenshot.

## More from the Mastery Path series

Free, structured, module-wise training across 98 other languages, platforms and disciplines — expand a category to browse:

<div class="mastery-tracks">
<details class="mastery-card">
<summary>Languages <span class="mastery-count">20</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/python-mastery-path/">🐍 Python</a>
  <a href="https://sigilipelli.github.io/java-mastery-path/">☕ Java</a>
  <a href="https://sigilipelli.github.io/javascript-mastery-path/">🟨 JavaScript</a>
  <a href="https://sigilipelli.github.io/typescript-mastery-path/">🔷 TypeScript</a>
  <a href="https://sigilipelli.github.io/csharp-mastery-path/">🔵 C#</a>
  <a href="https://sigilipelli.github.io/shell-mastery-path/">🐚 Shell/Bash</a>
  <a href="https://sigilipelli.github.io/powershell-mastery-path/">💻 PowerShell</a>
  <a href="https://sigilipelli.github.io/c-mastery-path/">🇨 C</a>
  <a href="https://sigilipelli.github.io/cpp-mastery-path/">➕ C++</a>
  <a href="https://sigilipelli.github.io/go-mastery-path/">🐹 Go</a>
  <a href="https://sigilipelli.github.io/rust-mastery-path/">🦀 Rust</a>
  <a href="https://sigilipelli.github.io/sql-mastery-path/">🗄️ SQL</a>
  <a href="https://sigilipelli.github.io/ruby-mastery-path/">💎 Ruby</a>
  <a href="https://sigilipelli.github.io/php-mastery-path/">🐘 PHP</a>
  <a href="https://sigilipelli.github.io/kotlin-mastery-path/">🟣 Kotlin</a>
  <a href="https://sigilipelli.github.io/swift-mastery-path/">🐦 Swift</a>
  <a href="https://sigilipelli.github.io/dart-mastery-path/">🎯 Dart</a>
  <a href="https://sigilipelli.github.io/scala-mastery-path/">🔴 Scala</a>
  <a href="https://sigilipelli.github.io/r-mastery-path/">📊 R</a>
  <a href="https://sigilipelli.github.io/matlab-mastery-path/">🟧 MATLAB</a>
</div>
</details>
<details class="mastery-card">
<summary>Web Frameworks <span class="mastery-count">11</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/react-mastery-path/">⚛️ React</a>
  <a href="https://sigilipelli.github.io/nextjs-mastery-path/">▲ Next.js</a>
  <a href="https://sigilipelli.github.io/angular-mastery-path/">🅰️ Angular</a>
  <a href="https://sigilipelli.github.io/vue-mastery-path/">💚 Vue.js</a>
  <a href="https://sigilipelli.github.io/html-css-mastery-path/">🎨 HTML & CSS</a>
  <a href="https://sigilipelli.github.io/tailwind-mastery-path/">🌬️ Tailwind CSS</a>
  <a href="https://sigilipelli.github.io/django-mastery-path/">🎸 Django</a>
  <a href="https://sigilipelli.github.io/fastapi-mastery-path/">🚀 FastAPI</a>
  <a href="https://sigilipelli.github.io/graphql-mastery-path/">🕸️ GraphQL</a>
  <a href="https://sigilipelli.github.io/nodejs-mastery-path/">🟩 Node.js</a>
  <a href="https://sigilipelli.github.io/spring-boot-mastery-path/">🍃 Spring Boot</a>
</div>
</details>
<details class="mastery-card">
<summary>Mobile <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/flutter-mastery-path/">💙 Flutter</a>
  <a href="https://sigilipelli.github.io/react-native-mastery-path/">📲 React Native</a>
</div>
</details>
<details class="mastery-card">
<summary>Testing & QA <span class="mastery-count">6</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/java-testing-mastery-path/">🧪 Java Testing</a>
  <a href="https://sigilipelli.github.io/cpp-testing-mastery-path/">🧪 C/C++ Testing</a>
  <a href="https://sigilipelli.github.io/python-testing-mastery-path/">🧪 Python Testing</a>
  <a href="https://sigilipelli.github.io/automotive-testing-mastery-path/">🚗 Automotive Testing</a>
  <a href="https://sigilipelli.github.io/selenium-mastery-path/">🧭 Selenium</a>
  <a href="https://sigilipelli.github.io/cypress-mastery-path/">🌲 Cypress</a>
</div>
</details>
<details class="mastery-card">
<summary>Security <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/cybersecurity-mastery-path/">🛡️ Cybersecurity</a>
  <a href="https://sigilipelli.github.io/ethical-hacking-mastery-path/">🎯 Ethical Hacking</a>
</div>
</details>
<details class="mastery-card">
<summary>Cloud Platforms <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/aws-mastery-path/">☁️ AWS</a>
  <a href="https://sigilipelli.github.io/azure-mastery-path/">☁️ Azure</a>
  <a href="https://sigilipelli.github.io/gcp-mastery-path/">☁️ GCP</a>
  <a href="https://sigilipelli.github.io/ibm-cloud-mastery-path/">☁️ IBM Cloud</a>
  <a href="https://sigilipelli.github.io/adobe-mastery-path/">🎨 Adobe</a>
</div>
</details>
<details class="mastery-card">
<summary>Data & Analytics <span class="mastery-count">7</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/data-engineering-mastery-path/">🛠️ Data Engineering</a>
  <a href="https://sigilipelli.github.io/data-science-mastery-path/">📈 Data Science</a>
  <a href="https://sigilipelli.github.io/tableau-mastery-path/">📊 Tableau</a>
  <a href="https://sigilipelli.github.io/excel-mastery-path/">📗 Excel</a>
  <a href="https://sigilipelli.github.io/pyspark-mastery-path/">⚡ PySpark</a>
  <a href="https://sigilipelli.github.io/etl-datalake-mastery-path/">🏞️ ETL & Data Lake</a>
  <a href="https://sigilipelli.github.io/power-bi-mastery-path/">📉 Power BI</a>
</div>
</details>
<details class="mastery-card">
<summary>Databases <span class="mastery-count">6</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/postgresql-mastery-path/">🐘 PostgreSQL</a>
  <a href="https://sigilipelli.github.io/mongodb-mastery-path/">🍃 MongoDB</a>
  <a href="https://sigilipelli.github.io/redis-mastery-path/">🟥 Redis</a>
  <a href="https://sigilipelli.github.io/snowflake-mastery-path/">❄️ Snowflake</a>
  <a href="https://sigilipelli.github.io/databricks-mastery-path/">🧱 Databricks</a>
  <a href="https://sigilipelli.github.io/python-mastery-path/vector-databases-mastery-path/">🧭 Vector Databases</a>
</div>
</details>
<details class="mastery-card">
<summary>AI / ML / LLM <span class="mastery-count">11</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/ai-ml-mastery-path/">🤖 AI/ML</a>
  <a href="https://sigilipelli.github.io/llm-dev-mastery-path/">🧠 LLM Dev</a>
  <a href="https://sigilipelli.github.io/rag-mastery-path/">📚 RAG</a>
  <a href="https://sigilipelli.github.io/edge-ai-mastery-path/">📱 Edge AI</a>
  <a href="https://sigilipelli.github.io/claude-training-mastery-path/">🔶 Claude Training</a>
  <a href="https://sigilipelli.github.io/ai-tools-mastery-path/">🧰 AI Tools</a>
  <a href="https://sigilipelli.github.io/ml-math-mastery-path/">➗ ML Math Foundations</a>
  <a href="https://sigilipelli.github.io/prompt-engineering-mastery-path/">✍️ Prompt Engineering</a>
  <a href="https://sigilipelli.github.io/ai-agents-mastery-path/">🕹️ AI Agents</a>
  <a href="https://sigilipelli.github.io/mlops-mastery-path/">🔁 MLOps</a>
  <a href="https://sigilipelli.github.io/computer-vision-mastery-path/">👁️ Computer Vision</a>
</div>
</details>
<details class="mastery-card">
<summary>Embedded Systems <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/embedded-mastery-path/">🔌 Embedded</a>
  <a href="https://sigilipelli.github.io/embedded-linux-mastery-path/">🐧 Embedded Linux</a>
  <a href="https://sigilipelli.github.io/embedded-python-mastery-path/">🐍 Embedded Python</a>
  <a href="https://sigilipelli.github.io/freertos-mastery-path/">⏱️ FreeRTOS</a>
  <a href="https://sigilipelli.github.io/s32k-mastery-path/">🔧 S32K</a>
</div>
</details>
<details class="mastery-card">
<summary>Leadership & Management <span class="mastery-count">5</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/product-manager-mastery-path/">📋 Product Manager</a>
  <a href="https://sigilipelli.github.io/product-lead-mastery-path/">🧭 Product Lead</a>
  <a href="https://sigilipelli.github.io/project-manager-mastery-path/">📅 Project Manager</a>
  <a href="https://sigilipelli.github.io/ai-manager-mastery-path/">🤖 AI Manager</a>
  <a href="https://sigilipelli.github.io/servant-leadership-mastery-path/">🤝 Servant Leadership</a>
</div>
</details>
<details class="mastery-card">
<summary>Professional Skills <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/english-fluency-mastery-path/">🗣️ English Fluency & IELTS</a>
  <a href="https://sigilipelli.github.io/workday-mastery-path/">🧑‍💼 Workday</a>
</div>
</details>
<details class="mastery-card">
<summary>Careers & Interviews <span class="mastery-count">2</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/dsa-interview-mastery-path/">🧩 DSA & Coding Interviews</a>
  <a href="https://sigilipelli.github.io/system-design-mastery-path/">🏗️ System Design</a>
</div>
</details>
<details class="mastery-card">
<summary>Process & APIs <span class="mastery-count">3</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/agile-mastery-path/">🔄 Agile/Scrum/Kanban</a>
  <a href="https://sigilipelli.github.io/rest-api-mastery-path/">🔗 REST API</a>
  <a href="https://sigilipelli.github.io/playwright-mastery-path/">🎭 Playwright</a>
</div>
</details>
<details class="mastery-card">
<summary>Infrastructure & Ops <span class="mastery-count">11</span></summary>
<div class="mastery-grid">
  <a href="https://sigilipelli.github.io/server-ops-mastery-path/">🖥️ Server Ops</a>
  <a href="https://sigilipelli.github.io/linux-mastery-path/">🐧 Linux</a>
  <a href="https://sigilipelli.github.io/nodemcu-mastery-path/">📶 NodeMCU/IoT</a>
  <a href="https://sigilipelli.github.io/terraform-mastery-path/">🌍 Terraform</a>
  <a href="https://sigilipelli.github.io/kubernetes-mastery-path/">☸️ Kubernetes</a>
  <a href="https://sigilipelli.github.io/docker-mastery-path/">🐳 Docker</a>
  <a href="https://sigilipelli.github.io/github-mastery-path/">🐙 GitHub & Git</a>
  <a href="https://sigilipelli.github.io/cicd-mastery-path/">🔁 CI/CD</a>
  <a href="https://sigilipelli.github.io/ansible-mastery-path/">🤖 Ansible</a>
  <a href="https://sigilipelli.github.io/kafka-mastery-path/">📨 Kafka</a>
  <a href="https://sigilipelli.github.io/microservices-mastery-path/">🧩 Microservices</a>
</div>
</details>
</div>
