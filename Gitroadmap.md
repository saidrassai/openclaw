# 🧠 Senior DevOps / GitOps Git Handbook (A → Z)

> **Audience**: Senior DevOps, Platform Engineers, SREs, GitOps Practitioners
>
> **Goal**: Production-grade Git mastery — clean history, safe workflows, automation-ready

---

## 1️⃣ Git Mental Model (You MUST internalize this)

Git is **not** a file system.
Git is **a directed acyclic graph (DAG) of commits**.

- **Commit** = snapshot + parent pointer
- **Branch** = movable pointer to a commit
- **HEAD** = where you currently are
- **Remote** = another reference namespace

```text
HEAD → branch → commit → parent → parent
```

Everything else is manipulation of pointers.

---

## 2️⃣ Repository Types (Know what you are in)

| Type | Description | Risk |
|----|----|----|
| Monorepo | Many services | Medium–High |
| Polyrepo | One service per repo | Low |
| Forked OSS | Upstream + Origin | Medium |
| Infra repo | Terraform / Helm / Argo | **Critical** |

---

## 3️⃣ Initial Setup (Mandatory for seniors)

### Global config
```bash
git config --global user.name "Your Name"
git config --global user.email "you@company.com"

git config --global pull.rebase true
git config --global fetch.prune true
git config --global init.defaultBranch main
```

### SSH (never use HTTPS in production)
```bash
ssh-keygen -t ed25519 -C "you@github"
ssh -T git@github.com
```

### Signed commits + audit trail
```bash
git config --global commit.gpgsign true
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
```

---

## 4️⃣ Repository Cloning Strategies

### Fork-based (OSS)
```bash
git clone git@github.com:you/project.git
cd project
git remote add upstream git@github.com:org/project.git
```

### Internal repo
```bash
git clone git@github.com:company/project.git
```

---

## 5️⃣ Branching Strategies (Choose ONE)

### ✅ GitHub Flow (recommended)
- main = always deployable
- short-lived feature branches

### GitFlow (legacy)
- develop, release, hotfix
- **Avoid for GitOps**

### Trunk-based (advanced)
- main only
- feature flags

---

## 6️⃣ Golden Rules (Non-negotiable)

❌ Never commit directly to `main`
❌ Never rewrite shared history
❌ Never force-push `main`

✅ Always branch
✅ Always review
✅ Always rebase before merge
✅ Protect `main` with required checks and reviews
✅ Require signed commits on protected branches

---

## 7️⃣ Daily Development Workflow

```bash
git checkout main
git pull upstream main

git checkout -b feature/<name>
```

Work → commit → push:
```bash
git add .
git commit -m "feat: clear message"
git push origin feature/<name>
```

---

## 8️⃣ Commit Message Standard (Conventional Commits)

```text
feat: add autoscaling logic
fix: resolve race condition
refactor: simplify agent pipeline
docs: update architecture
chore: bump dependencies
```

Why it matters:
- Auto changelogs
- Semantic versioning
- CI automation

---

## 9️⃣ Rebase vs Merge (SENIOR KNOWLEDGE)

### Rebase (preferred)
```bash
git fetch upstream
git rebase upstream/main
```

✔ Linear history
✔ Clean blame
❌ Requires discipline

### Merge
```bash
git merge upstream/main
```

✔ Safe
❌ Messy history

---

## 🔟 Conflict Resolution (Real-world)

```bash
git status
git diff
```

Fix → stage → continue:
```bash
git add <file>
git rebase --continue
```

Abort if needed:
```bash
git rebase --abort
```

---

## 1️⃣1️⃣ Reset vs Revert (CRITICAL)

### Reset (local only)
```bash
git reset --soft HEAD~1
git reset --hard HEAD~1
```

### Revert (shared history)
```bash
git revert <commit>
```

👉 **In GitOps: ONLY revert**

---

## 1️⃣2️⃣ Tags & Releases

```bash
git tag -a v1.2.0 -m "release"
git push origin v1.2.0
```

Use for:
- Releases
- Rollbacks
- Audits

