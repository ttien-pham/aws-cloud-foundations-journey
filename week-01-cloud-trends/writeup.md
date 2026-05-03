# Week 1 — Trends in Cloud Computing ☁️

> *AWS Academy Foundation | Module 1*

---

## 📌 Overview

This module introduces current trends in cloud computing, with a focus on how AWS
is evolving — particularly its push into generative AI tooling for developers and
enterprises.

---

## 🧠 What I Learned

Cloud computing is no longer a stable, slow-moving field. AWS ships new services,
pricing changes, and infrastructure updates continuously. The practical takeaway
is simple but important: **treating cloud knowledge as a one-time certification
is a career risk.**

The module highlights three specific directions AWS is betting on right now:

- **Lower cost & finer billing granularity** — making cloud accessible to
  smaller teams and experiments
- **Expanded global infrastructure** — new Regions reducing latency for
  end-users worldwide
- **Generative AI as a first-class citizen** — not bolted on, but built into
  the developer workflow

Two services that stood out:

**Amazon Bedrock** — a managed service giving access to foundation models
(FMs) from multiple providers without needing to train or host models yourself.
The interesting design choice here is that AWS acts as an *aggregator* of AI
models rather than pushing only their own.

**Amazon Q** — an AI assistant embedded across AWS products. The split between
*Q Business* (enterprise knowledge + task completion) and *Q Developer*
(coding assistant, security scanning, code upgrades) tells me AWS sees AI as
a layer on top of existing workflows, not a replacement.

---

## 🔍 My Understanding

The framing I find most useful: AWS is moving from being an **infrastructure
provider** to a **development ecosystem**. Ten years ago the value proposition
was "rent servers cheaply." Today it is "build faster with AI-assisted tooling
on managed infrastructure."

This changes what it means to be a developer on AWS. You are not just deploying
code — you are choosing which AI services to compose, which foundation models
to use, and how to manage the cost and trust implications of each.

---

## 🌍 Real-World Connection ⭐

I am currently building a student document management system. Mapping what I
learned today to that project:

| What I learned | How it could apply |
|---|---|
| Amazon Bedrock (FM access) | Auto-summarize uploaded documents |
| Amazon Q Developer | Accelerate backend code generation and review |
| Continuous AWS updates | Need to check service limits & pricing before production |

One honest constraint I have to keep in mind: **generative AI on AWS is not
free.** For a student project, cost needs to be the first thing I evaluate
before adding any AI feature — not an afterthought.

---

## 💡 Insights

**The one thing I did not expect:** AWS's strategy with Bedrock is to offer
*multiple* foundation models (from Anthropic, Meta, Mistral, etc.) rather than
locking customers into Amazon's own model. This is a marketplace play — it
reduces the risk of customers leaving because they prefer a different model
provider.

**The tension I notice:** Cloud makes development faster and accessible, but it
also creates dependency. If a service gets deprecated or prices change, teams
built around it are exposed. This is not a reason to avoid cloud — but it is
a reason to understand the abstraction layers you are building on.

---

## ⚖️ Reflection

**What clicked:**
- AWS is not just infrastructure — it is becoming the operating system for
  AI-assisted software development
- Staying current is not optional; it is part of the job

**What is still unclear to me:**
- How does Bedrock's model routing actually work? Does it add latency compared
  to calling a model API directly?
- What are the real-world cost differences between using Q Developer vs. GitHub
  Copilot for a small team?

**What I want to do next:**
- Get hands-on with S3 and EC2 — I want to understand the core before
  the AI layer
- Price out what a Bedrock integration would actually cost for my document
  project

---

## 🗣️ If I Had to Explain This to a Friend

Cloud computing used to mean "someone else's computer." Now it means
"someone else's computer, with an AI assistant built in, that helps you build
products faster than you could alone."

The catch: you pay for it, you depend on it, and you need to keep learning
because it changes every few months.

---

## 📚 Resources I Want to Follow

- [What's New with AWS](https://aws.amazon.com/new) — bookmark for weekly check-in
- [AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/)
- [Amazon Bedrock](https://aws.amazon.com/bedrock/)
- [Amazon Q](https://aws.amazon.com/q/)