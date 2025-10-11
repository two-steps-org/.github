# 🔁 Switching Between Multiple GitHub Accounts (Work & Personal)

This guide helps you configure Git and SSH to easily switch between multiple GitHub accounts (e.g., one for work and one personal).

---

# 🍏 macOS — Multiple GitHub Accounts (Work & Personal)

## 0) Folder layout (optional but recommended)

```
~/Projects/
├── personal/
└── work/
```

---

## 1) Create dedicated SSH keys (Ed25519; shorter & stronger than RSA)

```bash
# Personal key (file will be ~/.ssh/personal and ~/.ssh/personal.pub)
ssh-keygen -t ed25519 -C "your_personal_email@example.com" -f ~/.ssh/personal

# Work key (file will be ~/.ssh/work and ~/.ssh/work.pub)
ssh-keygen -t ed25519 -C "your_work_email@company.com" -f ~/.ssh/work
```

**Permissions (important on macOS/Linux):**

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/personal ~/.ssh/work
chmod 644 ~/.ssh/personal.pub ~/.ssh/work.pub
```

---

## 2) Add public keys to the right GitHub accounts

```bash
# Copy to clipboard (macOS)
pbcopy < ~/.ssh/personal.pub
pbcopy < ~/.ssh/work.pub
```

* Go to each GitHub account → **Settings → SSH and GPG keys** → **New SSH key** → paste.

> If your org uses **SSO**, you must **Authorize** the SSH key for the org:
> [https://github.com/orgs/](https://github.com/orgs/)<your-org>/sso → find your key → **Authorize**.

---

## 3) SSH config with host aliases (uses port 443—firewall friendly)

Edit `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Paste:

```ssh
# Personal GitHub over 443
Host github-personal
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/personal
  AddKeysToAgent yes
  UseKeychain yes

# Work GitHub over 443
Host github-work
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/work
  AddKeysToAgent yes
  UseKeychain yes
```

---

## 4) Load the keys into the macOS keychain

```bash
# Start the agent (usually auto on macOS)
eval "$(ssh-agent -s)"

# Add with keychain persistence
ssh-add --apple-use-keychain ~/.ssh/personal
ssh-add --apple-use-keychain ~/.ssh/work

# Confirm they’re loaded
ssh-add -l
```

---

## 5) Path-based Git identity (auto-switch email per folder)

Create identity snippets:

```bash
nano ~/.gitconfig-personal
```

```ini
[user]
  name = Your Name
  email = your_personal_email@example.com
```

```bash
nano ~/.gitconfig-work
```

```ini
[user]
  name = Your Name (Company)
  email = your_work_email@company.com
```

Main `~/.gitconfig`:

```ini
[pull]
  rebase = false

[core]
  autocrlf = input

[includeIf "gitdir:~/Projects/personal/"]
  path = ~/.gitconfig-personal

[includeIf "gitdir:~/Projects/work/"]
  path = ~/.gitconfig-work
```

> Tip: If your folders differ, update the `gitdir:` paths accordingly.
> Use a trailing slash and the **full** tilde path as shown.

---

## 6) Clone using the correct host alias

```bash
# Personal repos
git clone git@github-personal:your-username/repo.git ~/Projects/personal/repo

# Work repos (your case)
git clone git@github-work:two-steps-org/benchmark.git -b main ~/Projects/work/benchmark
```

---

## 7) ✅ Verification (same flow we used to fix your issue)

```bash
# 1) Confirm which account the alias uses (should show your correct GH username)
ssh -vT git@github-work
ssh -vT git@github-personal

# 2) Check repo access explicitly
git ls-remote git@github-work:two-steps-org/benchmark.git

# 3) Test identity auto-switch
cd ~/Projects/work/benchmark
git config user.email     # expect your_work_email@company.com

cd ~/Projects/personal/some-repo
git config user.email     # expect your_personal_email@example.com
```

**If you see “Repository not found”:**

* Double-check the path `two-steps-org/benchmark`.
* Ensure your **work** GH user has repo access.
* If SSO is enabled, **Authorize** the SSH key for the org (again).

---

# 🪟 Windows — Multiple GitHub Accounts (Git Bash or WSL)

> Choose **Git Bash** (Windows native) or **WSL** (Linux in Windows). Steps are the same; I’ll note differences where needed.

## 1) Generate separate SSH keys

