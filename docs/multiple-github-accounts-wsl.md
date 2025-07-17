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

# Gotcha

Following these steps you can clone the repository without issues. You can commit but you cannot push to the remote repository.
```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

This strongly suggests that Git is falling back to the **default GitHub host (`github.com`)**, which is not mapped to your custom SSH config (e.g. `github.com-personal` or `github.com-work`).

Let’s fix that with a clear 3-part checklist:

---

## ✅ 1. Confirm Your Git Remote URL Uses the Custom Host

Run this in your repository:

```bash
git remote -v
```

You’ll likely see something like:

```
origin  git@github.com:yourusername/your-repo.git (fetch)
origin  git@github.com:yourusername/your-repo.git (push)
```

This is the problem. It’s using `github.com`, which does not point to `github.com-personal` or `github.com-work` in your `~/.ssh/config`.

### ✅ Fix it:

Update the remote to match your SSH config alias:

```bash
git remote set-url origin git@github.com-personal:yourusername/your-repo.git
```

Verify:

```bash
git remote -v
```

You should now see:

```
origin  git@github.com-personal:yourusername/your-repo.git (fetch)
origin  git@github.com-personal:yourusername/your-repo.git (push)
```

---

## ✅ 2. Test the SSH Connection Directly

You can test if GitHub is seeing your key:

```bash
ssh -T git@github.com-personal
```

You should see something like:

```
Hi yourusername! You've successfully authenticated, but GitHub does not provide shell access.
```

If not, something is wrong with your key, config, or GitHub account setup.

---

## ✅ 3. Optional: Make Sure SSH Agent Has the Key

Sometimes WSL forgets keys after reboot.

Re-add it:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/github_personal
```

You can check loaded keys with:

```bash
ssh-add -l
```

## 🛠️ Step-by-Step: Auto-Load SSH Keys in WSL 2

If WSL forgets keys after reboot here’s a simple way to automatically start the SSH agent and add your GitHub keys every time your WSL session starts.


### ✅ 1. Edit Your `.bashrc` or `.zshrc` File

Assuming you're using Bash (default), run:

```bash
nano ~/.bashrc
```

Scroll to the bottom and add this block:

```bash
# === SSH Agent Setup for GitHub ===
if ! pgrep -u "$USER" ssh-agent > /dev/null; then
  eval "$(ssh-agent -s)"
fi

# Add keys if not already added
ssh-add -l > /dev/null 2>&1
if [ $? -ne 0 ]; then
  ssh-add ~/.ssh/github_personal 2>/dev/null
  ssh-add ~/.ssh/github_work 2>/dev/null
fi
```

> Replace `github_work` with your actual key name(s) if different.

---

### ✅ 2. Save and Reload

Save with `Ctrl+O`, Enter, then exit with `Ctrl+X`.

Then reload:

```bash
source ~/.bashrc
```

---

### ✅ 3. Test

Now close and reopen your WSL terminal, then run:

```bash
ssh-add -l
```

You should see both your keys listed:

```
256 SHA256:... /home/yourname/.ssh/github_personal (ED25519)
256 SHA256:... /home/yourname/.ssh/github_work (ED25519)
```

Now you can push without manually re-adding keys.

---

Would you like to make this work in VS Code’s integrated terminal too, or automatically reload when you boot Windows/WSL?