---

## 1️⃣3️⃣ GitOps Core Principles

> Git is the **single source of truth**

- No manual prod changes
- Infra = code
- State reconciliation

Tools:
- ArgoCD
- Flux
- Terraform
- Helm

---

## 1️⃣4️⃣ GitOps Repo Structure

```text
infra/
  clusters/
  apps/
  helm/
  terraform/
```

One PR = one intent.

### Environment promotion (preferred)
```text
dev/ -> staging/ -> prod/
```
Promote by PR (no manual kubectl).

---

## 1️⃣5️⃣ Rollback Strategy (MANDATORY)

Rollback = revert commit

```bash
git revert <bad-commit>
git push origin main
```

GitOps controller redeploys automatically.

---

## 1️⃣6️⃣ Audit & Compliance

```bash
git log --oneline --decorate
git blame <file>
```

Why auditors love GitOps:
- Immutable history
- Clear ownership
- Traceability

---

## 1️⃣7️⃣ Advanced Inspection

```bash
git reflog
git show <commit>
git bisect start
```

---

## 1️⃣8️⃣ Submodules (use sparingly)

```bash
git submodule add <repo>
git submodule update --init --recursive
```

Alternative: mono-repo or package registry.

---

## 1️⃣9️⃣ Hooks (Local Quality Gates)

```bash
.git/hooks/pre-commit
```

Use for:
- lint
- tests
- secrets scan

---

## 2️⃣0️⃣ CI/CD + Git

Git triggers pipelines:
- push
- PR
- tag

Never deploy from CI without Git state.

### Required CI gates (minimum)
- Lint + typecheck
- Unit tests
- Security scan (SAST + secrets)
- IaC scan (tfsec/checkov)
- Policy checks (OPA/Conftest where applicable)

---

## 2️⃣1️⃣ Security Best Practices

❌ No secrets in Git

Use:
- SOPS
- Vault
- Sealed Secrets

Enforce:
- CODEOWNERS for infra paths
- branch protections + required reviews
- least-privilege deploy tokens

Scan:
```bash
git secrets --scan
```

---

## 2️⃣2️⃣ Force Push (WHEN allowed)

```bash
git push --force-with-lease
```

✔ Safer than `--force`
❌ Never on main

---

## 2️⃣3️⃣ Cleanup & Maintenance

```bash
git branch -d <branch>
git fetch --prune
git gc
```

---

## 2️⃣4️⃣ Disaster Recovery

```bash
git reflog
git checkout <hash>
```

Git remembers **everything**.

---

## 2️⃣5️⃣ Senior Mindset

- Git is a production system
- History is sacred
- Automation reads Git
- Humans review intent

---

## ✅ Final Checklist

- [ ] SSH only
- [ ] No main commits
- [ ] Rebase before merge
- [ ] One PR = one change
- [ ] Git is source of truth
- [ ] Signed commits enabled
- [ ] Branch protections on
- [ ] CI gates enforced

---

# 🔧 Senior GitOps Addendum (Production-grade)

## A) Branch protection policy (baseline)
- Require PRs and 1–2 approvals
- Require status checks (lint, tests, security)
- Require signed commits
- Require linear history (no merge commits)
- Block force pushes to `main`

## B) Change management discipline
- One PR = one intent
- Changes are reversible (revert > reset)
- Every change links to an issue or change request

## C) GitOps layout patterns
```text
infra/
  clusters/
    dev/
    staging/
    prod/
  apps/
  policies/
  templates/
```
Keep environment-specific values isolated; reuse shared base manifests.

## D) Secrets management
- Store encrypted secrets in Git (SOPS) or external secret stores
- Rotate keys on environment boundaries
- Never store raw creds in CI logs

## E) Promotion workflow (immutable)
1) Merge to `dev`
2) Automated deploy to dev
3) Promote by PR to `staging`
4) Promote by PR to `prod`

## F) Audit-ready releases
- Tag releases (`vYYYY.M.D`)
- Attach build metadata (commit SHA, SBOM)
- Keep deployment manifests in Git history

---

