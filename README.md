# Bug-Report-Public

Public bug intake for the Sovereign Network. Community members submit bugs here via the issue form; an automated workflow transfers each report to the private `The-Sovereign-Network` tracker and adds it to the project board.

---

## One-Time Setup (Required)

Before the transfer workflow will work, a repository admin must add **one secret** in the repository settings:

1. Go to **Settings → Secrets and variables → Actions → New repository secret**
2. Create a secret named exactly **`NEW_SECRET`**
3. Set its value to a **Personal Access Token (classic)** or a fine-grained PAT that has **`repo` scope** (write access) on the `SOVEREIGN-NET/The-Sovereign-Network` repository and **`read:project` + `project` scope** to add items to the org project board

That's it. The workflow uses `GITHUB_TOKEN` (automatic) for all actions on *this* public repo and `NEW_SECRET` only for cross-repo operations.

---

## How It Works

1. A contributor opens an issue using the **Bug Report** template — the template automatically applies the `bug` label
2. The `labeled` event fires the **Transfer Bug Report to Sovereign Network** workflow
3. The workflow:
   - Creates a copy of the issue in `The-Sovereign-Network` with `bug`, `triage`, and `from-public` labels
   - Adds the new issue to Project board #5
   - Posts a thank-you comment on the public issue
   - Closes and locks the public issue to keep this repo clean