# Branch Protection Setup

This guide explains how to configure GitHub branch protection rules for EWB project repositories. Apply these rules to both `main` and `develop`.

---

## Why Branch Protection Matters

Without branch protection:
- Team members can push directly to `main`, bypassing code review and the CI pipeline
- Force pushes can overwrite and lose committed work
- Branches can be accidentally deleted

Branch protection enforces the pull request workflow and prevents these scenarios.

---

## Configuring Branch Protection Rules

### Step 1 — Open Branch Protection Settings

1. Go to your repository on GitHub
2. Click **Settings**
3. In the left sidebar, click **Branches**
4. Under **Branch protection rules**, click **Add rule**

---

### Step 2 — Set the Branch Name Pattern

Enter the branch name pattern:

```
main
```

Repeat this process for `develop`.

---

### Step 3 — Enable Required Rules

Configure the following settings:

#### Require a pull request before merging

- [x] **Require a pull request before merging**
  - **Required approvals:** `1`
  - [x] Dismiss stale pull request approvals when new commits are pushed
  - [x] Require review from code owners (if your project has a `CODEOWNERS` file)

#### Require status checks to pass before merging

- [x] **Require status checks to pass before merging**
  - [x] Require branches to be up to date before merging
  - In the search box, find and add the name of your QA pipeline job. The job name appears in the GitHub Actions UI (e.g. `React QA`, `Node.js QA`, `.NET QA`, `Java QA`)

#### Additional protection settings

- [x] **Do not allow bypassing the above settings**
- [x] **Restrict pushes that create matching branches** (optional — prevents creating branches directly named `main`)

#### Under "Rules"

- [x] **Block force pushes**
- [x] **Restrict deletions**

---

### Step 4 — Save the Rule

Click **Create** or **Save changes**.

---

## Confirming the Rules Are Active

1. Open a pull request against the protected branch
2. Confirm the following appear under **Checks**:
   - A check for the QA pipeline job (e.g. `React QA`)
   - A merge-blocking message if the check has not passed
3. Confirm the pull request cannot be merged without an approving review

---

## Recommended Settings Summary

| Setting | `main` | `develop` |
|---|---|---|
| Require PR before merging | Yes | Yes |
| Required approvals | 1 | 1 |
| Dismiss stale reviews | Yes | Yes |
| Require status checks | Yes (QA job) | Yes (QA job) |
| Require up to date branch | Yes | Yes |
| Block force pushes | Yes | Yes |
| Restrict deletions | Yes | Yes |
| Allow bypass | No | No |

---

## CODEOWNERS (Optional)

To require review from specific individuals or teams based on file paths, add a `CODEOWNERS` file to the repository root or `.github/` directory:

```
# .github/CODEOWNERS

# All files — require review from the development team
*  @EWBHoldings/dev-team

# Infrastructure and pipeline files — require DevOps review
.github/  @EWBHoldings/devops-team
```

Enable **Require review from code owners** in the branch protection rule for this to take effect.
