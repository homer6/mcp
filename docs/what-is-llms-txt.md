# llms.txt: What It Is and How It's Being Used

## Overview

`llms.txt` is a proposed web standard — a plain-text, Markdown-formatted file placed at a website's root directory (e.g., `example.com/llms.txt`) — designed to give large language models (LLMs) a curated, structured map of a site's most important content. Think of it as the AI-era counterpart to `robots.txt` and `sitemap.xml`, but purpose-built for reasoning engines rather than traditional search crawlers.[1][2][3][4]

The core problem it addresses: context windows are too small to process most websites in their entirety, and raw HTML pages are cluttered with navigation menus, JavaScript, ads, and other noise that reduces the space available for actual content. `llms.txt` solves this by presenting a clean, focused Markdown document that an AI agent can efficiently parse during inference.[5][6][2][7]

***

## Origin

The standard was proposed by **Jeremy Howard**, co-founder of Answer.AI and fast.ai, in September 2024. Howard's motivation was a specific technical challenge: AI systems needed a better way to navigate large documentation sites without blowing through their context windows or getting lost in HTML clutter. The proposal was originally published as a blog post on answer.ai, framing it explicitly as a proposal rather than a finalized protocol.[6][8][4][9][7][10]

It is important to note: `llms.txt` is **not** an officially ratified standard. No major AI platform — OpenAI, Google, or Anthropic — has formally endorsed or committed to reading it. Howard himself does not have it on his own site. Despite this, adoption accelerated rapidly once documentation platforms — especially Mintlify — began generating the file automatically for their customers in November 2024.[4][11][9][12]

***

## File Structure

The specification defines a Markdown file with a precise structure:[2][6]

- **H1 heading** — the project or site name (the only required element)
- **Blockquote** — a short summary providing key context for the LLM
- **Optional prose sections** — additional detail about the project or how to interpret the files
- **H2-delimited sections** — lists of URLs linking to further documentation or resources, each with a short description

The format was chosen over XML because it is both human-readable and natively understood by LLMs. Here is a simplified example structure:[2][4]

```markdown
# My Company Docs

> A developer platform for building payment APIs.

## Getting Started
- [Quickstart Guide](https://example.com/quickstart.md): Set up your first integration in 5 minutes.
- [Authentication](https://example.com/auth.md): API keys, OAuth, and security best practices.

## API Reference
- [Payments API](https://example.com/api/payments.md): Charge and capture payments.
- [Webhooks](https://example.com/api/webhooks.md): Receive real-time event notifications.
```

### Companion Files

The specification also introduces two companion file types:[13][6]

- **`llms-full.txt`** — A single, comprehensive Markdown export of all documentation. Cloudflare's version spans 3.7 million tokens across product-specific files. Vercel's is described as "a 400,000-word novel".[14]
- **`.md` page variants** — Individual pages can offer a clean Markdown version by appending `.md` to the URL (e.g., `example.com/docs/intro.md`), stripping HTML noise for direct LLM consumption.[5][2]

***

## How It Differs from robots.txt and sitemap.xml

| File | Purpose | Format | Audience | Enforcement |
|------|---------|--------|----------|-------------|
| `robots.txt` | Controls crawler access (allow/disallow) | Custom directives | Search engine bots | Widely followed convention |
| `sitemap.xml` | Lists all URLs for indexing | XML | Search engine bots | Used by crawlers for discovery |
| `llms.txt` | Curates key content for LLM understanding | Markdown | LLMs and AI agents | No enforcement; purely advisory |

Unlike `robots.txt`, `llms.txt` does not block anything — it is a guidance document, not an access control mechanism. And unlike `sitemap.xml`, which tries to enumerate every URL, `llms.txt` is intentionally curated: a short list of high-value content with descriptive context.[15][3][16][17]

***

## Who Is Using It

### Developer Tools and Documentation Platforms

The biggest accelerant of adoption was **Mintlify**, a documentation hosting platform that automatically enabled `llms.txt` generation for every site it hosts in November 2024. This instantly gave thousands of technical documentation sites the file, including:[11][4]

- **Anthropic** — A slim index file plus a comprehensive `llms-full.txt` Markdown export, designed to serve both real-time AI assistants and IDE integrations[13]
- **Cloudflare** — Organized by product/service (Workers, Agents, AI Gateway, etc.) so agents only fetch the relevant section rather than parsing the entire platform[18][14]
- **Stripe** — Organized around major product areas (Payments, Checkout, Webhooks, Testing) to make its sprawling API surface legible to models[14][13]
- **Vercel**, **Cursor**, **Pinecone**, **Windsurf**, **Hugging Face**, **Zapier** — all implemented the file[12][11]

