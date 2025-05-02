The error:

```
`catch_and_raise': ActiveSupport::MessageEncryptor::InvalidMessage
```

means that **Rails is trying to decrypt something (like credentials or encrypted cookies) using the wrong key**, and the message can't be decrypted.

---

## 🛠️ Common Causes and Fixes

### 🔐 1. **Wrong `SECRET_KEY_BASE` or `RAILS_MASTER_KEY`**
This is **by far the most common cause**. If you're using Rails 5.2+, 6, or 7, your app uses `config/credentials.yml.enc`, and it must be decrypted with the correct **master key**.

#### ✅ Fix:
- Make sure `config/master.key` exists **or** you have `RAILS_MASTER_KEY` set in the environment.

Check:
```bash
echo $RAILS_MASTER_KEY
```

If it's empty, set it:
```bash
export RAILS_MASTER_KEY=your_real_master_key
```

You can copy `master.key` from development or wherever it was originally created.

---

### 🧪 2. **Trying to Read Old/Invalid Encrypted Cookies**
If your app's `secret_key_base` changed (e.g., between deploys), and a client sends old cookies, Rails may throw this error.

#### ✅ Fix:
Clear cookies in your browser, or instruct users to do so. In dev, clear session storage manually:

```bash
rails console
Rails.cache.clear
```

Or delete the session store in Redis/memcached, depending on your setup.

---

### ⚙️ 3. **Mismatch Between Encrypted Files and Master Key**
If someone edited `credentials.yml.enc` on one machine and deployed it without updating the corresponding `master.key`, it’ll break.

#### ✅ Fix:
Ensure that `credentials.yml.enc` and `master.key` are from the **same environment/machine**.

---

### ✅ How to Check Everything

```bash
rails credentials:show
```

If that fails with the same error, your `master.key` or `RAILS_MASTER_KEY` is wrong.

---

## 🚀 Solution Workflow Summary

1. Run this to get/set the correct key:
```bash
RAILS_ENV=production rails credentials:edit
```

2. If you’re on a server:
- Set `RAILS_MASTER_KEY` in your environment
- OR place `config/master.key` on the server

3. If you're using `SECRET_KEY_BASE` manually:
```bash
RAILS_ENV=production bundle exec rake secret
export SECRET_KEY_BASE=that_value
```

---

Would you like help writing an Ansible task to securely copy the `master.key` to your server?

[Unit]
Description=Rails App
After=network.target

[Service]
Type=simple
User=deploy
WorkingDirectory=/root/pro_apis
Environment="RAILS_ENV=production"
Environment="PORT=3000"
ExecStart=/usr/local/bin/bundle exec puma -C config/puma.rb
Restart=always

[Install]
WantedBy=multi-user.target


### Enable and start it:

sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable rails-app
sudo systemctl start rails-app

#### To check logs:
journalctl -u rails-app -f

## port check