<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [TODO: `<Project Name>` — pr-management-triage configuration](#todo-project-name--pr-management-triage-configuration)
  - [Identifiers](#identifiers)
  - [Project-specific labels](#project-specific-labels)
  - [Grace windows](#grace-windows)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

<!-- SPDX-License-Identifier: Apache-2.0
     https://www.apache.org/licenses/LICENSE-2.0 -->

# TODO: `<Project Name>` — pr-management-triage configuration

This file is the **per-project configuration** for the
[`pr-management-triage`](../../.claude/skills/pr-management-triage/SKILL.md) skill.
It holds the concrete values for your adopter project.

Copy this file into your own
`<project-config>/pr-management-config.md` and replace every
`<placeholder>` with your project's value. The suggested label
strings and grace-window defaults below are reasonable starting
points — keep them as-is or override with your project's
existing conventions.

## Identifiers

| Key | Value | Used by |
|---|---|---|
| `committers_team` | `<github-org>/<committers-team-slug>` | `classify-and-act.md` row F5b — team-mention detection. Used to recognise PR comments that `@`-mention the project's committers as a maintainer-to-maintainer ping. Example: `apache/airflow-committers`. |
| `area_label_prefix` | `area:` | `classify-and-act.md`, `pr-management-stats` — area-label grouping. Adjust to the prefix your project uses for area labels (e.g. `comp:`, `module:`), or leave blank if your project doesn't group PRs by area. |

## Project-specific labels

Labels the skill applies or watches for. Each row maps a generic
**framework concept** to whatever label string the adopter uses.
If the project doesn't have a given concept, leave the value blank
and the skill will skip that row of decision-table actions.

The labels below are **suggested defaults** — readable English
strings that work for most projects. Override with your project's
existing label names if any are already in use.

| Concept | Suggested label | Notes |
|---|---|---|
| `ready_for_maintainer_review` | `ready for maintainer review` | Applied by the `mark-ready` action; used by `pr-management-code-review` as a default selector. |
| `quality_violations_close` | `closed because of multiple quality violations` | Applied when a PR is closed for failing the project's PR quality criteria after multiple opportunities to fix. |
| `suspicious_changes` | `suspicious changes detected` | Applied to first-time-contributor workflow approvals where the diff looks suspicious (binary blobs, unrelated CI changes, etc.). |
| `work_in_progress` |  | Leave blank if your project doesn't use a dedicated WIP label (the skill relies on draft status instead); fill in the label name if your project does. |

## Grace windows

Tunable thresholds. The defaults below are sized for a project
with **~50–100 open PRs and a triage sweep every 1–2 days**.
Scale them up for projects with lower contributor traffic — less
frequent sweeps imply longer grace windows so the skill doesn't
fire stale-action proposals on PRs the maintainer hasn't had a
chance to look at yet.

| Concept | Default | Project value |
|---|---|---|
| Stale-draft close threshold (triaged) | 7 days | 7 days |
| Stale-draft close threshold (untriaged) | 14 days | 14 days |
| Inactive-open → draft threshold | 28 days | 28 days |
| Stale-review-ping cooldown | 7 days | 7 days |
| Stale-workflow-approval threshold | 28 days | 28 days |
| Stale-Copilot-review threshold | 7 days | 7 days |
