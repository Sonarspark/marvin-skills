# paperclip-skills

Backup of all skills installed in the Paperclip instance (container `paperclip`, company `cfcfc30d-…`), exported 2026-10-12.

- `skills/<slug>/` – skill files (current version of the GitHub-sourced skills in Paperclip, plus `pptx-from-html` from `/paperclip/tools/skills`). Paperclip-specific skills (bundled with the image) are deliberately not included.
- `MANIFEST.md` – source of every skill.

Not included: skill DB metadata (comments, stars, test runs) – those are in the Paperclip Postgres backup. Third-party skills keep their original licenses.
