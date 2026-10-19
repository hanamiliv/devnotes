# How to Manage Separate SSH Keys for Work and Personal Git Accounts

Many developers use one laptop for both work and personal projects. If you use one SSH key and one Git identity for everything, you can end up pushing to the wrong account or committing personal code with your work email.

This guide shows how to keep them separate on macOS. Here is the setup we will build:

| | Work | Personal |
|---|---|---|
| Git host | `gitlab.com` | `github.com` |
| SSH key | `~/.ssh/id_ed25519_work` | `~/.ssh/id_ed25519_personal` |
| Projects folder | `~/work/` | `~/personal/` |
| Commit email | `you@company.com` | `12345+yourname@users.noreply.github.com` |

There are two separate things to set up:

- **SSH key** controls *who can push*, meaning which account you authenticate as.
- **Git config** controls *who the commit says it's from*, meaning the name and email in the commit history.

You need both. A correct SSH key with the wrong email still produces commits under the wrong identity.

---

## Step 1: Check Your Existing Keys

```bash
ls -al ~/.ssh
```

If you already have a key (e.g. `id_ed25519`) used for work, leave it alone. We will add a new key next to it instead of replacing it.

---

## Step 2: Generate a New Key for Personal Use

```bash
ssh-keygen -t ed25519 -C "personal-github" -f ~/.ssh/id_ed25519_personal
```

- `-t ed25519` uses the modern, recommended key type.
- `-C` is just a label to help you recognize the key later. It doesn't have to be an email.
- `-f` sets the file name, so it won't overwrite your existing key.

You will be asked for a **passphrase**. This is a password that encrypts your private key file, so a stolen key file is useless without it. Note that:

- Nothing appears on screen while you type. This is normal.
- Each key can have its own passphrase, and using different ones is better.
- Store it in a password manager. If you forget it, you cannot recover it and must create a new key.

This creates two files:

- `id_ed25519_personal` is your **private key**. Never share it.
- `id_ed25519_personal.pub` is your **public key**. You upload this one to GitHub.

---

## Step 3: Save the Passphrase in macOS Keychain

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519_personal
```

You enter the passphrase once, and macOS remembers it. After that, `git push` and `git pull` work without asking again.

---

## Step 4: Tell SSH Which Key to Use for Each Host

Open `~/.ssh/config` and add one block per host:

```
Host *
  AddKeysToAgent yes
  UseKeychain yes

# Work
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
  IdentitiesOnly yes

# Personal
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes
```

Here is what each part does:

- `Host *` applies to every connection. It loads keys into the agent and uses Keychain automatically.
- `IdentityFile` sets which key to use for that host.
- `IdentitiesOnly yes` is important. It makes SSH offer **only** that key. Without it, SSH may try your other keys too, and the host could log you in as the wrong account.

If your config already has other lines (for example, tool-generated `Include` lines at the top), keep them as they are and just add the new block at the bottom.

---

## Step 5: Add the Public Key to GitHub

Copy the public key:

```bash
pbcopy < ~/.ssh/id_ed25519_personal.pub
```

Then in GitHub, go to **Settings → SSH and GPG keys → New SSH key**, choose **Authentication Key**, paste, and save.

---

## Step 6: Test the Connection

```bash
ssh -T git@github.com
```

You should see:

```
Hi yourname! You've successfully authenticated...
```

If the username is wrong, check the `IdentityFile` and `IdentitiesOnly` lines in Step 4.

---

## Step 7: Use a Different Commit Identity per Folder

Your global `~/.gitconfig` probably has your work name and email. We'll keep that as the default and override it for everything inside `~/personal/`.

**7a. Create a personal config file** in your home folder (`~` is your home directory, e.g. `/Users/yourname`):

```bash
cat > ~/.gitconfig-personal <<'EOF'
[user]
  name = yourname
  email = 12345+yourname@users.noreply.github.com
EOF
```

Use GitHub's **noreply email**, which you can find under **Settings → Emails**, to keep your real email private in public commit history.

**7b. Include it conditionally** by adding this to the **end** of `~/.gitconfig`:

```bash
cat >> ~/.gitconfig <<'EOF'

[includeIf "gitdir:~/personal/"]
  path = ~/.gitconfig-personal
EOF
```

- Use `>>` (append), **not** `>`. A single `>` would erase your existing config.
- The trailing slash in `~/personal/` is required. It makes the rule apply to every repo inside that folder.
- Put it at the end so it overrides the default values above it.

---

## Step 8: Verify

`includeIf` only works **inside a Git repository**, so test from one:

```bash
mkdir -p ~/personal/test-repo && cd ~/personal/test-repo && git init
git config user.email
```

- Inside `~/personal/`, this should print your personal noreply email.
- Inside a work repo, it should print your work email.

Clean up afterwards:

```bash
rm -rf ~/personal/test-repo
```

---

## Daily Usage

Clone personal repos into `~/personal/` using the SSH URL:

```bash
cd ~/personal
git clone git@github.com:yourname/my-project.git
```

To switch an existing repo from HTTPS to SSH:

```bash
git remote set-url origin git@github.com:yourname/my-project.git
```

---

## Bonus: Two Accounts on the Same Host

The setup above works because work and personal use **different hosts** (GitLab vs GitHub). If both are on GitHub, use host aliases instead:

```
Host github-work
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_work
  IdentitiesOnly yes

Host github-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
  IdentitiesOnly yes
```

Then use the alias in the clone URL:

```bash
git clone git@github-personal:yourname/my-project.git
git clone git@github-work:company/service.git
```

---

## Quick Reference

| Task | Command |
|---|---|
| Generate a key | `ssh-keygen -t ed25519 -C "label" -f ~/.ssh/id_ed25519_name` |
| Save to Keychain | `ssh-add --apple-use-keychain ~/.ssh/id_ed25519_name` |
| Copy public key | `pbcopy < ~/.ssh/id_ed25519_name.pub` |
| Test connection | `ssh -T git@github.com` |
| Check commit email | `git config user.email` |
| Change passphrase | `ssh-keygen -p -f ~/.ssh/id_ed25519_name` |