# pr-log

A [Claude Code](https://claude.com/claude-code) skill that logs every pull request you open in your GitHub profile README, one line each: date, state, link, and a summary of at most 200 characters.

```
- 2025-01-31 · 🟣 Merged · [owner/repo#123](https://github.com/owner/repo/pull/123) — Fixes a crash when the temp path is too long.
- 2025-01-20 · 🟡 Open · [owner/repo#98](https://github.com/owner/repo/pull/98) — Adds a --dry-run flag to the export command.
```

- Entries are sorted by date, newest first.
- States: 🟡 Open, ⚪ Draft, 🟣 Merged, 🔴 Closed.
- Private repositories are never logged.

## Install

1. Copy the `pr-log` folder to `~/.claude/skills/pr-log` (Windows: `%USERPROFILE%\.claude\skills\pr-log`).
2. Install and sign in to the [GitHub CLI](https://cli.github.com/) (`gh auth login`). You need a profile repository named `<your-login>/<your-login>` with a `README.md`.
3. Open a PR as usual. The skill adds the entry after the PR is created, or run `/pr-log <PR URL>`.

The skill writes to your public profile README with `gh api`, so it only logs public PRs and adds no agent attribution.

## Optional: refresh states automatically

States change after you open a PR (merged, closed). To keep the labels current without spending any tokens, copy `pr-log/pr-log-refresh.yml` to `.github/workflows/pr-log-refresh.yml` in your profile repository. It runs daily (edit the `cron` line) and on demand, reads each logged PR with `gh pr view`, and commits only when a label changed. No LLM is involved. It only edits entries that already have a label.

Adding a workflow file through the API needs a token with the `workflow` scope; otherwise add the file in the GitHub web UI.

## 한국어

PR을 올릴 때마다 GitHub 프로필 README에 날짜, 상태, 링크, 200자 이내 요약을 한 줄씩 추가하는 Claude Code 스킬입니다. 위 설치 방법대로 `pr-log` 폴더를 `~/.claude/skills/`에 복사하고, `<내 아이디>/<내 아이디>` 프로필 저장소와 `gh` 로그인을 준비하세요. 요약은 README가 쓰인 언어로 작성됩니다.

## License

MIT
