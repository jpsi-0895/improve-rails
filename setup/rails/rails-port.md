You got it — running a Rails server on a **custom port** is super easy.

---

### 🚀 To Start the Rails Server on a Specific Port

Just pass the `-p` flag when starting the server:

```bash
rails server -p 4000
```

That’ll start it on `http://localhost:4000`

---

### ✅ You can also combine it with host if needed:

```bash
rails server -b 0.0.0.0 -p 4000
```

- `-b 0.0.0.0` makes the app accessible from other devices on your local network
- `-p 4000` sets the port

---

### 🔁 Want to set the port **by default**?

You can add this to a `.railsrc` file in your home directory:

```bash
--port=4000
```

Or, set an alias in your shell config (`~/.bashrc`, `~/.zshrc`):

```bash
alias rs='rails server -p 4000'
```

---

Let me know if you want to run multiple Rails apps on different ports or deploy it behind NGINX or something like that 👀