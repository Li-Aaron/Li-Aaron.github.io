---
name: aaron-blog-style
description: "Use when writing or rewriting Chinese technical blog posts in Aaron's personal style, especially for document/blog-*.md or when asked to imitate posts from document/_posts. Covers Leanote, self-hosting, debugging, deployment, proxy, iOS/Android, RAG, and similar practical engineering notes."
---

# Aaron Blog Style

Use this skill when the task is to write or rewrite a Chinese technical blog post so it reads like the posts under `document/_posts`.

## Core Style

The target voice is a practical personal blog style, not a polished essay style.

- Write like someone sharing a useful折腾记录, not like someone writing a project retrospective.
- Start from a concrete motivation, pain point, or usage scenario.
- Explain things directly, usually with `本文介绍...`, `这里...`, `下面...`, `可以看到...`, `效果如下...`.
- Personal judgment is common, but should be short and practical, such as `推荐`, `不推荐`, `建议`, `这里要注意`.
- The tone is grounded, sometimes casual, and often a bit colloquial.

## Typical Structure

Common patterns in `_posts` are:

1. A short opening or `## 0. 前言`
2. A sentence explaining what the article covers
3. Numbered sections that follow the actual implementation or troubleshooting flow
4. Screenshots, examples, config snippets, and notes inserted where useful
5. A short closing note, update note, or unresolved issue if needed

Recommended heading patterns:

- `## 0. 前言`
- `## 1. ...`
- `## 2. ...`

Some posts also skip `前言` and go straight into `## 1 ...` if the topic is very direct.

## Language Habits

- Prefer short to medium paragraphs.
- Frequently use transitional phrases such as:
  - `本文介绍...`
  - `这里...`
  - `下面...`
  - `这里简单讲一下...`
  - `这里直接...`
  - `这里推荐...`
  - `需要注意...`
  - `效果如下：`
  - `详见...`
- Use first person when describing real choices or experience, but do not force it into every section.
- It is acceptable to include direct opinions like `官方这个地方没说清楚`, `其实不改也可以`, `不建议这样做`.

## Content Preferences

- Favor usable information over abstract reflection.
- Keep the article close to actual usage, deployment, troubleshooting, or configuration steps.
- When summarizing a system or feature, break it into practical categories instead of philosophical themes.
- Mention tradeoffs in a plain way: what works, what is troublesome, what is enough for now.
- If the topic is broad, add short bullets or lists to improve readability.

## Markers Found In `_posts`

These are representative patterns from the reference posts and should guide the tone:

- `本文介绍...`
- `这里重新介绍一下...`
- `虽然...，但是...`
- `这里简单写一点。`
- `下面可以...了。`
- `这里推荐...`
- `建议...`
- `不推荐...`
- `关于...详见...`
- `效果如下：`
- `注：...`

## What To Avoid

- Do not write like a formal project report.
- Do not overuse abstract summaries such as “平台化”“能力层”“统一治理” unless the article really needs them.
- Do not sound like product copy or AI-generated commentary.
- Do not force dramatic contrast phrases like `表面上看...但实际上...` unless they naturally match the source style.
- Do not make every section introspective; old posts are usually more direct and utility-first.

## Rewriting Checklist

When rewriting an existing article into this style:

1. Change abstract opening paragraphs into concrete background and purpose.
2. Replace retrospective phrasing with practical explanation.
3. Add direct guide-style connectors like `这里...` and `下面...`.
4. Keep technical substance, but make it read more like a personal how-to note.
5. End with a short takeaway, recommendation, or current limitation if appropriate.

## Output Pattern

When possible, produce:

- A direct title
- A short intro or `## 0. 前言`
- Numbered sections following the real order of work
- Practical notes, recommendations, or caveats
- A brief ending or update note

The result should feel like a real post from `document/_posts`: practical, direct, and useful, with a small amount of personal experience mixed in.