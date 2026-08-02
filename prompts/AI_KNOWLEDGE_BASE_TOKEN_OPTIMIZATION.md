# 🧠 PowerX Keys — AI Knowledge Base & Token Optimization Master Spec

> **SOP Reference**: Multi-AI Collaboration Protocol (`/factory`)  
> **Target Audience**: LLM Architects, AI Engineers, Performance Optimization Specialists  
> **App Context**: PowerX Keys V2 — Windows Desktop Automation App (C# WPF / .NET 10)

---

## 📌 IMPORTANT INSTRUCTIONS FOR REVIEWING AI AGENTS

> **ATTENTION AGENTS**: You must **thoroughly read and analyze** this entire document before answering.  
> 
> We are seeking **high-impact, actionable solutions** to optimize prompt token usage, reduce latency, and lower API quota consumption for the PowerX Keys AI Assistant while maintaining 100% response accuracy and intelligence.

---

## 🏛️ Current Context & Problem Statement

Currently, the PowerX Keys AI Assistant sends the full `AIAssistant_KnowledgeBase.txt` (~12KB / ~2,500 - 3,000 tokens) as context on **every single user request**, including simple greetings like *"Hi"* or *"Hello"*.

### ⚠️ Challenges With Current Approach
1. **Unnecessary Token Overhead**: 2,500+ tokens consumed per prompt even for trivial messages.
2. **Rate Limit Impact**: Accelerates reaching Tokens-Per-Minute (TPM) limits on free API tiers (Gemini, OpenRouter, Cerebras).
3. **Response Latency**: Processing larger input context adds 200–500ms of extra processing delay per request.

---

## 🎯 5 Core Brainstorming Topics for Reviewing AI Agents

Please provide detailed, structured recommendations for each of the following 5 topics:

---

### 1. 🗜️ Topic 1: Knowledge Base Compression & Token Pruning
* How can we compress a 12KB domain knowledge base down to **300–500 core tokens** without losing critical guardrails, hotkey rules, or app context?
* What information should be kept permanently in the system prompt vs stripped out?

---

### 2. 🔀 Topic 2: Mode-Based Context Splitting
* PowerX Keys has two distinct AI modes: **Chat Assistant** (Q&A/help) and **Macro Builder** (AHK v2 code generation).
* How should we split the knowledge base so Chat Mode only receives Q&A rules (~200 tokens) and Macro Builder Mode only receives code generation syntax rules (~400 tokens)?

---

### 3. 🧠 Topic 3: Server-Side Prompt Caching (Gemini & OpenRouter)
* Modern LLM providers (Google Gemini 2.0/2.5, OpenRouter, Anthropic) support **Implicit & Explicit Context Caching**.
* How can our backend cloud proxy leverage prompt caching so the 12KB Knowledge Base is cached once on the server, making subsequent prompt context cost **$0 / 0 extra token quota**?

---

### 4. ⚡ Topic 4: Dynamic Intent Classification (Greeting Bypassing)
* When a user sends casual greetings (e.g. *"hi"*, *"hello"*, *"thanks"*), passing the full knowledge base is wasteful.
* How can our lightweight client/proxy classifier detect simple casual greetings and reply instantly with a zero-context lightweight system prompt?

---

### 5. 🚀 Topic 5: Micro-RAG vs System Prompt Trade-offs
* Is implementing a local micro-vector database / RAG (Retrieval-Augmented Generation) overkill for a desktop app, or is a modular system prompt strategy superior for fast responses (<1s)?

---

## 📥 Required Response Format for Reviewing AI Agents

When answering, please structure your feedback into:
1. **Executive Evaluation & Architectural Score** (1-10 rating).
2. **Detailed Answers & Code/Prompt Strategy for Topics 1–5**.
3. **Sample Optimized System Prompt** (<400 tokens).
4. **Top 3 Recommended Quick Wins** for immediate implementation.
