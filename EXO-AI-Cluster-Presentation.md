# 🖥️ EXO AI Cluster — Executive Presentation

> **In one line:** We built a **private, on-premise AI (LLM) cluster** by connecting 4 Mac minis. No cloud, no subscription, no data leaving the building. It is running now and ready to chat.

---

## 1. Executive Summary

| | |
|---|---|
| **What it is** | A private LLM cluster running on 4× Mac mini (exo / exolabs) |
| **Key advantage** | Fully **private (offline)**, no cloud, **no per-token / monthly fees**, data stays in-house |
| **Current status** | ✅ Live — Qwen3.6-27B model loaded, chat-ready |
| **Access** | `http://192.168.5.14:52415` (in-browser, on the local network) |
| **Capability tier** | Comparable to a mid-tier commercial model (**GPT-4o-mini / Claude Haiku**) |

---

## 2. System & Hardware

**4 × Apple Mac mini (M4)**

| Spec | Per node | Total (4 nodes) |
|---|---|---|
| Processor | Apple **M4** chip (10-core CPU / 10-core GPU / 16-core Neural Engine) | 4 chips |
| Unified memory | **16 GB** | **64 GB** |
| Operating system | macOS 26.5.1 | identical (required for RDMA) |
| Networking | Gigabit Ethernet + Thunderbolt 4 (3 ports) | switch + Thunderbolt links |
| Software | exo (exolabs) official app v1.0.71 | merges the nodes into one cluster |

**Connection design:**
- **Ethernet** — device discovery + control plane.
- **Thunderbolt** — fast node-to-node data path (4 nodes currently cabled in a ring).
- A 5th node (extra Mac mini) can be added.

**Resilience:** Auto-login enabled + sleep disabled — if power drops or a node reboots, the Macs log in and the cluster re-forms automatically.

---

## 3. AI Capabilities

**Memory ceiling (64 GB):** in practice runs open-source models up to **~27–32 billion parameters (27–32B)**.

**Currently running:** `Qwen3.6-27B` (8-bit) — a reasoning model.

**Models it can run:** Qwen 27–32B, GLM-4.7-Flash, Gemma 27–31B, Llama Nemotron, and similar open-source models.

### What paid model is it equivalent to?

| | |
|---|---|
| **Tier** | ≈ **GPT-4o-mini · Claude Haiku · Gemini Flash** (mid-tier / fast commercial models) |
| **Good at** | Chat, translation, document Q&A (RAG), summarization, coding assistance, offline AI |
| **Limits** | Does not reach flagship-tier reasoning (GPT-4o / Claude Sonnet–Opus / Gemini Pro) |

> ⚠️ Reaching flagship-tier models needs much more memory (more nodes, or 64–128 GB Macs / Mac Studio).

---

## 4. Performance & Limits

| Metric | Value |
|---|---|
| Speed | ~**3 tokens/second** (27B model, split across 4 nodes) |
| Bottleneck | Per-node compute (M4 GPU) + total memory |
| Model load | ~15 seconds from cache |

**Note:** ~3 tok/s is fine for chat, document work, and experimentation; slower for long-form generation. The main levers to increase speed are a smaller model (more tok/s) or more / more-powerful nodes.

---

## 5. Thunderbolt Upgrade & Cable Cost

**Current state:** the nodes are connected with **ordinary USB-C (monitor) cables** → links run at **20 Gb/s**. They work, but these are **not true Thunderbolt cables**.

**Upgrade:** to activate exo's fastest **RDMA (RDMA-over-Thunderbolt)** path, you need **certified Thunderbolt 4 (40 Gb/s) cables** (marked with the ⚡ symbol and "Thunderbolt 4 / 40Gbps").

### Cable prices — researched (July 2026)

You need short (0.5–0.8 m) **certified Thunderbolt 4 / 40 Gbps** cables. **4 cables** for a ring (RDMA minimum), **6** for a full mesh (best).

