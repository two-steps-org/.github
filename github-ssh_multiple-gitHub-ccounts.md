# Multi-Account Git & SSH Setup on Ubuntu

A step-by-step guide to configure separate Git identities and SSH keys for work and personal GitHub accounts on Ubuntu.

## 1. Generate SSH Keys

```bash
ssh-keygen -t ed25519 -C "your-personal@email.com" -f ~/.ssh/personal -N ""
ssh-keygen -t ed25519 -C "your-work@email.com" -f ~/.ssh/work -N ""
```

## 2. Configure SSH

Create `~/.ssh/config`:

```
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  AddKeysToAgent yes
```

## 3. Load Keys into SSH Agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/personal
ssh-add ~/.ssh/work
```

To auto-load keys on login, add the following to `~/.bashrc`:

```bash
eval "$(ssh-agent -s)" > /dev/null 2>&1
ssh-add ~/.ssh/personal ~/.ssh/work 2>/dev/null
```

## 4. Add Public Keys to GitHub

Copy each public key and add it to the corresponding GitHub account under **Settings > SSH and GPG keys > New SSH key**.

```bash
cat ~/.ssh/personal.pub
cat ~/.ssh/work.pub
```

## 5. Configure Git

Create `~/.gitconfig`:

```
[pull]
  rebase = false

[core]
  autocrlf = input
  sshCommand = ssh -i ~/.ssh/work
  # or use personal depend on you what you want to use by default

[user]
  name = Your Name
  email = your-work@twosteps.ai
  # or personal

# you can choose another folder directory depends on you.
[includeIf "gitdir:~/projects/work/"]
  path = ~/.gitconfig-work

# same here 
[includeIf "gitdir:~/projects/personal/"]
  path = ~/.gitconfig-personal
```

Create `~/.gitconfig-work`:

```
[user]
  name = Your Name (Work)
  email = your-work@email.com

[core]
  sshCommand = ssh -i ~/.ssh/work
```

Create `~/.gitconfig-personal`:

```
[user]
  name = Your Name
  email = your-personal@email.com

[core]
  sshCommand = ssh -i ~/.ssh/personal
```

## 6. Create Project Directories

```bash
mkdir -p ~/projects/work
mkdir -p ~/projects/personal
```

## 7. Verify

From a git repository under `~/projects/work/`:

```bash
git config user.email
# should output your work email

ssh -T git@github.com
# should authenticate with work key
```

From a git repository under `~/projects/personal/`:

```bash
git config user.email
# should output your personal email

ssh -T git@github.com
# should authenticate with personal key
```

**Note:** The `includeIf "gitdir:..."` directive only activates inside a git repository. Make sure to run `git init` or `git clone` inside the project directories.