Stripe's implementation stands out as an outlier: beyond a structured index, it includes an **instructions section** that functions like a system prompt — shaping how AI coding assistants like Cursor or Claude describe and recommend Stripe's products when a developer asks for payment integration advice. This represents a qualitatively new use: using `llms.txt` not just as a content map, but as a mechanism to **influence AI behavior at inference time**.[14]

### SEO Plugins and CMS Integrations

By late 2025 and into 2026, mainstream SEO plugins like Yoast began offering `llms.txt` generation features, though not enabling it by default. This has been a significant driver of adoption among non-developer websites.[19]

***

## Adoption Numbers

Adoption figures vary depending on the dataset and methodology:

- A study of the **top 100 websites** found 0% adoption among the largest global platforms[20]
- A study of the **top 1,000 websites** found only 3 out of 1,000 (0.3%) had implemented it as of mid-2025[21]
- A broader scan of **~300,000 domains** found ~10.13% had an `llms.txt` file, with adoption surprisingly flat across traffic tiers — high-traffic sites (100K+ visits) had slightly *lower* adoption (8.27%) than mid-traffic sites[22][23]
- **BuiltWith** tracked over **844,000 websites** implementing the file as of October 2025[11]

The wide variance in these numbers reflects different methodologies and the rapid pace of adoption growth. The consensus is that adoption is still early and uneven, heavily concentrated in developer tooling and documentation sites.

***

## Does It Actually Work?

This is where significant skepticism exists. Research to date suggests `llms.txt` has **no measurable impact on AI citations or referral traffic**:

- SE Ranking crawled ~300,000 domains and ran Spearman correlation tests plus an XGBoost regression model: no meaningful correlation was found between having an `llms.txt` and being cited in AI-generated answers. The machine learning model's performance actually *improved* when the `llms.txt` variable was removed — indicating it adds noise, not signal.[23]
- A 90-day experiment tracking traffic from AI tools (ChatGPT, Claude, Perplexity, Gemini) before and after adding `llms.txt` found traffic increases at two sites, but deeper investigation attributed those gains to other changes — not the file itself.[16]
- Google's official documentation on AI Overviews makes no mention of `llms.txt`; existing Search signals remain the basis for inclusion.[23]
- OpenAI's crawler documentation focuses on `robots.txt` controls and makes no promises about `llms.txt` influencing ranking or citations.[23]
- Server logs from multiple sites show very few AI crawlers actually requesting the file at all.[16]

The honest picture: **no major AI platform has confirmed they read or act on `llms.txt`**. The file may be fetched by some agents opportunistically, but there is no documented mechanism by which it improves AI visibility in the way that advocates claim.[9][12]

***

## Why Adopt It Anyway?

Despite the lack of confirmed efficacy, there are defensible reasons to implement `llms.txt`:

- **Low cost** — For documentation platforms and developer tools, generating the file is trivial, especially with tools like Mintlify that automate it[4]
- **Future-proofing** — If AI platforms do begin formally reading the file, early adopters will be ahead[19][11]
- **AI agent workflows** — For sites like Stripe, the file directly influences AI coding assistant behavior during developer sessions, independent of any ranking consideration[14]
- **Content clarity** — The exercise of curating a structured, concise summary of your site's key content has standalone value for any LLM that does fetch it[24][15]
- **Growing plugin support** — SEO tools are making it easier than ever to generate and maintain the file automatically[19]

The counterargument is straightforward: there is currently no strong evidence it moves the needle for AI citations, and content quality, structured data, and authoritative coverage of topics remain the most reliable drivers of AI visibility.[16][23]

***

## Current Status (April 2026)

As of early 2026, `llms.txt` occupies an interesting position: widely implemented among technical companies and documentation platforms, but unproven in its claimed purpose. SEO experts remain divided. Adoption is rising — driven largely by SEO plugins making it a one-click option — but it has not yet crossed the threshold of becoming a recognized best practice the way `robots.txt` or `sitemap.xml` have. Whether it evolves into a genuine standard depends largely on whether major LLM providers formally adopt it into their crawling and indexing workflows.[25][11][19]