| Source | Example product | Price / cable | Ring (4×) | Mesh (6×) |
|---|---|---|---|---|
| 🇨🇳 **AliExpress** ⭐ *(ships to Mongolia)* | UGOURD / Byscoon TB4, 40 Gbps, 8K, 100W | **~₮25,000–30,000** (~$8) | **~₮110,000** (~$32) | **~₮170,000** (~$48) |
| 🇨🇳 AliExpress (budget) | USB4 40 Gbps 8K (E-marker) | ~₮16,000 (~$5) | ~₮65,000 | ~₮98,000 |
| 🇲🇳 **Stora.mn** *(order-in)* | Maxonar TB4, 40 Gbps, 240W, 8K | ~₮42,500 ($20 / 2-pack) | ~₮170,000 | ~₮255,000 |
| 🇲🇳 techzone.ub (Instagram) | TB4 2-in-1 cable | ~₮150,000 (~$42) | ~₮600,000 | ~₮900,000 |
| 🇲🇳 Shoppy.mn / CODY.mn | **Apple** Thunderbolt 4 Pro (1 m) | ~₮310,900 *(out of stock)* | ~₮1,243,600 | ~₮1,865,400 |

> 💡 **Recommendation:** the practical choice is **AliExpress (~₮25–30k/cable, ships to Mongolia)** — a 4-cable ring costs ~**₮110,000 (~$32)**. If you want faster local delivery, **Stora.mn (Maxonar)**. Avoid the Apple Pro cable unless you specifically want it (10× the price).
> *Note: ikon.mn is a news site and Monos is a pharmacy — neither sells cables.*

### ⚠️ Honest note (important for the director)
For the **current workload (pipeline-parallel inference), the network is NOT the bottleneck** — per-node compute is. So even with Thunderbolt 4 cables, speed likely **won't improve dramatically** (still ~3 tok/s). RDMA's real benefit appears in **tensor-parallel mode** and **when scaling up**. Therefore we recommend treating the cable spend as a **future-scaling investment, not an urgent purchase**.

---

## 6. Remote Access (using it from outside the local network)

The cluster API is at `http://192.168.5.14:52415`, reachable on the **local network only**. To use it from elsewhere:

> ⚠️ **The exo API has NO authentication.** Never expose port 52415 directly to the internet (raw port-forwarding) — anyone who finds it could use or abuse the cluster.

| Method | Security | Exposes home IP | Ease | Best for |
|---|---|---|---|---|
| **Tailscale (VPN)** ⭐ | High (encrypted) | No | Very easy | Your own devices only |
| **Cloudflare Tunnel + Access** ⭐ | High (auth + HTTPS) | No | Medium | Public domain + login |
| Reverse proxy + auth (Caddy/nginx) | Medium–high | Yes | Medium | If you have a VPS/domain |
| Raw port-forwarding | ❌ Poor | Yes | Easy | **Don't** — no auth |
| SSH tunnel | High | No | Easy | Occasional access |

**Recommended:**
- **Private (just you):** install **Tailscale** on a node → reach `http://<tailscale-ip>:52415` from anywhere. No port-forwarding, works behind NAT/CGNAT, encrypted.
- **Public with a domain:** run **Cloudflare Tunnel** on a node → map a domain (e.g. `exo.linky.mn`) and put **Cloudflare Access** in front so only your email/SSO can enter. HTTPS is automatic and the home IP stays hidden.

---

## 7. Cost & Value

| | This solution | Cloud AI API |
|---|---|---|
| Cost | **One-time** hardware | Ongoing, per token / usage |
| Privacy | **Fully private**, data in-house | Data sent to a third party |
| Limits | Unlimited use | Usage caps / rate limits |
| Internet | Not required (offline) | Required |

**Hardware cost (approx.):** Mac mini M4 (16 GB) ~$599 × 4 ≈ **$2,400**. If already purchased, the only additional cost is cables (optional).

---

## 8. Conclusion & Recommendations

✅ **Usable today:** a private, cloud-free, fee-free AI cluster is running and ready for mid-tier (GPT-4o-mini / Haiku) tasks.

**Recommendations:**
1. **Now:** start using it at the current 20 Gb/s setup — chat, translation, document work, internal tools.
2. **Next (optional):** for stronger AI, adding memory (more nodes / higher-RAM Macs) delivers more than Thunderbolt cables (bigger models, higher quality).
3. **Thunderbolt 4 cables:** buy only if you plan tensor-parallel / scaling in the future (~$100–240). Not urgent for current chat workloads.
4. **Remote access:** if needed, use Tailscale (private) or Cloudflare Tunnel (with a domain) — securely, not raw port-forwarding.

---

*Prepared from the actual EXO cluster setup and test results. Pricing is approximate — verify before purchase.*
