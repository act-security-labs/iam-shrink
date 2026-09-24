---
name: iam-shrink
description: Shrink a long AWS IAM action list into wildcard patterns that match exactly those actions and nothing more, verified by a round trip through iam-expand, by running iam-shrink through npx with nothing installed. Use when a policy is near its size limit or an action list is too long to review.
license: MIT
compatibility: Node.js 22 or newer with npx; network access to registry.npmjs.org on first run.
metadata:
  author: act-security
  version: "0.1.0"
---

[iam-shrink](https://github.com/act-security-labs/iam-shrink) replaces a list of IAM actions with wildcard patterns that match exactly those actions. Same two input modes as `iam-expand`: actions as arguments or stdin lines, or a JSON policy on stdin rewritten in place.

## Run

```bash
npx -y @actsecurity/iam-shrink@latest < actions.txt
cat policy.json | npx -y @actsecurity/iam-shrink@latest --iterations 0
```

Actions are split on camel-case words (`s3:GetObjectTagging` is Get, Object, Tagging) and only whole words are replaced with `*`, one at a time, so `s3:GetObject` can become `s3:Get*` but never `s3:*et*`. The result stays reviewable by a human.

`--help` is the authoritative flag list. Only what it does not say is below.

## What `--help` does not say

- **A bare `*` behaves differently per input mode.** As an argument or inside a JSON policy the whole result becomes `*`, which is correct. On plain stdin lines the `*` line is dropped and the rest is shrunk, so a grant-everything list comes back looking narrow. Check the input for `*` first; if it is there, the user's question is not a shrink question.
- **Existing wildcards are kept** unless they match nothing, are covered by a broader pattern already present, or a smaller pattern replaces them. To rebuild from scratch, expand first: `npx -y @actsecurity/iam-expand@latest | npx -y @actsecurity/iam-shrink@latest`.
- **`--iterations` trades readability for size.** Default 2 keeps the patterns recognizable; `0` runs until nothing changes and gives the smallest, least readable result, the setting when a policy is over its size limit. Measure with `wc -m` before and after so the user sees the gain.
- **`--levels read list tagging`** leaves every write and permissions-management action spelled out, the safer default when rightsizing rather than fighting a size limit.
- **`--remove-sids` and `--remove-whitespace`** exist for size limits only; they change nothing about permissions.
- **Stale catalog.** If stderr warns the data package is over five days old, re-run once as `npx -y -p @actsecurity/iam-data@latest -p @actsecurity/iam-shrink@latest iam-shrink`.

## Verify before handing back

A shrunk pattern is only correct if it expands back to the original set:

```bash
npx -y @actsecurity/iam-shrink@latest < actions.txt | npx -y @actsecurity/iam-expand@latest | sort > after.txt
npx -y @actsecurity/iam-expand@latest < actions.txt | sort > before.txt
diff before.txt after.txt
```

An empty diff is the completion criterion. A non-empty diff means the shrink widened the grant: report the extra actions and do not present the shrunk list as equivalent.

## Present

The patterns verbatim, the before and after character counts, and the sentence that the round trip was identical. Then your reading in its own section, labeled as yours.
