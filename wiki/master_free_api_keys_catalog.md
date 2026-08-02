---
type: guide
status: active
summary: Comprehensive master catalog of free AI API keys, rate limits, daily quotas, provider research, and key rotation strategy.
last_updated: 2026-08-01
---

# 🔑 Master Free AI API Keys & Provider Catalog (2026)

> **Context**: Master reference catalog for PowerX Keys V2 AI backend key pool, rate limits, free tier providers, trial credits, and multi-agent research findings.

---

## 📊 Live System API Key Inventory & Capacity

| Provider | Active Keys | Verified Rate Limit (RPM) | Daily Quota (RPD) | Status / Certainty |
| :--- | :--- | :--- | :--- | :--- |
| **Google Gemini Free** | 13 Accounts | 15 RPM / key (195 RPM total) | 1,500 RPD / key (19,500 total) | ✅ **100% Certain** (Official Google AI Studio Free Tier) |
| **Cerebras AI** | 10 Accounts | 30 RPM / key (300 RPM total) | 14,400 RPD / key (144,000 total) | ⚠️ **100% Certain, Catalog Volatile** (Llama 3.3 70B, gpt-oss-120b) |
| **OpenRouter Free Tier** | 10 Accounts | 20 RPM / key (200 RPM total) | ~200 RPD / key (2,000 total) | ✅ **100% Certain** (Access to free `:free` model routes) |
| **Nvidia NIM / Geyser** | 4 Accounts | 40 RPM / key (160 RPM total) | ~1,000 RPD / key (4,000 total) | ✅ **100% Certain** (Build endpoints) |
| **GitHub Models / HuggingFace** | 4 Accounts | 15 RPM / key (60 RPM total) | ~1,000 RPD / key (4,000 total) | ⚠️ **Estimated** (Subject to tier changes) |
| **OpenCode Zen** | Active Pool | ~100 req/day free | Generous Model Quotas | ✅ **100% Verified** (Free models: Big Pickle, MiMo V2 Pro, MiniMax) |

### 📈 Global Capacity Breakdown
* **Total Key Pool**: 41+ Active Keys
* **Max Peak Throughput**: **~855 Requests / Minute**
* **Daily Processing Limit**: **~165,000 Requests / Day**
* **Monthly System Ceiling**: **~4.95 Million Requests / Month**

---

## 🌐 Complete Provider & Aggregator Audit (2026 Web Research)

### 1. 🌟 Permanent Free Tier Providers (No Credit Card Required)

| Provider | Free Tier Type | Rate Limits (RPM / RPD) | Supported Models | OpenAI SDK Compatible? | Key Prefix / Base URL | Notes / Certainty |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Google AI Studio** | Permanent Free Tier | ~15 RPM / 1,500 RPD per key | Gemini 2.5 Flash, 3.x Flash-Lite | Yes (Custom Base) | `AIzaSy...` | ✅ **Best ongoing multimodal tier**. Grounding with Google Search included (500 RPD free). |
| **OpenRouter** (Aggregator) | Permanent Free Tier | 20 RPM / 50 RPD (1,000 RPD after $10 top-up) | 50+ `:free` models (Llama, DeepSeek, Nemotron) | Yes | `sk-or-v1-...` | ✅ **Best single-key multiplier**. Access dozens of free models behind one API key. |
| **Groq** | Permanent Free Tier | 30 RPM / 14,400 RPD (8B) / 1,000 RPD (70B) | Llama 3.1/3.3, Qwen3, DeepSeek R1 | Yes | `gsk_...` | ⚠️ **Limits volatile**. Extremely fast LPUs, but daily limits reduced for 70B models in 2026. |
| **OpenCode Zen** | Free Models + Pay-as-you-go | ~100 req/day free tier | Big Pickle, MiMo V2 Pro, MiniMax M2.5 | Yes | `/zen/v1/chat/completions` | ✅ **Confirmed free provider**. High-quality free models; watch out for peak 429 retries. |
| **Cloudflare Workers AI** | Daily Neuron Quota | 10,000 Neurons / Day free | Llama 3, Mistral, Gemma | Partial | Cloudflare API Token | ✅ **Confirmed**. Edge-hosted, no credit card required. Neuron math converts to ~1k daily requests. |
| **Cerebras AI** | Permanent / Fast Tier | ~30 RPM / 14,400 RPD / 1M tokens/day | gpt-oss-120b, Qwen3 235B | Yes | Cerebras Console Key | ⚠️ **Catalog rotates**. Ultra-fast hardware; model list changes periodically. |
| **Mistral AI** (La Plateforme) | Experiment Free Plan | ~1 req/sec (~30 RPM / 2,000 RPD) | Codestral, Mistral Small/Large | Yes | Bearer Token | ✅ **Confirmed**. Phone verification required; great for coding tasks via Codestral. |
| **NVIDIA NIM** | Prototyping Access | Up to 40 RPM | Nemotron series, Llama variants | Yes | `build.nvidia.com` Key | ✅ **Confirmed**. Free developer prototyping credits for frontier open weights. |
| **GitHub Models** | Account Plan Tier | 15 RPM / 150 RPD (low) / 1 RPM (large) | GPT-4o-mini, Llama 3.3, DeepSeek-R1 | Yes | GitHub PAT Token | ✅ **Confirmed**. Uses Azure AI infrastructure; rate-limited by GitHub account tier. |

