<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [CVE-credit policy for bot / AI accounts](#cve-credit-policy-for-bot--ai-accounts)
  - [Why](#why)
  - [Detection — when does the rule fire?](#detection--when-does-the-rule-fire)
  - [Default behaviour — what the skills do when the rule fires](#default-behaviour--what-the-skills-do-when-the-rule-fires)
  - [Where the rule applies](#where-the-rule-applies)
  - [Where the rule does NOT apply](#where-the-rule-does-not-apply)
  - [Worked examples](#worked-examples)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

<!-- SPDX-License-Identifier: Apache-2.0
     https://www.apache.org/licenses/LICENSE-2.0 -->

# CVE-credit policy for bot / AI accounts

The CVE record's `credits[]` array records *people* who discovered a
vulnerability (`type: "finder"`) or shipped the fix
(`type: "remediation developer"`). When an obvious bot or AI account
appears as the candidate for either role, the default is to **not**
credit them. The skills that populate the *Reporter credited as* and
*Remediation developer* tracker body fields enforce this rule at the
point of extraction, and ask the user before adding the credit when
the rule matches.

This file is the single source of truth for *who counts as a bot* and
*how the skills behave when one is detected*. Skills reference it
instead of duplicating the heuristic.

## Why

* Bot accounts (Dependabot, Renovate, GHSA Probot, GitHub Actions,
  Snyk, Mend, …) act on behalf of an organisation or a piece of
  automation — they are not the *person* who found the bug or fixed
  it. Naming them as `finder` or `remediation developer` in a public
  CVE record misrepresents authorship and pollutes the CNA-feed
  with handles that are not actionable as contributor credit.
* AI / LLM agents that opened a PR or filed a report are tools the
  human used. The human who drove the agent — if there is one
  identifiable in the thread — is the candidate for credit. The
  agent itself never is.
* Forwarding services (`noreply@github.com`,
  `security-alerts@<scanner>.com`, …) sit between the actual
  reporter and us; their address is a relay, not an identity.

## Detection — when does the rule fire?

A handle or email is *obvious bot/AI* when **any** of these match:

1. **GitHub `[bot]` suffix** — handle ends in `[bot]`. The canonical
   GitHub convention (e.g. `dependabot[bot]`, `github-actions[bot]`,
   `renovate[bot]`, `copilot[bot]`, `ghsa-probot[bot]`).
2. **Known-bot / automation-name list** — case-insensitive match on
   the handle's un-bracketed stem **or** on a whole-word occurrence
   in a free-form credit string (e.g. *"discovered by Automated
   Scanner v3"*) against:
   `dependabot`, `renovate`, `snyk-bot`, `snyk`, `copilot`,
   `ghsa-probot`, `github-actions`, `mend-bot`, `mend`, `whitesource`,
   `sonatype-lift`, `lift-bot`, `codecov`, `mergify`, `mergifyio`,
   `allcontributors`, `fossabot`, `imgbotapp`, `pre-commit-ci`,
   `claude`, `chatgpt`, `gpt-bot`, `gpt`, `anthropic`, `openai`,
   `automated`, `automation`, `scanner`, `auto-scanner`,
   `vulnerability-scanner`, `security-scanner`, `sast`, `dast`.
3. **Suffix / contains-pattern handles** — handle matches one of
   these case-insensitive regex patterns:
   * `*-bot`, `*bot` (e.g. `securitybot`, `release-bot`)
   * `*-ai`, `*ai` (e.g. `scan-ai`, `securityai`)
   * `*-agent`, `*agent` (e.g. `triage-agent`)
   * `*-gpt`, `*gpt` (e.g. `secaudit-gpt`)
   * `*scanner*`, `*-scan` (e.g. `securityscanner-7`, `audit-scan`)
   * `*automat*` (e.g. `automated-triage`, `automation-svc`,
     `automaton`)
4. **Service email senders** — the `From:` address local-part
   contains `noreply`, `no-reply`, `donotreply`, or starts with
   `bot@`, `security-alerts@`, `notifications@`,
   `mailer-daemon@` (these are relays, never identities).

The detection is intentionally broad. False positives are cheap (the
user is shown what was skipped and can override with one word —
*"include X"*); false negatives are expensive (a bot lands in a
public CVE record).

## Default behaviour — what the skills do when the rule fires

Each skill that extracts a credit candidate applies these rules at
extraction time:

1. **Skip silently in the data flow.** Do not write the bot's name
   into the *Reporter credited as* or *Remediation developer* body
   field. Leave the field at its current value (typically
   `_No response_` for a fresh tracker, or whatever the prior
   resolved value was for an existing tracker).
2. **Surface the skip in the user-facing proposal.** Include a
   one-line entry in the proposal's "skipped" section of the shape:

   ```text
   skipped credit: <handle> (matches bot policy — ends with [bot])
                                                  ^^^^^ which rule matched
   skipped credit: <handle> (matches bot policy — in known-bot list)
   skipped credit: <handle> (matches bot policy — *-bot suffix pattern)
   skipped credit: <email>  (matches bot policy — noreply service sender)
   ```

   The reason — *which* rule fired — is mandatory; it lets the user
   judge whether the skip is correct.
3. **Honour an explicit override.** The user may override the skip
   for a specific tracker with any of these phrasings (or obvious
   variants):

   * *"include `<handle>` anyway"*
   * *"credit `<handle>` as finder"*
   * *"credit `<handle>` as remediation developer"*
   * *"yes, add `<handle>`"*

   When the user confirms the override, set the appropriate body
   field exactly as the user dictated. Do not auto-extend the
   override to other trackers — overrides are per-tracker.

4. **Draft a clarification reply to the reporter when an email
   thread exists *and* the tracker is in direct-reporter mode.**
   When the candidate would have been credited as the *finder*
   (i.e. the reporter themselves is the bot-looking name), the
   tracker has an inbound `<security-list>` mail thread to reply
   on, **and** the tracker's routing mode (per
   [`docs/security/forwarder-routing-policy.md`](../../docs/security/forwarder-routing-policy.md))
   is *direct-reporter*, propose a **Gmail draft** (not a sent
   message) on the same thread asking whether the bot/AI handle
   is the intended credit or whether there's a human behind it
   who should be credited instead. The draft should be:

   * **Polite and short** — one or two short paragraphs; no
     accusations, no jargon.
   * **Specific** — name the handle that was detected and which
     rule fired (so the reporter sees the same reasoning the
     security team did).
   * **Actionable** — offer two clear paths: *"credit `<handle>`
     as-is"* or *"credit `<human-name>` instead — please reply
     with the preferred attribution"*.
   * **Sent only after explicit user confirmation** per the
     [framework's never-send-without-asking rule](../../AGENTS.md).

   **In via-forwarder mode the standalone clarification draft
   is suppressed.** It is a *credit-acceptance confirmation*
   message (asking the reporter to confirm the AI/bot handle is
   the intended credit, or to accept a different one) — and
   credit-acceptance confirmations are on the
   [forwarder-routing-policy negative list](../../docs/security/forwarder-routing-policy.md#negative-space--do-not-relay).
   The forwarder cannot meaningfully accept a credit on behalf
   of the original reporter, so the message becomes a chase-up
   loop. The bot-credit detection still runs and still keeps
   the bot/AI handle out of the credit field; what is suppressed
   is the dedicated message to confirm the alternative.

   The credit *question* itself (initial ask, *"if the reporter
   has a preferred credit form, please pass it back"*) is **not**
   suppressed — it is folded as a single line into whatever
   milestone draft the via-forwarder lifecycle next produces (the
   Step 7 receipt-of-confirmation, the *Report accepted as
   valid* milestone, the *CVE allocated* notification). The
   credit field stays at `_No response_` (or whatever the
   original report's `Credit:` line yielded after bot filtering)
   until a meaningful answer comes back.

   When the tracker has no inbound mail thread at all (e.g. a
   `security-issue-import-from-pr` tracker — the
   forwarder-routing policy explicitly does **not** apply
   there), skip the draft step — there is no reporter to ask.

   A reusable template for the draft body lives in
   [`<project-config>/canned-responses.md`](../../<project-config>/canned-responses.md);
   if no project-local template exists yet, generate the body
   inline and propose adding it to the canned-responses file as a
   follow-up.

## Where the rule applies

This policy fires at every site where the suite *auto-extracts* a
credit candidate without explicit user instruction:

| Skill | Extraction site |
|---|---|
| [`security-issue-import`](../../.claude/skills/security-issue-import/SKILL.md) | Reporter name from email `From:` header; ASF-relay `Credit:` line |
| [`security-issue-import-from-pr`](../../.claude/skills/security-issue-import-from-pr/SKILL.md) | PR author → *Remediation developer* |
| [`security-issue-import-from-md`](../../.claude/skills/security-issue-import-from-md/SKILL.md) | Reporter / finder name from markdown metadata |
| [`security-issue-sync`](../../.claude/skills/security-issue-sync/SKILL.md) | Reporter credit mined from email replies; PR author auto-append to *Remediation developer* |
| [`security-issue-deduplicate`](../../.claude/skills/security-issue-deduplicate/SKILL.md) | Credit consolidation from two trackers |

The rule does **not** fire when the user *explicitly* types a name
into a credit field, or when a tracker already carries a credit that
a human security-team member set — those are explicit decisions and
the skill respects them.

## Where the rule does NOT apply

* [`generate-cve-json`](generate-cve-json/SKILL.md) stays neutral.
  Whatever is in the tracker's credit fields is what lands in the
  CVE JSON. The filter is upstream, at the skills, so that an
  intentional human override survives JSON regeneration.
* The *PR with the fix* body field on a tracker still records the
  PR even when its author is a bot — the field captures the
  artifact, not the author. Only *Remediation developer* is
  bot-filtered.

## Worked examples

**`security-issue-import-from-pr` import of a Dependabot PR.** The
PR's author is `dependabot[bot]`. The skill skips the
*Remediation developer* assignment and surfaces:
`skipped credit: dependabot[bot] (matches bot policy — ends with
[bot])`. The user can override with *"credit dependabot[bot] as
remediation developer"* if a real human at Dependabot HQ is owed the
credit (unusual but allowed).

**`security-issue-sync` mining a reporter email reply.** The
reporter wrote *"please credit me as claude-bot, I used Claude
Code"*. The skill skips the credit, surfaces `skipped credit:
claude-bot (matches bot policy — *-bot suffix pattern)`, **and
proposes a Gmail draft** on the original report thread asking
*"the credit you suggested (`claude-bot`) reads like an AI/agent
handle — would you prefer we credit you under your name (e.g. the
one on the original report) instead, or is `claude-bot` the
attribution you want?"* The user reviews the draft and approves
the send. If the reporter replies with a human name, sync picks
it up on the next pass.

**`security-issue-import` from a relay address.** The `From:`
header is `security-alerts@scanner.example.com`. The skill skips
*Reporter credited as*, surfaces `skipped credit:
security-alerts@scanner.example.com (matches bot policy — noreply
service sender)`, and prompts the user for the actual reporter
identity (typically findable inside the email body).

**`security-issue-import` of an ASF-relay report with an
automation credit line.** The forwarded body ends with *"This
vulnerability was discovered and reported by Automated Security
Scanner v3 (run by ACME Sec Team)"*. The skill matches both the
`automated` known-name and the `*scanner*` contains-pattern,
skips *Reporter credited as*, surfaces `skipped credit:
"Automated Security Scanner v3" (matches bot policy — known
automation name + *scanner* pattern)`, and routes the
credit-preference question to `@raboof` / Arnout via the
ASF-relay credit-preference flow. The user can override with
*"credit ACME Sec Team as finder"* if the human team behind the
scanner is owed the credit.
