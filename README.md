# Backport Workflow

A **reusable GitHub Actions workflow** that lets maintainers back‑port merged pull requests to a release branch (or a semver‑based tag) by posting a single comment:

```text
/backport <branch-or-tag>
```

> Example: “`/backport release-2025.05`” or “`/backport v1.0.12`”

The workflow lives entirely in this repository and can be **imported with one line** from any other repo in your organisation—keeping maintenance in one place and roll‑outs painless.

---

## ✨ Features

| Capability                                | Details                                                                                                                                                                                                         |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------                                          |
| **Comment trigger**                       | Listens for new `issue_comment` events that start with `/backport `.                                                                                                                                            |
| **Branch *or* tag**                       | Accepts either an existing branch name *or* a tag.                                                                                                                                                              |
| **SemVer bump**                           | If a tag seems to have a patch number, it is automatically incremented (`v1.0.12` → `v1.0.13`) and a branch `backport/v1.0.13` is created from that tag.                                                        |
| **Back‑port PR naming**                   | The PR is opened from `backport/<original‑branch>-<random-hash>` into the target branch.                                                                                                                        |
| **Powered by `kiegroup/git-backporting`** | Relies on the proven cherry‑pick tool to handle conflicts and open the PR.                                                                                                                                      |
| **Reusable workflow**                     | Imported via `uses:` – no copy/paste of YAML into product repos.                                                                                                                                                |
| **Org‑wide updates**                      | Fixes roll out by updating a tag in *one* place.                                                                                                                                                                |

---

## 🚀 Quick start (consumer repository)

Add a minimal stub that forwards the `issue_comment` event:

```yaml
# .github/workflows/backport-on-comment.yml
name: Backport trigger

on:
  issue_comment:
    types: [created]

jobs:
  backport:
    uses: oat-sa/backport-action/.github/workflows/backport.yml@v1

    # Required permissions for the called workflow
    permissions:
      contents: write
      pull-requests: write

    # Pass repository secrets through
    secrets: inherit
```

Push that file to `main`. Done — maintainers can now type `/backport …` on any merged PR.

---

## 🔍 How it works under the hood

1. **Parameter extraction** – Grabs the first token after `/backport `.
2. **Remote lookup** – Uses `git ls-remote` to test for an existing branch or tag.
3. **Tag handling** – If tag patch number is easy to identify, bumps PATCH and creates `release/…`; otherwise mirrors the tag name.
4. **Original branch lookup** – Reads the PR’s `head.ref` via the GitHub API and builds `backport/<head.ref>`.
5. **`kiegroup/git-backporting` action** – Invoked with:

   * `target-branch`: the branch created/detected in step 3.
   * `bp-branch-name`: the branch name built in step 4.
   * `pull-request`: URL of the original PR.
6. **Feedback** – If a new branch was created, the workflow will comment back on the original PR.

---

## 🔑 Required permissions

| Scope                  | Reason                                                             |
| ---------------------- | ------------------------------------------------------------------ |
| `contents: write`      | Push a temporary branch to the repo.                               |
| `pull‑requests: write` | Open the back‑port PR and comment on the original PR.              |
| `issues: write`        | For the comment‑back step.                                  |

**Note:** Reusable workflows do **not** inherit permissions from the caller. The stub shown above grants the minimal rights explicitly.

---

## 🗂 Repository structure

```text
backport-action/
└─ .github/
   └─ workflows/
      └─ backport-on-comment.yml   # ← the reusable workflow
```

No other files are required.