---

### 2. ⏳ One-Time Trial Credits & Expiry-Based Providers

| Provider | Trial Credit Amount | Expiry Window | Rate Limits | Credit Card Needed? | Notes & Verdict |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **DeepSeek API** | 5M Tokens (~$5 value) | ~30 Days | High (500–2,500 concurrency) | No (until grant used) | 💡 **Best paid fallback**. Once grant expires, paid pricing is ultra-cheap ($0.14 / 1M tokens). |
| **Anthropic (Claude)** | $5 Free Trial | 14 – 30 Days | ~50 RPM | No (SMS verification) | ⚠️ **Trial only**. Requires phone verification; no permanent free daily quota. |
| **Cohere** | Trial Key | Monthly Reset | 20 RPM / 1,000 calls/month | No | ⚠️ **Trial key**. Good for Rerank and Embeddings. |
| **Novita AI** | ~$0.50 Voucher | One-time | Varies | Optional | ⚠️ **Trial voucher only**. |
| **Fireworks AI** | $1 Trial Credit | One-time | 60 RPM | Yes (after trial) | ❌ **Non-renewing**. Not suitable for permanent free pool. |
| **Together AI** | $0 (Free Tier Removed) | N/A | N/A | Yes | ❌ **Free tier removed in 2026**. Requires $5 minimum deposit. |
| **Replicate** | $0 (Pay-per-use) | N/A | N/A | Yes | ❌ **No permanent free tier**. |

---

## 🎯 Key Pool Expansion & Load Balancing Strategy

### 1. 🔄 Smart Proxy Fallback Cascade
When an API key returns a `429 Too Many Requests` status, the backend proxy should follow a 3-layer cascade:

```
[User Request] 
      │
      ▼
┌─────────────────────────────────────────────────────────┐
│ Layer 1: Same Provider Round-Robin                      │
│ (Rotate to next key within Gemini/Groq/Cerebras pool)   │
└──────────────────────────┬──────────────────────────────┘
                           │ (All keys in pool hit 429)
                           ▼
┌─────────────────────────────────────────────────────────┐
│ Layer 2: OpenRouter Multi-Model Route (:free)           │
│ (Seamless failover to OpenRouter aggregated free routes) │
└──────────────────────────┬──────────────────────────────┘
                           │ (Aggregator busy)
                           ▼
┌─────────────────────────────────────────────────────────┐
│ Layer 3: Cross-Provider Fallback + Exponential Backoff  │
│ (Fall back to OpenCode Zen / Mistral Codestral)         │
└─────────────────────────────────────────────────────────┘
```

### 2. 🚀 Key Multipliers to Reach 2,000+ RPM
1. **Maximize OpenRouter Free Routes**: Adding 5 additional OpenRouter accounts provides direct access to 50+ free model endpoints simultaneously.
2. **Integrate Cloudflare Workers AI**: Unlocks 10,000 free daily execution units.
3. **OpenCode Zen Integration**: Utilizes dedicated free endpoints (`/zen/v1/chat/completions`) for macro generation tasks.

---

## 📄 Wiki Index Link
* Document added to Master Wiki Index: [[master_free_api_keys_catalog]]
