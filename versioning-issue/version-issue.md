If your **server shows Ruby 3.3.0 when you run `ruby -v`**, but you’re still getting this error:

```
Your Ruby version is 3.2.3, but your Gemfile specified 3.3.0
```

…it means the **Ruby version running your app (via the deploy script, service, or CI runner)** is **not** the same Ruby you're using in your terminal session.

---

## 🧠 Root Cause

This often happens when:

1. ✅ You installed Ruby 3.3.0 with a version manager (`rbenv`, `mise`, etc.)
2. ❌ But your **GitHub Actions**, **Puma**, **systemd service**, or **deployment script** is not using that Ruby
3. So it's falling back to the **system Ruby** (3.2.3)

---

## ✅ Fix It Step by Step

### ✅ 1. Confirm Ruby in Terminal

Log into the server and check:

```bash
ruby -v           # ✅ 3.3.0
which ruby        # ✅ Should be under ~/.rbenv or ~/.local/share/mise
```

Good. But this is your interactive shell.

---

### ❌ 2. What's Really Running in GitHub Action or Deploy Script?

If you SSH into the server through GitHub Actions, or run `bundle install` in a script or systemd, it **might not load your environment** (like `.bashrc` or `.bash_profile`).

### 🔎 Test This:

SSH in the same way GitHub Actions does (non-interactive shell):

```bash
ssh user@your-server 'ruby -v'
```

If that shows `ruby 3.2.3`, that's your problem.

---

## ✅ Solution Options

### ✅ A. Explicitly load your version manager in the script

**For rbenv**, in your deploy script or GitHub Actions step:

```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
rbenv shell 3.3.0

ruby -v  # should now be 3.3.0
bundle install
```

**For mise**, do:

```bash
eval "$(mise activate bash)"  # or zsh
ruby -v
```

### ✅ B. Use full path to correct Ruby (if no version manager used)

Find the path to 3.3.0 Ruby:

```bash
which ruby
# Example output: /home/deploy/.rbenv/versions/3.3.0/bin/ruby
```

Then explicitly use that path in your deploy script or systemd service.

---

### ✅ C. Check `.ruby-version`

Ensure your project has a file:

```bash
.ruby-version
```

With:

```
3.3.0
```

Your version manager should auto-switch when you `cd` into the project.

---

## ✅ Summary

| Problem                        | Fix                                                       |
| ------------------------------ | --------------------------------------------------------- |
| App using wrong Ruby version   | Scripts/system services not loading your version manager  |
| GitHub Actions runs wrong Ruby | Add `eval "$(rbenv init -)"` or `mise activate` in script |
| `.ruby-version` mismatch       | Create `.ruby-version` file with correct version          |

---

Would you like me to adjust your GitHub Actions deployment YAML or a systemd service file to use Ruby 3.3.0 explicitly?
