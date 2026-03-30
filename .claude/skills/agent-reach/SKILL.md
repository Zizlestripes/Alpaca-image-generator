---
name: agent-reach
description: Gives AI agents internet access across 17+ platforms including Twitter/X, YouTube, Reddit, GitHub, Bilibili, XiaoHongShu, Douyin, Weibo, WeChat, LinkedIn, podcasts, RSS feeds, and web search — all free, zero API fees. Use when the user wants to install internet browsing capabilities for their agent, set up web access tools, read content from social media, search online platforms, or run "agent-reach install". Also triggers for: "give agent internet access", "browse the web", "search Twitter", "read YouTube", "access Reddit", "Chinese social media", "agent web tools".
---

# Agent Reach

Agent Reach equips AI agents with internet access across 17+ platforms. It is an **installer and health checker** — after setup, you call upstream tools directly (bird CLI, yt-dlp, gh CLI, mcporter, etc.).

## Installation

Tell your AI agent (or run directly):

```
帮我安装 Agent Reach：https://raw.githubusercontent.com/Panniantong/agent-reach/main/docs/install.md
```

Or install manually:

```bash
# Recommended
pipx install https://github.com/Panniantong/agent-reach/archive/main.zip
agent-reach install --env=auto

# If pipx unavailable (Homebrew Python / PEP 668)
python3 -m venv ~/.agent-reach-venv
source ~/.agent-reach-venv/bin/activate
pip install https://github.com/Panniantong/agent-reach/archive/main.zip
agent-reach install --env=auto
```

After install, run `agent-reach doctor` to verify which channels are active.

## Supported Platforms

| Platform | Tool | Notes |
|----------|------|-------|
| Web / General | Jina Reader + curl | No auth needed |
| Exa Search | mcporter | Configured automatically |
| Twitter/X | bird CLI | Needs cookies |
| YouTube | yt-dlp | No auth needed |
| Bilibili | yt-dlp | Residential proxy on servers |
| Reddit | curl | Residential proxy on servers |
| GitHub | gh CLI | Configured automatically |
| XiaoHongShu | mcporter + Docker | Needs cookies + Docker |
| Weibo | mcporter | No auth needed |
| Douyin | mcporter | No auth needed |
| WeChat articles | Camoufox | Auto-configured |
| LinkedIn | mcporter | Needs browser login |
| Xiaoyuzhou podcast | transcribe.sh | Needs free Groq key |
| V2EX | curl | Public API |
| Xueqiu | curl | Public API |
| RSS/Atom | feedparser | No auth needed |

## Key Commands

| Command | Purpose |
|---------|---------|
| `agent-reach install --env=auto` | Full auto-setup |
| `agent-reach install --env=auto --safe` | Safe mode (no auto system changes) |
| `agent-reach install --env=auto --dry-run` | Preview only |
| `agent-reach doctor` | Show channel status |
| `agent-reach watch` | Health + update check |
| `agent-reach configure twitter-cookies "..."` | Unlock Twitter |
| `agent-reach configure proxy URL` | Unlock Reddit/Bilibili on servers |
| `agent-reach configure groq-key gsk_xxx` | Unlock podcast transcription |
| `agent-reach configure xhs-cookies "..."` | Unlock XiaoHongShu |

## Using Platforms After Install

```bash
# Web reading
curl -s "https://r.jina.ai/https://example.com"

# Twitter search
bird search "query" -n 10

# YouTube metadata
yt-dlp --dump-json "https://youtube.com/watch?v=ID"

# Reddit (public)
curl -s "https://reddit.com/r/programming.json?limit=10"

# GitHub search
gh search repos "topic:llm" --limit 10

# Exa web search
mcporter call 'exa.web_search_exa(query: "your query", num_results: 5)'

# Weibo trending
mcporter call 'weibo.get_trendings(limit: 10)'

# XiaoHongShu search
mcporter call 'xiaohongshu.search_feeds(keyword: "query")'

# Podcast transcription
bash ~/.agent-reach/tools/xiaoyuzhou/transcribe.sh "https://www.xiaoyuzhoufm.com/episode/ID"
```

## Boundaries

- Never run commands with `sudo` unless user explicitly approved
- Never create files in the agent workspace — use `/tmp/` or `~/.agent-reach/`
- Never install packages not listed in the guide
- If a fix requires elevated permissions, tell the user and let them decide

## Credentials Setup

**Twitter cookies** — install [Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm), go to x.com, Export → Header String, then:
```bash
agent-reach configure twitter-cookies "PASTED_STRING"
```

**Residential proxy** (for Reddit/Bilibili on servers):
```bash
agent-reach configure proxy http://user:pass@ip:port
```

**Groq API key** (free, no credit card) — get at https://console.groq.com:
```bash
agent-reach configure groq-key gsk_xxxxx
```

## Resources

- GitHub: https://github.com/Panniantong/Agent-Reach
- Install guide: https://raw.githubusercontent.com/Panniantong/agent-reach/main/docs/install.md
- License: MIT
