# Research Agent with Evidence

A Python research CLI that compares direct search with query planning. Both
strategies create a Vietnamese brief and retain the source excerpts used to
write it. A Python check requires each claim's quoted evidence to appear
verbatim in the cited excerpt.

## Run on Windows

Python 3.11 is enough. There are no extra packages to install.

```powershell
python research.py
Get-Content reports/demo/comparison.json -Encoding utf8
Get-Content reports/demo/planned/brief.md -Encoding utf8
```

The offline example searches a synthetic corpus about a fictional recycling
program. Its query plan and source text are prepared in advance. The example
uses text taken directly from the retrieved documents as claims, with no
model call. Topic coverage counts predefined source IDs represented in the
brief. The result demonstrates report behavior on this designed fixture.

## Research a topic with Wikipedia and Groq

The program sends queries to the selected Wikipedia language edition and
sends the topic and up to three article lead excerpts to Groq. Use a public
research topic. The default Groq model is `openai/gpt-oss-120b`.

```powershell
$researchKey = Read-Host "Groq API key" -AsSecureString
$env:GROQ_API_KEY = [System.Net.NetworkCredential]::new('', $researchKey).Password
python research.py --mode live --strategy compare --language en --topic "Precision and recall in information retrieval"
Get-Content reports/live/comparison.json -Encoding utf8
Get-Content reports/live/planned/brief.md -Encoding utf8
Remove-Item Env:GROQ_API_KEY
```

Use `--strategy direct` or `--strategy planned` to run just one approach.
Use `--output reports/another-topic` to keep runs separate. If either provider
returns an error, the command exits with code 2. A strategy completed before
the error retains its own report.

## Compare the strategies

Direct search sends the topic as one query. Planned search asks Groq for up to
three queries and takes search results in round-robin order. Both strategies
pass at most three unique excerpts to the same synthesis model. A complete
comparison makes up to four Wikipedia requests and three Groq requests.
The reports record actual calls, elapsed time and token usage reported by Groq.
The strategies have the same source limit and different request budgets.

1. Decide which aspects the brief should cover before reading model output.
2. Read each claim and its quote. Mark whether the quote supports the full claim.
3. Score topic coverage and count unsupported claims. Compare the two reports
   alongside the number of requests and reported token usage.
4. Note any disagreements or open questions. The command records those reported
   by the model without independently confirming them.

`report.json` contains the exact excerpts, source IDs, URLs, retrieval times,
and rejected claims. `brief.md` is the human-readable output. When publishing,
use short excerpts with attribution and link to the source.

## Live pilot results

Both strategies researched **precision and recall in information retrieval**
using English Wikipedia and `openai/gpt-oss-120b`. Each brief used three unique
source excerpts.

| Strategy | Search calls | Groq calls | Reported tokens | Claims kept | Claims rejected |
| --- | ---: | ---: | ---: | ---: | ---: |
| Direct | 1 | 1 | 2,230 | 6 | 0 |
| Planned | 3 | 2 | 2,644 | 5 | 1 |

The planned run spent more requests and tokens. One proposed claim failed the
verbatim quote check and was excluded from its brief. Counts describe the
output of that check on this topic. Semantic support and topic coverage are
assessed separately by reading the brief and its sources.

## Scope

Live search covers Wikipedia lead sections in one language edition, not the
whole web. Wikipedia is a starting point for finding primary sources. An exact
quote check establishes that the text was retrieved. Human review is needed
to assess whether it supports the claim and whether important context is missing.
The model can miss a contradiction. The program saves no independent truth score.
