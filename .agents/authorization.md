# Authorization & Workflow — Bug Report Public Intake

## Purpose

This repo (`Bug-Report-Public`) is a **public intake point only**.
- The public submits bug reports via the GitHub Issue Form
- An automated workflow immediately transfers each report to the **private internal repo**
- The public issue is then closed and locked — the report is **not** lost, just moved

The private repo is where all triage, assignment, resolution, and tracking takes place.

---

## Secrets Required

These must be set in the **public repo's** GitHub Settings → Secrets → Actions:

| Secret Name          | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| `PRIVATE_REPO_TOKEN` | A GitHub Personal Access Token (PAT) with `repo` scope on the private repo |
| `PRIVATE_REPO_OWNER` | The GitHub org or username that owns the private repo (e.g. `SOVEREIGN-NET`)|
| `PRIVATE_REPO_NAME`  | The name of the private repo (e.g. `internal-tracker`)                     |

> **Token scope:** The PAT only needs `repo` access for issue creation on the private repo. It should be a fine-grained token scoped to that specific repo only, with Issues: Read & Write permission.

---

## Roles

| Role         | Repo Access        | Description                                              |
|--------------|--------------------|----------------------------------------------------------|
| `public`     | Public repo only   | Submits bug reports via the issue form. No write access. |
| `triage`     | Private repo       | Reviews incoming `from-public` issues, labels, assigns   |
| `maintainer` | Private repo       | Assigns to devs, sets milestones, escalates              |
| `admin`      | Both repos         | Manages secrets, templates, workflows, team access       |

---

## End-to-End Workflow

```
1. PUBLIC SUBMISSION
   └─ Reporter opens New Issue on Bug-Report-Public
   └─ GitHub presents the YAML form (blank issues disabled)
   └─ Reporter fills in form and submits
   └─ Issue created with labels: bug, triage

2. AUTO-TRANSFER (GitHub Actions: transfer-issue.yml)
   └─ Triggered on: issues.opened WHERE label = bug
   └─ Creates mirror issue in private repo with labels: bug, triage, from-public
   └─ Body includes: original issue number, reporter handle, link back to public issue
   └─ Comments on public issue: confirms receipt, thanks reporter
   └─ Labels public issue: transferred
   └─ Closes public issue (state_reason: not_planned)
   └─ Locks public issue (lock_reason: resolved)

3. TRIAGE (Private repo — triage team)
   └─ Review issue labelled from-public
   └─ Apply severity: critical / high / medium / low
   └─ Request more info via comment if needed (reporter is linked in body)
   └─ Escalate Critical/High to maintainer immediately

4. ASSIGNMENT (Private repo — maintainer)
   └─ Assign to developer or team
   └─ Set milestone / sprint
   └─ Remove triage label, add in-progress

5. RESOLUTION (Private repo — developer)
   └─ Fix implemented, PR linked: "Closes #<issue_number>"
   └─ PR merged → issue auto-closed

6. VERIFICATION (Private repo — triage / maintainer)
   └─ Confirm fix in staging / production
   └─ Label issue: resolved
   └─ Optionally notify original reporter via public issue comment
```

---

## Label Definitions (Private Repo)

| Label          | Meaning                                          |
|----------------|--------------------------------------------------|
| `bug`          | Confirmed bug                                    |
| `triage`       | Awaiting initial review                          |
| `from-public`  | Transferred from the public intake repo          |
| `critical`     | Requires immediate attention                     |
| `high`         | High priority                                    |
| `medium`       | Normal priority                                  |
| `low`          | Low priority / cosmetic                          |
| `needs-info`   | Awaiting more details from reporter              |
| `in-progress`  | Actively being worked on                         |
| `resolved`     | Fix verified and deployed                        |
| `duplicate`    | Already reported                                 |
| `invalid`      | Not a valid bug report                           |
| `wont-fix`     | Acknowledged, will not be addressed              |

---

## Escalation Path

```
public reporter
    └─► triage team     (within 48 hours of transfer)
            └─► maintainer   (Critical/High severity OR >5 days unresolved)
                    └─► admin     (security vulnerability OR systemic failure)
```

---

## Security Notes

- The `PRIVATE_REPO_TOKEN` is never exposed in logs — it is only referenced as a secret
- The public repo's built-in `GITHUB_TOKEN` is used only for commenting/closing the public issue (no private repo access)
- Blank issues are disabled — the public cannot bypass the form
- Public issues are locked after transfer to prevent further input on the closed ticket
