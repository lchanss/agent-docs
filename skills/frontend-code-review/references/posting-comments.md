# Posting Review Comments

Read this only when the user passed `--comment`. Without that flag, never call
`gh api` or `gh pr comment`.

## Hard rules

- **Never** `gh pr review --approve` or `gh pr review --request-changes`.
  Approval is a human decision.
- Post only `blocking` and `important` findings inline. Nits and praise stay in
  the chat report.
- Post at most 5 inline comments. If more survived verification, post the top 5
  and say in the report what was held back.
- Every comment body ends with the footer below, so nobody mistakes it for a
  human review.

## Line numbers

Comments anchor to a line in the **new** file. Read it off the diff hunk header:

```
@@ -10,5 +12,7 @@
        old start,count   new start,count
```

The first line after this header is new-file line 12. Count forward, skipping
lines that start with `-` (they do not exist in the new file). Context lines and
`+` lines both advance the counter.

Verify the anchor: the line you computed must contain the code your finding
quotes. If it does not, recount rather than posting at a guessed line.

## Single comment

```bash
gh api "repos/{owner}/{repo}/pulls/{number}/comments" \
  -f body="$BODY" \
  -f path="src/pages/Editor.tsx" \
  -f commit_id="$HEAD_SHA" \
  -F line=42 \
  -f side="RIGHT"
```

`commit_id` is the PR's head SHA — `gh pr view <n> --json headRefOid`. A stale
SHA makes the call fail.

For a range, add `-F start_line=38 -f start_side="RIGHT"`.

## Body format

Match the report's language (Korean by default). Keep it to the claim, the
failure scenario, and — for suggestions — the trade-off.

~~~
🔴 **정확성**

<한 줄 주장>

<구체적 실패 시나리오: 어떤 입력/상태에서 무엇이 잘못되는가>

```suggestion
<대체 코드, 해당 라인을 그대로 교체할 수 있을 때만>
```

---
> 🤖 Claude Code 자동 리뷰입니다. 승인/거부는 사람이 판단합니다.
~~~

A `suggestion` block must replace exactly the commented line range, and must
compile on its own. If the fix spans more context than the anchor covers,
describe it in prose instead.

## Fallback

If the inline call fails (outdated SHA, line outside the diff, file renamed),
do not retry with a different line. Post one general comment instead:

```bash
gh pr comment <number> -R <owner/repo> --body-file <path>
```

State the file and line in the body since there is no anchor.

## Nothing found

Post a plain comment, never an approval:

```bash
gh pr comment <number> -R <owner/repo> -b "리뷰 결과 지적 사항 없습니다.

---
> 🤖 Claude Code 자동 리뷰입니다. 승인/거부는 사람이 판단합니다."
```

## After posting

Report back with the count posted, the count held back, and the URL of each
comment so the user can find them.
