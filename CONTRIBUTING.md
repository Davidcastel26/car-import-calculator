# Contributing to auto-market

Thank you for helping build a transparent tool for vehicle import cost calculation in Guatemala. This project maintains the highest standards for code integrity through the **Developer Certificate of Origin (DCO)**.

## Developer Certificate of Origin

By contributing to this project, you certify that:
1. The code is your original work or you have the right to submit it
2. You understand the implications of the DCO
3. You agree to the Apache 2.0 license terms

**All commits must be signed with the `--signoff` flag.** This is a hard requirement enforced by Probot at the GitHub level.

---

## Quick Start

### 1. Configure Git (One-Time Setup)

```bash
# Set your identity
git config --global user.name "Your Full Name"
git config --global user.email "your.email@example.com"

# Optional: Create an alias for convenience
git config --global alias.cc "commit --signoff"
```

### 2. Commit with Signoff

**Using `--signoff`:**
```bash
git commit --signoff -m "Your commit message"
# Short form: git commit -S -m "message"
```

**Using the alias (if configured):**
```bash
git cc -m "Your commit message"
```

### 3. Verify Your Signature

```bash
git log --oneline -1
# Output should show:
# abc1234 Your commit message
#     Signed-off-by: Your Full Name <your.email@example.com>
```

---

## Monorepo Structure

```
/auto-market
  ├── apps/
  │   ├── web/          (Frontend React application)
  │   └── api/          (Backend/Gateway service)
  ├── packages/
  │   ├── database/     (Shared Prisma/TypeORM logic)
  │   ├── shared-types/ (TypeScript interfaces & DTOs)
  │   └── ui-config/    (Tailwind & ESLint configs)
  ├── .github/
  │   ├── workflows/    (CI/CD pipelines)
  │   └── dco.yml       (Probot configuration)
  ├── CONTRIBUTING.md   (This file)
  ├── LICENSE           (Apache 2.0)
  └── package.json      (Workspace configuration)
```

**DCO applies project-wide.** Whether you commit to `apps/api` or `packages/shared-types`, the same signature requirement applies.

---

## Typical Workflow

### Creating a Feature Branch

```bash
git checkout -b feat/ipr-calculation

# Make your changes...
echo "IPR logic" >> packages/shared-types/index.ts

# Commit with signoff
git commit --signoff -m "Add IPR depreciation calculation logic"

# Push to GitHub
git push origin feat/ipr-calculation

# Create a Pull Request on GitHub
# Probot automatically checks: ✓ DCO Valid
# Your PR is ready for review and merge
```

### If Probot Blocks Your PR

Probot will leave a comment like:
```
❌ DCO Check Failed
Commit abc1234 is missing a valid Signed-off-by trailer.
```

**Fix it with:**
```bash
git commit --amend --signoff
git push --force-with-lease origin your-branch

# Probot will automatically re-check and pass ✓
```

---

## Troubleshooting

### "I forgot to sign my commits"

**For the most recent commit:**
```bash
git commit --amend --signoff
git push --force-with-lease origin your-branch
```

**For multiple commits:**
```bash
git rebase -i HEAD~3    # Adjust the number to match your commits
# Mark commits as 'reword'
# Amend each with --signoff when prompted
git push --force-with-lease origin your-branch
```

### "I'm using the wrong name/email"

```bash
# Check your current settings
git config user.name
git config user.email

# Fix it
git config --global user.name "Correct Name"
git config --global user.email "correct@email.com"

# Amend recent commits
git commit --amend --signoff
git push --force-with-lease origin your-branch
```

### "What if multiple people work on the same branch?"

Each developer signs their own commits with their own identity. Coordinating via pull requests ensures proper attribution.

---

## For Team Members

### First Time Contributing?

1. Read this file
2. Run the Git configuration commands above
3. Create a test commit: `git commit --allow-empty --signoff -m "Test commit"`
4. Verify with: `git log --oneline -1`
5. You're ready! Delete the test commit if it's not needed: `git reset --soft HEAD~1`

### Code Review Standards

- **Every commit must be signed** — no exceptions
- **One sign-off per commit** — Probot enforces this
- **Team ownership** — each package should have clear owners in `.github/CODEOWNERS`

---

## License

By contributing, you agree that your work will be distributed under the **Apache License 2.0**. See the LICENSE file for details.

---

## Questions?

If you encounter issues with DCO or need clarification, refer to the full setup guide: **DCO_Setup_Guide.docx**

Thank you for helping build fiscal transparency in Guatemala! 🇬🇹

---

*Last updated: April 2026*
