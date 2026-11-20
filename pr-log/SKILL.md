---
name: pr-log
description: After a pull request is opened for the user, add a one-line entry (date, state, PR link, what changed, 200 characters or fewer) to the PR log section of their GitHub profile README (the `<login>/<login>` repository), and refresh the state (Open / Draft / Merged / Closed) of the PRs already logged. Use right after every successful `gh pr create` or Create-PR action, and when the user says "log this PR", "refresh PR states", /pr-log. Skip adding a PR that is already logged.
---

# pr-log

Keeps a running list of the user's pull requests, with their state, in their profile README.
The profile repo is `<login>/<login>` (`README.md`, default branch), where `<login>` comes from `gh api user --jq .login`. If that repo does not exist, stop and tell the user.
Once the user has installed this skill, do it right after opening a PR, without asking again.

## Entry format

One line, sorted by date descending (newest first; the same date keeps the newer entry on top):

```
- 2025-01-31 · 🟡 Open · [owner/repo#123](https://github.com/owner/repo/pull/123) — <summary>
```

- Date: the PR's `createdAt` date (`gh pr view <url> --json createdAt`), not today's.
- State, from `gh pr view <url> --json state,isDraft`:
  - `🟡 Open` (open, ready for review)
  - `⚪ Draft` (open draft)
  - `🟣 Merged`
  - `🔴 Closed` (closed without merge)
- `<summary>`: what the PR changes, in the language the README is written in, **at most 200 characters** (count the summary text only). One sentence, no emojis.
- Use only what is public on the PR (repo, number, title, diff). No private repo names, no secrets.
- A private repo's PR is not logged: check `gh repo view <owner/repo> --json visibility` and skip when it is not PUBLIC.

## Steps

1. `gh pr view <url> --json number,title,url,createdAt,state,isDraft` and write the summary from the diff, not just the title. (When only refreshing states, skip to step 2 and add no new entry.)
2. Read the README and its sha:
   ```
   gh api repos/<login>/<login>/contents/README.md --jq .sha
   gh api repos/<login>/<login>/contents/README.md -H "Accept: application/vnd.github.raw" > <scratchpad>/README.md
   ```
3. Refresh the state: for every entry between the markers, run `gh pr view <its url> --json state,isDraft` and replace only its state label when it changed; an older entry with no label gets one (`· <label> ·` after the date). Never touch the summary of an existing entry. If the optional workflow (`pr-log-refresh.yml`, see the repo README) is installed it refreshes labels on a schedule, so this step only matters between its runs; it only edits entries that already have a label.
4. If the new PR's URL is already in the file, add nothing. If nothing was added and no state changed, stop without committing.
5. The log lives between `<!-- pr-log:start -->` and `<!-- pr-log:end -->`. If the markers are missing, add a section with a heading in the README's language (e.g. `# Open-source pull requests`) containing the two markers, right below the introduction: after the first `---` separator in the file, or after the first heading block if there is none. End the section with its own `---` if the other sections use them.
6. Insert the new line so the entries stay sorted by date, newest first: put it before the first entry whose date is the same or older (directly under `<!-- pr-log:start -->` if there is none newer). If an existing entry is out of order, fix the order too (stable by date, descending). Change nothing else in the file, and keep its line endings.
7. Commit it with the contents API (sha from step 2; `content` is the file base64-encoded without line breaks):
   ```
   gh api -X PUT repos/<login>/<login>/contents/README.md -f message="docs: log PR <owner/repo>#<n>" -f content="$(base64 -w0 <scratchpad>/README.md)" -f sha=<sha> -f branch=<default branch>
   ```
   For a states-only refresh use the message `docs: refresh PR states`. If it fails with a sha conflict, re-read the README and redo from step 2.
8. Tell the user the entry that was added and any state that changed, in one line.

## Notes

- This writes to the user's public profile, so keep the entry factual and never exceed 200 characters.
- No agent attribution in the entry or commit message: the profile should read as the user's own log.
- If the push is denied, do not work around it. Give the user the exact line to paste.
