# 🔁 Switching Between Multiple GitHub Accounts (Work & Personal)

This guide helps you configure Git and SSH to easily switch between multiple GitHub accounts (e.g., one for work and one personal).

## 📁 Folder Structure Recommendation

Use this structure for your local repos:

```
~/Projects/
├── personal/
└── work/
```

---

## 🍏 macOS Setup Guide

### 1. Generate SSH Keys

Open Terminal and run:

```bash
# Personal GitHub key
ssh-keygen -t rsa -C "your_personal_email@example.com" -f ~/.ssh/id_rsa_personal

# Work GitHub key
ssh-keygen -t rsa -C "your_work_email@company.com" -f ~/.ssh/id_rsa_work
```

### 2. Add SSH Keys to GitHub

* Copy the contents of the `.pub` files:

  ```bash
  cat ~/.ssh/id_rsa_personal.pub
  cat ~/.ssh/id_rsa_work.pub
  ```
* Go to GitHub → **Settings > SSH and GPG Keys** → Add the keys under the relevant accounts.

### 3. Configure SSH Config

Edit or create `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add:

```ssh
# Personal GitHub
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_personal

# Work GitHub
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_work
```

### 4. Clone Using the Correct Host

Use the custom host when cloning:

```bash
# Personal
git clone git@github-personal:your-username/repo.git

# Work
git clone git@github-work:your-company-org/repo.git
```

### 5. Set Up Role-Based Git Identity

#### Create config files:

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

#### Update `~/.gitconfig`:

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

---

## 🪟 Windows Setup Guide (Git Bash or WSL)

### 1. Generate SSH Keys

Open **Git Bash** or **WSL** and run:

```bash
ssh-keygen -t rsa -C "your_personal_email@example.com" -f ~/.ssh/id_rsa_personal
ssh-keygen -t rsa -C "your_work_email@company.com" -f ~/.ssh/id_rsa_work
```

### 2. Add Public Keys to GitHub

Display the keys:

```bash
cat ~/.ssh/id_rsa_personal.pub
cat ~/.ssh/id_rsa_work.pub
```

Copy and paste into GitHub → **Settings > SSH and GPG keys**.

### 3. Configure SSH Config File

Create or edit `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Add:

```ssh
# Personal
Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_personal

# Work
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa_work
```

### 4. Clone Repos Using Host Alias

```bash
git clone git@github-personal:your-username/repo.git
git clone git@github-work:your-company-org/repo.git
```

### 5. Set Role-Based Git Identity

#### Create identity files

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

#### Edit main `~/.gitconfig`

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

---

## ✅ Test It

```bash
cd ~/Projects/work/some-repo
git config user.email  # should return your work email

cd ~/Projects/personal/some-repo
git config user.email  # should return your personal email
```

---

## 🧠 Tips

* If you use **VS Code**, make sure it uses the correct SSH key when pushing (see `.vscode/settings.json` if needed).
* You can also create aliases or scripts to simplify cloning and switching accounts.

---

## 👨‍💻 Questions?

Contact your DevOps/Infra team or `Shay Bushary` for help.
