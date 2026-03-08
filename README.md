# 🔒 pre-commit + Gitleaks Demo

> **Companion repository for DevSecOps Series — Article 2**
> *Hardening Your Git Workflow: .gitignore, Gitleaks, and Pipeline Secret Scanning*

This repository demonstrates **Option B: The pre-commit Framework** — using the [pre-commit framework](https://pre-commit.com) to run [Gitleaks](https://gitleaks.io) as a version-controlled, team-wide Git hook. Secrets are caught on your machine before they ever reach the remote.

---

## 💡 Why the pre-commit Framework Over a Raw Hook?

Writing directly to `.git/hooks/` has one major drawback — those files aren't committed to the repository, so each developer manages their own. Standards drift, and coverage becomes inconsistent across the team.

The pre-commit framework solves this by centralising all hook definitions in a YAML file that **does** live in version control. Any developer who clones the project and runs `pre-commit install` automatically inherits the same protections — one command, consistent security posture across the whole team.

---

## 📁 Repository Structure

```
.
├── .gitignore                   # Excludes .env, keys, credentials, logs
├── .env.example                 # Safe placeholder template — commit this, never .env
├── .pre-commit-config.yaml      # Hook definitions (Gitleaks + housekeeping checks)
├── .gitleaks.toml               # Custom rules and false-positive suppression
├── test.txt                     # Demo file used in the walkthrough
└── README.md
```

---

## ✅ Prerequisites

- Git
- Python 3.8+ **or** Homebrew (macOS)
- [Gitleaks v8.30.0](https://github.com/gitleaks/gitleaks/releases/tag/v8.30.0)

Install Gitleaks first:

```bash
# macOS
brew install gitleaks

# Linux (Debian / Ubuntu)
sudo apt install gitleaks

# Linux (Red Hat / Fedora)
sudo dnf install gitleaks

# Windows (Chocolatey)
choco install gitleaks

# Verify
gitleaks version   # Expected: v8.30.0
```

---

## 🚀 Setup

### Step 1 — Install the pre-commit framework

```bash
brew install pre-commit  # macOS
pip install pre-commit   # cross-platform
```

### Step 2 — Clone this repo

```bash
git clone https://github.com/your-username/precommit-gitleaks-demo.git
cd precommit-gitleaks-demo
```

### Step 3 — Activate and verify the hooks

```bash
pre-commit install
# pre-commit installed at .git/hooks/pre-commit
```

Done. Every `git commit` on this machine now runs Gitleaks automatically against staged files.

---

## ⚙️ How the Hooks Are Configured

The hook definitions live in `.pre-commit-config.yaml` at the repository root:

```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.30.0
    hooks:
      - id: gitleaks

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
```

Gitleaks is pinned to `v8.30.0` — the version is locked in source control so every developer on the team runs the exact same scanner.

---

## 🧪 Demo Walkthrough

### 🔴 Trigger — Commit Blocked by a Detected Secret

Deliberately stage a file containing a fake AWS secret:

```bash
echo 'AWS_SECRET=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY' >> test.txt
git add test.txt && git commit -m "test"
```

Expected output — the commit is **blocked**:

```
Detect hardcoded secrets................................................Failed
- hook id: gitleaks
- exit code: 1

    Finding:     AWS_SECRET=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    RuleID:      generic-api-key
    Entropy:     4.712
    File:        test.txt
    Line:        1
```

The secret never reaches the repository. Clean up the test file:

```bash
git restore test.txt
```

---

### 🟢 Pass — Clean Commit Goes Through

Stage the restored (clean) file and commit:

```bash
git add test.txt && git commit -m "clean commit"
```

Expected output — all hooks pass:

```
Detect hardcoded secrets................................................Passed
Fix trailing whitespace.................................................Passed
Fix End of Lines........................................................Passed
Check Yaml..............................................................Passed
Check for added large files.............................................Passed
[main 3f2a1b9] clean commit
```

---

## 🔁 Onboarding a New Developer

Every new contributor needs just **one command** after cloning:

```bash
pre-commit install
```

No manual hook setup. No documentation to follow precisely. The `.pre-commit-config.yaml` committed to this repo ensures everyone runs identical checks, at the same version, every time.

---

## 📖 Read the Full Article

This repository is a hands-on companion to the full DevSecOps guide, which covers:

- Setting up a security-first `.gitignore`
- Scanning your full commit history with Gitleaks
- Running Gitleaks in CI/CD pipelines (GitHub Actions, GitLab CI, Azure DevOps, Bitbucket, Jenkins)
- Responding when a secret gets through

👉 [Read Article 2 on Medium](#) *(replace with your Medium article URL)*

---

## 📚 References

- [Gitleaks](https://gitleaks.io) — github.com/gitleaks/gitleaks
- [pre-commit Framework](https://pre-commit.com)
- [pre-commit-hooks](https://github.com/pre-commit/pre-commit-hooks)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)

---

**Tags:** `devsecops` `gitleaks` `pre-commit` `secret-scanning` `git-hooks` `security`
