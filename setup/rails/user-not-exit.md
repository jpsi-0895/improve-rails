The error:

```
rails-app.service: Failed to determine user credentials: No such process
```

means that **the `User=` specified in your `rails-app.service` file does not exist on the system**.

---

## ✅ How to Fix

### 🔍 Step 1: Check the user

Open your `rails-app.service` file:

```ini
[Service]
User=deploy
```

Now run on your server:

```bash
id deploy
```

If you see:

```
id: ‘deploy’: no such user
```

Then the `deploy` user doesn’t exist — and systemd cannot start the service under a nonexistent user.

---

### 🛠️ Step 2: Choose or create a user

#### Option A: Use an existing user

Replace `User=deploy` in your service file with:

```ini
User=ubuntu        # or whatever user you're logged in as
```

Check with:

```bash
whoami
```

#### Option B: Create the `deploy` user (if needed)

```bash
sudo adduser --disabled-password --gecos "" deploy
```

Then make sure your app files are owned by `deploy`:

```bash
sudo chown -R deploy:deploy /var/www/myapp
```

---

### 🔄 Step 3: Reload systemd and try again

```bash
sudo systemctl daemon-reload
sudo systemctl restart rails-app
sudo journalctl -u rails-app -f
```

---

## ✅ Summary

- ❌ Error happens because the `User=` in your service doesn't exist
- ✅ Use a valid user (`ubuntu`, `www-data`, or create `deploy`)
- 🔁 Reload systemd and restart the service

---

Would you like help configuring a safe user setup for deploying your Rails app long-term?