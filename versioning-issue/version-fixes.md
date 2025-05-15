Great — if you're using **`rbenv`**, and you see this when running via SSH:

```
ruby -v
# => ruby 3.2.3
```

…it means `rbenv` is **not initialized** in non-interactive shells (like SSH commands or GitHub Actions).

---

## ✅ Fix `rbenv` for SSH & Deploy Scripts

### 🛠️ Step-by-step:

---

### ✅ 1. Add `rbenv` to your `PATH` and initialize it in `.bashrc` and `.bash_profile`

Edit both files:

```bash
nano ~/.bashrc
```

Add:

```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init - bash)"
```

Now also edit `.bash_profile` or `.profile`:

```bash
nano ~/.bash_profile
```

Add the same:

```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init - bash)"
```

> ✅ This ensures **both login and non-login shells** (like SSH) load `rbenv`.

---

### ✅ 2. Reload the shell environment

```bash
source ~/.bashrc
source ~/.bash_profile
```

---

### ✅ 3. Set global Ruby version

Make sure `rbenv` is pointing to Ruby 3.3.0:

```bash
rbenv global 3.3.0
rbenv rehash
```

Verify:

```bash
which ruby     # Should be something like ~/.rbenv/shims/ruby
ruby -v        # Should be ruby 3.3.0
```

---

### ✅ 4. Test via SSH

```bash
ssh user@your-server 'ruby -v'
```

✅ You should now see:

```
ruby 3.3.0
```

---

### ✅ 5. Optional: Force `rbenv` activation in scripts

If you run commands from **GitHub Actions**, **Ansible**, or scripts, add this before using `ruby`, `bundle`, etc.:

```bash
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init - bash)"
rbenv shell 3.3.0
```

Example SSH script:

```bash
ssh user@server <<'EOF'
  export PATH="$HOME/.rbenv/bin:$PATH"
  eval "$(rbenv init - bash)"
  rbenv shell 3.3.0

  cd /var/www/app
  ruby -v         # ✅ should be ruby 3.3.0
  bundle install
EOF
```

---

## ✅ Summary

| Issue                                 | Solution                                         |
| ------------------------------------- | ------------------------------------------------ |
| `ruby -v` shows wrong version via SSH | Add `rbenv` setup to `.bashrc` & `.bash_profile` |
| Ruby not defaulting to 3.3.0          | Run `rbenv global 3.3.0`                         |
| Deploy script uses wrong Ruby         | Add `rbenv` activation lines manually            |

---

Let me know if you’d like a **reusable shell script** that does all of this setup automatically.
