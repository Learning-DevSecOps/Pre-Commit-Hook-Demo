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
├── secret_test.py               # Demo file used in the walkthrough
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
git clone https://github.com/Learning-DevSecOps/precommit-gitleaks-demo.git
cd precommit-gitleaks-demo
```

### Step 3 — Activate the hooks

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
      - id: check-merge-conflict
      - id: detect-private-key
```

Gitleaks is pinned to `v8.30.0` — the version is locked in source control so every developer on the team runs the exact same scanner.

---

## 🧪 Demo Walkthrough

### 🔴 Trigger — Commit Blocked by a Detected Secret

Write a fake Stripe secret key into `secret_test.py` using the command for your platform:

**macOS / Linux**
```bash
echo 'STRIPE_SECRET_KEY=sk_live_4xKqP9mN2rL7vB8wE3jH6tY1' > secret_test.py
```

**Windows CMD**
```cmd
echo STRIPE_SECRET_KEY=sk_live_4xKqP9mN2rL7vB8wE3jH6tY1 > secret_test.py
```

**Windows PowerShell**
```powershell
Set-Content secret_test.py 'STRIPE_SECRET_KEY=sk_live_4xKqP9mN2rL7vB8wE3jH6tY1'
```

> 💡 **Easiest cross-platform option:** Open `secret_test.py` in any editor, paste the line below, and save.
> ```
> STRIPE_SECRET_KEY=sk_live_4xKqP9mN2rL7vB8wE3jH6tY1
> ```

Now stage and commit:

```bash
git add secret_test.py
git commit -m "test"
```

Expected output — the commit is **blocked**:

```
Detect hardcoded secrets................................................Failed
- hook id: gitleaks
- exit code: 1

    Finding:     STRIPE_SECRET_KEY=REDACTED
    Secret:      REDACTED
    RuleID:      stripe-access-token
    Entropy:     4.875000
    File:        secret_test.py
    Line:        1
    Fingerprint: secret_test.py:stripe-access-token:1

10:05PM WRN leaks found: 1
```

The secret is redacted in the output and the commit never reaches the repository. Restore the file:

```bash
git restore secret_test.py
```

---

### 🟢 Pass — Clean Commit Goes Through

Stage the restored (clean) file and commit:

```bash
git add secret_test.py
git commit -m "clean commit"
```

Expected output — all hooks pass:

```
Detect hardcoded secrets................................................Passed
trim trailing whitespace................................................Passed
fix end of files........................................................Passed
check yaml...........................................(no files to check)Skipped
check for added large files.............................................Passed
check for merge conflicts...............................................Passed
detect private key......................................................Passed
[main 3f2a1b9] clean commit
```

---

## ⚠️ Why AWS and GitHub Example Keys Don't Trigger Gitleaks

You might be tempted to use the well-known AWS example key from documentation:

```
# These will NOT trigger Gitleaks — they are internally allowlisted
AWS_SECRET=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY    ❌
GITHUB_TOKEN=ghp_aBcDeFgHiJkLmNoPqRsTuVwXyZ123456789    ❌
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE                   ❌
```

Gitleaks v8.30.0 has these exact strings on an **internal allowlist** because they appear across thousands of official documentation pages and tutorials. Scanning them would generate noise on every repo that has a README.

For a reliable demo, always use a **realistic-looking key with random characters** that matches a known secret format but isn't a documented example string:

```
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxx    ✅ Blocked
```

Gitleaks catches this because it matches the `sk_live_` prefix pattern, has high entropy (4.875), and is not on any allowlist.

---

## ⚠️ Common Pitfall — Secret Inside Quotes (Windows)

On Windows, `echo '...' > secret_test.py` writes the single quotes as part of the file content:

```
'STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxx'   ❌ Gitleaks misses this
STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxx     ✅ Gitleaks catches this
```

Always verify with `type secret_test.py` (Windows) or `cat secret_test.py` (macOS/Linux) before staging to confirm there are no surrounding quotes.

---

## 🔁 Onboarding a New Developer

Every new contributor needs just **one command** after cloning:

```bash
pre-commit install
```

No manual hook setup. No documentation to follow precisely. The `.pre-commit-config.yaml` in this repo ensures everyone runs identical checks, at the same pinned version, every time.

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
