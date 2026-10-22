# hermes-linkedin-posts

A Hermes agent skill for writing sourced LinkedIn posts with benchmark visuals, publishing them through Composio, and iterating from analytics snapshots.

- `linkedin-posts/SKILL.md` — workflow: research and cite sources, capture real benchmark visuals, draft, publish (post + carousel/image + first comment), schedule one-shot posts, analytics-informed iteration.
- `linkedin-posts/references/composio-linkedin-publish.md` — publishing through Composio's managed LinkedIn connection (images, comments, video upload).
- `DESCRIPTION.md` — category description.

Install: copy `linkedin-posts/` into `$HERMES_HOME/skills/social-media/`. Link LinkedIn first (`composio link linkedin`); the skill never handles passwords or tokens.

License: MIT.
