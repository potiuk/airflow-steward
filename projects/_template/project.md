<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [TODO: `<Project Name>` — project manifest](#todo-project-name--project-manifest)
  - [Identity](#identity)
  - [Repositories](#repositories)
  - [Mailing lists](#mailing-lists)
  - [Tools enabled](#tools-enabled)
  - [CVE tooling](#cve-tooling)
  - [GitHub project board](#github-project-board)
  - [Mail sources](#mail-sources)
    - [Backend declaration](#backend-declaration)
    - [Per-backend config](#per-backend-config)
  - [Issue-template fields](#issue-template-fields)
  - [Pointers to sibling files](#pointers-to-sibling-files)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

<!-- SPDX-License-Identifier: Apache-2.0
     https://www.apache.org/licenses/LICENSE-2.0 -->

# TODO: `<Project Name>` — project manifest

This is the **project configuration** for `TODO: <project-name>`.
Every skill under [`../../.claude/skills/`](../../.claude/skills/)
reads the project name from `<project-config>/project.md` and then loads this manifest to resolve project-specific identity,
repositories, mailing lists, and references to the other files in
this directory.

Grep for `TODO` to see every field you still need to fill in:

```bash
grep -n TODO projects/<name>/project.md
```

## Identity

| Key | Value |
|---|---|
| `project_name` | TODO: e.g. `Apache Foo` |
| `vendor` | TODO: e.g. `Apache Software Foundation` |
| `short_name` | TODO: e.g. `Foo` |
| `product_family_url` | TODO: e.g. `https://foo.apache.org/` |

The `vendor` / `project_name` pair is what lands in the `vendor` and
`product` fields of the CVE 5.x record the CVE-JSON generator
produces.

## Repositories

| Key | Value | Purpose |
|---|---|---|
| `tracker_repo` | TODO: e.g. `foo-s/foo-s` | Private security tracker (this repo) |
| `tracker_repo_url` | TODO | |
| `tracker_default_branch` | TODO: e.g. `main` | Default PR target for the tracker repo |
| `tracker_project_board_url` | TODO: URL of the GitHub Project V2 board, if any | Security board |
| `upstream_repo` | TODO: e.g. `apache/foo` | Public codebase where fixes land |
| `upstream_repo_url` | TODO | |
| `upstream_default_branch` | TODO: e.g. `master` (older default) or `main` | Upstream's default branch — what `<default-branch>` resolves to. Distinct from `tracker_default_branch` (the security tracker repo's PR target) |
| `upstream_agents_md_url` | TODO: `https://github.com/<upstream>/blob/main/AGENTS.md` | Conventions this repo mirrors |
| `upstream_contributing_docs_url` | TODO | |
| `upstream_genai_disclosure_anchor` | TODO: URL + anchor for the project's Gen-AI disclosure guideline | |
| `upstream_security_policy_url` | TODO: `https://github.com/<upstream>/security/policy` | |

## Mailing lists

| Key | Value | Notes |
|---|---|---|
| `security_list` | TODO: e.g. `security@foo.apache.org` | Inbound reports; **not** publicly archived |
| `private_list` | TODO: e.g. `private@foo.apache.org` | PMC escalation; **not** publicly archived |
| `users_list` | TODO: e.g. `users@foo.apache.org` | Public advisories end up here; publicly archived |
| `dev_list` | TODO: e.g. `dev@foo.apache.org` | Release `[RESULT][VOTE]` threads; publicly archived |
| `announce_list` | TODO: e.g. `announce@apache.org` | Cross-project announcement list; publicly archived |
| `commits_list` | TODO: e.g. `commits@foo.apache.org` | Publicly archived |
| `asf_security_list` | `security@apache.org` | ASF-wide security team; relays some inbound reports |

**Public archives** typically live at
`https://lists.apache.org/list.html?<list>`. **Private** lists on
`lists.apache.org/thread/<id>` 404 for non-members. Only URLs on
publicly archived lists may appear in CVE `references[]` as
`vendor-advisory`; see `../../AGENTS.md` and
[`security-model.md`](security-model.md).

## Tools enabled

| Capability | Tool | Adapter directory | Config knobs declared here |
|---|---|---|---|
| Issue tracking + source control + project board | `github` | [`../../tools/github/`](../../tools/github/) | `tracker_repo`, `upstream_repo`, `github_project_board_*`, `issue_template_fields` |
| Inbound email / drafts | `<one or more mail-source backends>` | [`../../tools/mail-source/contract.md`](../../tools/mail-source/contract.md) (abstract) + per-backend adapter dirs (`tools/gmail/`, `tools/ponymail/`, `tools/mail-source/imap/`, `tools/mail-source/mbox/`, ...) | See [Mail sources](#mail-sources) below — declare each backend's role (primary / preferred-for-`<op>` / fallback / optional) and `mandatory` flag |
| CVE allocation + record mgmt | `vulnogram` | [`../../tools/vulnogram/`](../../tools/vulnogram/) | see [CVE tooling](#cve-tooling) below |
| Release voting / announce | TODO: ASF mailing lists — or replace with the project's release-comms backend | — | via `dev_list` / `announce_list` / `users_list` |

To replace a tool (e.g. swap GitHub issues for JIRA), declare an
alternate tool in the table above, add a `tools/<name>/` adapter
directory, and make sure the values the generic skills need are still
reachable from this manifest.

## CVE tooling

TODO: describe which CNA tool the project uses. For ASF projects the
default is ASF's Vulnogram; other CNAs will substitute their own
equivalents. The Vulnogram-side mechanics live under
[`../../tools/vulnogram/`](../../tools/vulnogram/); the per-project
values below are what the generic recipes substitute in.

| Key | Value |
|---|---|
| `cve_tool` | TODO: e.g. `vulnogram` (ASF-hosted) |
| `cve_tool_allocate_url` | TODO: e.g. `https://cveprocess.apache.org/allocatecve` |
| `cve_tool_record_url_template` | TODO: e.g. `https://cveprocess.apache.org/cve5/<CVE-ID>` |
| `cve_tool_source_tab_url_template` | TODO |
| `cve_allocation_gated_by` | TODO: e.g. `Foo PMC membership (ASF OAuth)` |
| `asf_org_id` | TODO: project's CNA org UUID (for ASF projects: `f0158376-9dc2-43b6-827c-5f631a4d8d09`) |
| `cna_private_owner` | TODO: e.g. `foo` (CNA_private.owner — identifies the project slug inside the ASF CNA queue) |
| `cna_private_projecturl` | TODO: e.g. `https://foo.apache.org/` |
| `cna_private_userslist` | TODO: e.g. `users@foo.apache.org` |

## GitHub project board

If the project uses a Projects V2 board for its security-issue view,
declare the node IDs below. Fetch with the introspection query in
[`../../tools/github/project-board.md`](../../tools/github/project-board.md#introspection--re-fetch-the-option-ids).
If the project does not run a board, leave the table blank — skills
treat missing board config as *"no board reconciliation"*.

| Key | Value |
|---|---|
| `project_board_url` | TODO |
| `project_board_number` | TODO |
| `project_board_node_id` | TODO |
| `status_field_node_id` | TODO |

**`Status` column → option-ID mapping** (re-fetch if any write
returns `not found`):

| Column | Option ID |
|---|---|
| `Needs triage` | TODO |
| `Assessed` | TODO |
| `CVE allocated` | TODO |
| `PR created` | TODO |
| `PR merged` | TODO |
| `Fix released` | TODO |
| `Announced` | TODO |

## Mail sources

The skills treat every supported mail backend the same way —
through the abstract operations defined in
[`../../tools/mail-source/contract.md`](../../tools/mail-source/contract.md).
The adopter declares which backends are configured, what *role*
each plays, and whether any are *mandatory*. The skill's resolution
rule (see the contract) then picks the right backend per operation
at run time.

### Backend declaration

One row per configured backend. **Exactly one** row carries
`role: primary`. Multiple rows may carry `preferred for <op>` to
override the primary for specific operations. `fallback` rows are
tried in order when no preferred / primary backend supports the op.
`mandatory: yes` means the skill **refuses to run** when that
backend is unavailable; `no` means the skill continues with the
remaining backends (and skips ops that no available backend supports).

| Backend | Role | Mandatory | Notes |
|---|---|---|---|
| TODO: `gmail` | TODO: e.g. `primary` | TODO: `yes` / `no` | TODO: e.g. "Triager Gmail account subscribed to `<security-list>` and `<private-list>`" |
| TODO: `ponymail` | TODO: e.g. `fallback` or `preferred for thread_url` | TODO | TODO: e.g. "Read-only archive backstop; PMC LDAP session required for private-list reads" |
| TODO: *(add more rows as needed — `imap`, `mbox`, project-specific adapter)* | | | |

Reference adapter docs:
[`tools/gmail/tool.md`](../../tools/gmail/tool.md) (full read+write),
[`tools/ponymail/tool.md`](../../tools/ponymail/tool.md) (read-only ASF archive),
[`tools/mail-source/imap/README.md`](../../tools/mail-source/imap/README.md) (stub),
[`tools/mail-source/mbox/README.md`](../../tools/mail-source/mbox/README.md) (read-only offline archive — stub).

### Per-backend config

Per-backend values the generic recipes substitute in. Only fill in
the rows for backends declared above; leave the rest blank or
remove the row.

| Key | Backend | Value |
|---|---|---|
| `security_list_domain` | `gmail` | TODO: e.g. `security.foo.apache.org` — Gmail `list:` operator uses the domain form |
| `ponymail_private_search_url_template` | `ponymail` | TODO |
| `ponymail_public_search_url_template` | `ponymail` | TODO |
| `ponymail_api_url_template` | `ponymail` | TODO |
| `ponymail_thread_url_template` | `ponymail` | `https://lists.apache.org/thread/<hash>?<list>` |
| `imap_host` | `imap` | TODO: e.g. `imap.example.org` |
| `imap_account` | `imap` | TODO: e.g. `security-triage@example.org` |
| `imap_security_list_folder` | `imap` | TODO: e.g. `INBOX.security-list` |
| `imap_drafts_folder` | `imap` | TODO: e.g. `Drafts` (or leave blank to declare `create_draft` unsupported on this adapter) |
| `mbox_archive_path` | `mbox` | TODO: e.g. `/srv/audit/security-list-2024.mbox` |

## Issue-template fields

The skills' body-field roles map to the following concrete `###`
headings in the project's issue template (the concrete YAML file lives in the
adopter's `<upstream>` repo; the generic role → field contract is in The generic role → GitHub-field
contract lives in
[`../../tools/github/issue-template.md`](../../tools/github/issue-template.md);
the concrete names below are what skills read and write for this
project.

| Role (generic) | Field name | Template type | Required? |
|---|---|---|---|
| `issue-description` | TODO | `textarea` | TODO |
| `public-summary` | TODO | `textarea` | TODO |
| `affected-versions` | TODO | `input` | TODO |
| `security-thread` | TODO | `input` | TODO |
| `public-advisory-url` | TODO | `input` | TODO |
| `reporter-credit` | TODO | `input` | TODO |
| `pr-with-fix` | TODO | `input` | TODO |
| `cwe` | TODO | `input` | TODO |
| `severity` | TODO | `dropdown` | TODO |
| `cve-tool-link` | TODO | `input` | TODO |

## Pointers to sibling files

- [`release-trains.md`](release-trains.md) — fast-moving release state, release-manager attribution, security-team roster.
- [`milestones.md`](milestones.md) — milestone naming conventions.
- [`scope-labels.md`](scope-labels.md) — scope label → CVE product mapping.
- [`security-model.md`](security-model.md) — Security-Model URL + anchors.
- [`title-normalization.md`](title-normalization.md) — CVE title strip cascade.
- [`fix-workflow.md`](fix-workflow.md) — fork / toolchain / commit-trailer specifics.
- [`naming-conventions.md`](naming-conventions.md) — project-specific editorial rules.
- [`canned-responses.md`](canned-responses.md) — reporter-facing reply templates.
- [`README.md`](README.md) — project file index + onboarding checklist.
