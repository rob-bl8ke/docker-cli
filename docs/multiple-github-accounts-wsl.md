# Manage multiple GitHub accounts via SSH in WSL 2 and using Visual Studio Code for development and commits.


## ✅ Step 1: Check for Existing SSH Keys

Open your WSL terminal and run:

```bash
ls -al ~/.ssh
```

You’re looking for files like:

* `id_rsa` and `id_rsa.pub` (default key pair)
* `id_ed25519` and `id_ed25519.pub` (newer standard)
* Custom keys like `github_personal`, `github_work`, etc.

If the folder doesn’t exist, create it:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

---

## 🗑️ Step 2: Remove Any Existing SSH Keys for GitHub (Optional)

If you want a clean slate, and you're sure these keys aren't used for other services:

```bash
rm ~/.ssh/id_*
rm ~/.ssh/github_*
```

You can remove just GitHub-related ones if you recognize them (e.g., `rm ~/.ssh/github_work*`).

> ⚠️ Only remove keys you're sure aren't needed elsewhere.

---

## 🛠️ Step 3: Generate Separate SSH Keys for Each GitHub Account

Let’s say you have a **personal** and **work** GitHub account:

```bash
ssh-keygen -t ed25519 -C "your-email@personal.com" -f ~/.ssh/github_personal
ssh-keygen -t ed25519 -C "your-email@work.com" -f ~/.ssh/github_work
```

Just press Enter to accept defaults for passphrase or add one if you want.

---

## 🔑 Step 4: Add SSH Keys to the SSH Agent

Make sure the SSH agent is running:

```bash
eval "$(ssh-agent -s)"
```

Add both keys:

```bash
ssh-add ~/.ssh/github_personal
ssh-add ~/.ssh/github_work
```

---

## 📝 Step 5: Create or Edit SSH Config File

Create or edit `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Paste the following:

```ssh
# Personal GitHub
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_personal

# Work GitHub
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_work
```

This trick lets you refer to `github.com-personal` and `github.com-work` as if they’re different hosts.

---

## 🧪 Step 6: Add Public Keys to GitHub

Run the following to get the public key:

```bash
cat ~/.ssh/github_personal.pub
```

Copy the output and add it to your [GitHub account → Settings → SSH and GPG Keys → New SSH key](https://github.com/settings/keys)

Do the same for `github_work.pub` on your other account.

---

## 📁 Step 7: Set Up Code Folder with Sub-Folders per Account

Let’s say:

```bash
/code/github-personal/
/code/github-work/
```

Clone your repos using the custom hostnames:

```bash
# For personal repo
git clone git@github.com-personal:yourusername/some-repo.git ~/code/github-personal/some-repo

# For work repo
git clone git@github.com-work:yourworkusername/another-repo.git ~/code/github-work/another-repo
```

Now Git knows which key to use based on the alias (`github.com-personal` or `github.com-work`).

---

## 👨‍💻 Step 8: Use VS Code to Work Normally

From WSL terminal:

```bash
code ~/code/github-personal/some-repo
```

Make commits and push as normal. Git will use the correct SSH identity under the hood.

---

## 🔁 Optional: Verify Connection

```bash
ssh -T git@github.com-personal
ssh -T git@github.com-work
```

You should see a success message from each GitHub account.
