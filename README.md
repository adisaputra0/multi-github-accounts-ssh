# Setting Up Multiple GitHub Accounts on One Laptop

This guide explains how to configure 2 or more GitHub accounts on a single laptop using SSH, without needing to log in/out in the browser or Git CLI every time you switch projects.

This method is useful if you have:

- A personal GitHub account
- An organization / work / campus GitHub account
- A freelance project GitHub account

## 1. How It Works

Let's say we have two accounts:

| Account      | Purpose                | Example Username   |
| ------------ | ---------------------- | ------------------ |
| Work Account | Organization / Company | `work-account`     |
| Main Account | Personal               | `personal-account` |

We'll create a separate SSH key for each account, then register an SSH host alias so Git knows which key to use.

```
Laptop
├── SSH Key: id_ed25519_work     ──► GitHub: work-account     (Host: github-work)
└── SSH Key: id_ed25519_personal ──► GitHub: personal-account (Host: github-personal)
```

Example SSH remote URLs:

- `git@github-work:work-account/work-project.git` → Uses the work key
- `git@github-personal:personal-account/portfolio.git` → Uses the personal key

## 2. Separating Authentication from Commit Identity

Understand these two important concepts:

- **SSH Key & Remote URL**: Determines authentication access rights to the GitHub repository (who can push/pull).
- **Git Config (`user.name` & `user.email`)**: Determines the commit author identity recorded in the Git history.

> ⚠️ Setting up Account A's SSH key does NOT automatically change your commit `user.email` to Account A. Both need to be configured separately.

## 3. Check Full Prerequisites

Open Git Bash, then check your Git installation and the existence of the SSH folder:

```bash
git --version
ls ~/.ssh
```

(If `ls ~/.ssh` shows a "folder not found" error, the folder will be created automatically in the next step.)

## 4. Create an SSH Key for the Work Account

> ⚠️ **IMPORTANT (Windows / Git Bash specific)**: Don't manually type `~/.ssh/...` inside the interactive `ssh-keygen` prompt, since the tilde (`~`) isn't evaluated by the prompt and will cause an error. Use the `-f` flag directly in the command instead.

Run this one-line command:

```bash
ssh-keygen -t ed25519 -C "work-email@example.com" -f ~/.ssh/id_ed25519_work
```

Press Enter if you don't want to use a passphrase (or enter one for extra security).

The key is created at `~/.ssh/id_ed25519_work`.

## 5. Create an SSH Key for the Personal Account

Run the following command:

```bash
ssh-keygen -t ed25519 -C "personal-email@example.com" -f ~/.ssh/id_ed25519_personal
```

## 6. Verify the Key Files

Check whether the key files were created:

```bash
ls ~/.ssh
```

Make sure at least the following files exist:

- `id_ed25519_work` (Private Key)
- `id_ed25519_work.pub` (Public Key)
- `id_ed25519_personal` (Private Key)
- `id_ed25519_personal.pub` (Public Key)

## 7. Create an SSH Config File (`~/.ssh/config`)

Create and open the SSH config file:

```bash
touch ~/.ssh/config
nano ~/.ssh/config
```

Paste in the following configuration:

```
# ==============================
# GitHub - Work Account
# ==============================
Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes

# ==============================
# GitHub - Personal Account
# ==============================
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes
```

**How to save in Nano:**
Press `Ctrl + X` → Type `Y` → Press `Enter`.

Set stricter permissions on the config file for security:

```bash
chmod 600 ~/.ssh/config
```

## 8. Add the Public Key to Each GitHub Account

### A. Work Account

Display and copy the public key content:

```bash
cat ~/.ssh/id_ed25519_work.pub
```

Open GitHub (log in to the work account) → **Settings** → **SSH and GPG keys** → **New SSH key**.

Give it a title (e.g. `Laptop - Work`), paste the key into the Key field, then click **Add SSH key**.

### B. Personal Account

Display and copy the public key content:

```bash
cat ~/.ssh/id_ed25519_personal.pub
```

Open GitHub (log in to the personal account) → **Settings** → **SSH and GPG keys** → **New SSH key**.

Give it a title (e.g. `Laptop - Personal`), paste the key into the Key field, then click **Add SSH key**.

## 9. Test the SSH Connection

Run the following test commands in Git Bash:

```bash
ssh -T github-work
```

Success message: `Hi <work-username>! You've successfully authenticated...`

```bash
ssh -T github-personal
```

Success message: `Hi <personal-username>! You've successfully authenticated...`

## 10. How to Use in Projects (Workflow)

### A. Cloning a New Repository

Use the host alias (`github-work` or `github-personal`), not the regular `github.com` or HTTPS.

```bash
# Clone a work repo
git clone git@github-work:work-account/work-project.git

# Clone a personal repo
git clone git@github-personal:personal-account/portfolio.git
```

### B. Updating an Existing Repository

If a repo was already cloned via regular HTTPS/SSH, update its remote URL:

```bash
# In the Work project folder:
git remote set-url origin git@github-work:work-account/work-project.git

# In the Personal project folder:
git remote set-url origin git@github-personal:personal-account/portfolio.git
```

## 11. Setting Commit Identity (`user.name` & `user.email`)

Go into each project's folder and run `git config --local` so commits are recorded under the correct email.

**For the Work project:**

```bash
cd path/to/work-project
git config user.name "Work Full Name"
git config user.email "work-email@example.com"
```

**For the Personal project:**

```bash
cd path/to/portfolio
git config user.name "Personal Full Name"
git config user.email "personal-email@example.com"
```

## 12. Checklist Before `git push`

Use these commands to make sure your local repository's configuration is correct:

```bash
# 1. Check the remote URL (push/pull authentication)
git remote -v

# 2. Check the local commit identity
git config user.name
git config user.email
```

## Quick Command Reference

| Action              | Command                                                            |
| ------------------- | ------------------------------------------------------------------ |
| Check SSH keys      | `ls ~/.ssh`                                                        |
| Test SSH connection | `ssh -T github-work`                                               |
| Check remote repo   | `git remote -v`                                                    |
| Set remote repo     | `git remote set-url origin git@<HOST_ALIAS>:<USERNAME>/<REPO>.git` |
| Set local email     | `git config user.email "email@example.com"`                        |
