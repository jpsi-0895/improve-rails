Great — this confirms the **core issue**:

> ✅ You have Ruby 3.3.0 installed under a version manager (like `rbenv`, `mise`, or `rvm`),
> ❌ but **non-interactive shells** (like `ssh user@host 'command'`) fall back to **system Ruby 3.2.0**.

---

## ✅ Fix: Ensure Shell Loads Your Ruby Version Manager

You need to **manually load your Ruby version manager** in non-interactive sessions (e.g., during SSH or GitHub Actions deployments).

Below are solutions depending on your Ruby version manager:

---

## 🛠️ 1. If You're Using `rbenv`

### ✅ Fix It by Exporting rbenv in Scripts

Update your deploy script (or GitHub Action `script:`) to include:

```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init - bash)"
rbenv shell 3.3.0

ruby -v  # ✅ should now say 3.3.0
bundle install --deployment
```

### ✅ You can also test with:

```bash
ssh user@server 'export PATH="$HOME/.rbenv/bin:$PATH"; eval "$(rbenv init -)"; ruby -v'
```

---

## 🛠️ 2. If You're Using `mise`

### ✅ Add this before `ruby -v`:

```bash
eval "$(mise activate bash)"
ruby -v
```

So in GitHub Actions or scripts:

```yaml
script: |
  eval "$(mise activate bash)"
  ruby -v
  bundle install
```

### ✅ Test via SSH:

```bash
ssh user@host 'eval "$(mise activate bash)"; ruby -v'
```

---

## 🛠️ 3. (Optional) Set Up `~/.bashrc` for Non-Login Shells

Add the appropriate initialization to your `~/.bashrc` or `~/.profile` (non-login shells):

### For `rbenv`:

```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```

### For `mise`:

```bash
eval "$(mise activate bash)"
```

Then make sure `.bashrc` is sourced in your `.bash_profile` or `.profile` for login shells too:

```bash
# ~/.bash_profile or ~/.profile
[ -f ~/.bashrc ] && . ~/.bashrc
```

---

## ✅ Final Verification

Run this:

```bash
ssh user@your-server 'ruby -v'
```

You should now see:

```
ruby 3.3.0
```

---

## ✅ Summary

| Fix Method                       | Description                                      |
| -------------------------------- | ------------------------------------------------ |
| `rbenv` not loaded in script     | Add `export PATH + eval "$(rbenv init -)"`       |
| `mise` not activated             | Add `eval "$(mise activate bash)"`               |
| `.bashrc`/.profile missing setup | Add manager to `.bashrc` for non-interactive use |
| GitHub Actions fallback Ruby     | Modify script to explicitly activate version     |

---

Would you like me to generate a full GitHub Actions workflow using `rbenv` or `mise` with these fixes baked in?