```bash
# In Git Bash or WSL
ssh-keygen -t ed25519 -C "your_personal_email@example.com" -f ~/.ssh/personal
ssh-keygen -t ed25519 -C "your_work_email@company.com" -f ~/.ssh/work
```

**Permissions (Git Bash/WSL):**

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/personal ~/.ssh/work
chmod 644 ~/.ssh/personal.pub ~/.ssh/work.pub
```

---

## 2) Add public keys to the right GitHub accounts

```bash
# Print then copy manually
cat ~/.ssh/personal.pub
cat ~/.ssh/work.pub
```

* GitHub → **Settings → SSH and GPG keys** → add each key to the correct account.
* If your org uses **SSO**, go to the org SSO page and **Authorize** the SSH key.

---

## 3) Configure SSH aliases (over port 443)

Edit `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Paste:

```ssh
# Personal GitHub over 443
Host github-personal
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/personal

# Work GitHub over 443
Host github-work
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile ~/.ssh/work
```

> `UseKeychain` is macOS-only, so we omit it on Windows.
> Git Bash/WSL uses the ssh-agent instead.

---

## 4) Start ssh-agent and add keys (Windows)

**Git Bash (recommended):**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/personal
ssh-add ~/.ssh/work
ssh-add -l
```

**WSL note:** If your Windows user already has keys in Pageant/Windows agent, they aren’t auto-shared with WSL. Add the keys inside WSL as above.

---

## 5) Path-based Git identity (auto-switch per folder)

Create identity files:

```bash
nano ~/.gitconfig-personal
```

```ini
[user]
  name = Your Name
  email = your_personal_email@example.com
```

```bash
nano ~/.gitconfig-work
```

```ini
[user]
  name = Your Name (Company)
  email = your_work_email@company.com
```

Main `~/.gitconfig` (Windows differences noted):

```ini
[pull]
  rebase = false

# On Windows, line endings usually:
[core]
  autocrlf = true

# If your repos live under C:\Users\<you>\Projects\...
# In Git Bash, that’s /c/Users/<you>/Projects/...
# Use the Git Bash paths below (adjust <you>):
[includeIf "gitdir:/c/Users/<you>/Projects/personal/"]
  path = ~/.gitconfig-personal

[includeIf "gitdir:/c/Users/<you>/Projects/work/"]
  path = ~/.gitconfig-work
```

> If you keep repos elsewhere, set `gitdir:` to that Git Bash/WSL path.
> Always include the trailing slash.

---

## 6) Clone with the correct host alias

```bash
# Personal
git clone git@github-personal:your-username/repo.git /c/Users/<you>/Projects/personal/repo

# Work (your case)
git clone git@github-work:two-steps-org/benchmark.git -b main /c/Users/<you>/Projects/work/benchmark
```

---

## 7) ✅ Verification (same checks)

```bash
# 1) Which account is used by each alias?
ssh -vT git@github-work
ssh -vT git@github-personal

# 2) Repo access
git ls-remote git@github-work:two-steps-org/benchmark.git

# 3) Identity auto-switch
cd /c/Users/<you>/Projects/work/benchmark
git config user.email     # expect your_work_email@company.com

cd /c/Users/<you>/Projects/personal/some-repo
git config user.email     # expect your_personal_email@example.com
```

**If “Repository not found”:**

* Confirm the path `two-steps-org/benchmark`.
* Ensure your **work** account has access.
* If SSO is enabled for the org, **Authorize** the SSH key.

---

## 🔧 Troubleshooting (Mac & Windows)

* **Wrong account used** when connecting?
  Check which key is offered:

  ```bash
  ssh -vvv git@github-work |& sed -n '1,120p'
  ```

  Look for lines like: `Offering public key: ~/.ssh/work`.
  If the wrong key is offered, re-check `~/.ssh/config`, run `ssh-add -D` then re-add the right keys.

* **Host key prompt for `ssh.github.com:443`**: say **yes** (it’s normal on first connect).

* **VS Code using wrong SSH**: VS Code uses system Git/SSH; ensure your `~/.ssh/config` exists and keys are loaded. On Windows, make sure you’re using Git Bash shell inside VS Code when running Git commands, or configure the Git path in settings.

---

If you want, tell me your exact Windows repo path (e.g., `C:\Users\Shay\Projects\...`) and I’ll pre-fill the `gitdir:` lines for you.
