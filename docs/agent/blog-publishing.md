# Blog article and publishing workflow

## Prepare the article

1. Identify the source material, intended audience, title, publication intent, and any details that must remain private.
2. Read `template/notes.md` and one or two nearby posts with similar content.
3. Create or update only the intended file under `content/post/`.
4. Restructure source notes into an article with a clear narrative. Preserve technical evidence, distinguish verified facts from interpretation, and remove conversational scaffolding.

## Frontmatter

Follow the repository template and current posts. A typical article uses:

```yaml
---
title: 文章标题
description: 一句话摘要
date: YYYY-MM-DD
image: https://image-1258996033.cos.ap-shanghai.myqcloud.com/westlake
tags:
  - Git
categories:
  - 工具
math: true
license:
hidden: false
comments: true
draft: false
toc: true
build:
  list: always
lastmod: YYYY-MM-DD
---
```

Use `draft: true` for an unpublished draft. Set `draft: false` only when the user intends the post to appear on the site. Update `lastmod` when revising an existing article.

Keep this footer at the end of a published article:

```markdown
# 参考文献

---

本文原载于 [巴巴变的博客](http://blog.bugxch.top)，遵循CC BY-NC-SA 4.0协议，复制请保留原文出处。
```

## Convert Obsidian Markdown

Hugo uses Goldmark, so convert repository-external Obsidian syntax:

| Obsidian source | Hugo article |
|---|---|
| `[[note]]` or `[[note\|alias]]` | public Markdown link, or plain text when the note is private |
| `> [!tip]` or another callout | ordinary blockquote with a bold lead-in |
| `![[image.png]]` | Markdown image whose asset or URL is publicly reachable |
| nested/private tags | concise public tags in frontmatter |
| `last modified: YYYY/MM/DD` | `lastmod: YYYY-MM-DD` |

The renderer allows raw HTML. Put literal placeholders such as `<path>` or `<branch>` inside backticks so Goldmark does not treat them as tags.

## Validate locally

From the repository root, inspect the exact article and run the strongest available checks:

```bash
python3 -c 'import pathlib, yaml; p=pathlib.Path("content/post/ARTICLE.md"); yaml.safe_load(p.read_text().split("---", 2)[1]); print("YAML OK")'
rg -n '<|\{\{' 'content/post/ARTICLE.md'
hugo --gc --minify
```

Review code fences manually or with a small parser; they must be balanced. Findings from `rg` are inspection candidates, not automatic failures: normal links and intentional Hugo syntax may be valid.

If local Hugo is unavailable, still validate YAML, frontmatter fields, Markdown links/images, code fences, footer, and repository diff. Do not claim a Hugo build passed when it was not run.

## Review and stage

Before committing:

```bash
git status --short
git diff -- 'content/post/ARTICLE.md'
git add -- 'content/post/ARTICLE.md'
git diff --cached --check
git diff --cached --stat
```

Stage supporting public assets only when the article actually uses them. Never stage unrelated vault, editor, theme, workflow, or generated `public/` changes.

Use a focused Chinese commit message consistent with repository history, for example:

```text
post: 发布文章标题
```

## Publish and verify

Push only when the user requested publication or push:

```bash
git push origin master
```

Then inspect the workflow and live page:

```bash
gh run list --workflow deploy.yml --limit 3
gh run view RUN_ID --log-failed
curl -sS -o /dev/null -w '%{http_code}\n' 'https://blog.bugxch.top/post/SLUG/'
```

Wait for the deployment run to complete before claiming publication succeeded. Report the article path, commit hash, pushed branch, Actions result, and live URL. If the slug differs from the filename, derive it from Hugo output or the effective permalink instead of guessing.
