# Repository instructions

## Scope

These instructions apply to the whole Hugo blog repository.

## Repository facts

- The published site is <https://blog.bugxch.top>.
- Posts live in `content/post/`.
- Use `template/notes.md` and nearby published posts as the current frontmatter and writing reference.
- The publish branch is `master`; `.github/workflows/deploy.yml` also accepts `main`.
- A push to a publish branch builds Hugo and deploys GitHub Pages.
- This repository is also an Obsidian vault and may contain unrelated automatic vault-backup changes.

## Article work

- Write Chinese unless the user requests another language.
- Preserve the author's meaning and voice. Turn notes and conversations into a coherent article rather than a transcript or a list of AI answers.
- Convert Obsidian-only syntax to Hugo-compatible Markdown.
- Put new articles at `content/post/<title-or-slug>.md` and follow existing frontmatter. Use `draft: false` only when the user intends the article to be public.
- Keep the existing copyright footer and a `# 参考文献` section at the end of published articles.
- Do not publish secrets, credentials, internal addresses, proprietary source, model data, private names, or identifying logs. Generalize or omit private details before writing.
- Do not change the theme, workflows, site configuration, or unrelated posts unless the user asks.

Read [the publishing workflow](docs/agent/blog-publishing.md) when creating, revising, validating, publishing, or verifying an article.

## Git and publication

- Inspect `git status --short` before edits and before staging. Preserve unrelated Obsidian, vault-backup, theme-update, and user changes.
- Stage explicit intended paths only; never use `git add -A` or `git add .` for an article publication.
- Writing or revising an article does not authorize commit or push. Stop after validation unless the user asked to commit or publish.
- A request to publish or push authorizes staging the intended article, committing it, and pushing the current publish branch. Report the commit and push result.
- After a publish push, verify the GitHub Actions deployment and live URL. If deployment fails, inspect the failed job, fix only the article-related cause, and retain the evidence.
