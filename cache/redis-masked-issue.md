Ah, got it — that error means the `redis.service` is **masked**, which prevents it from starting or restarting. It’s a systemd thing, and it's easy to fix.

---

## ❌ Error
```
Failed to restart redis.service: Unit redis.service is masked.
```

> Masked = disabled at the system level (intentionally or by mistake). You can’t start it until it’s unmasked.

---

## ✅ Fix It: Unmask & Restart Redis

Run the following in your terminal:

```bash
sudo systemctl unmask redis.service
sudo systemctl enable redis.service
sudo systemctl start redis.service
```

Then check that it’s running:

```bash
sudo systemctl status redis.service
```

You should now see Redis active and running. ✅

---

## 🧪 Bonus Check: Is Redis Working?

```bash
redis-cli ping
# => PONG
```

---

## 💡 Why Does This Happen?

- If you previously installed Redis using Snap, Docker, or from source, systemd might mask the service.
- Sometimes masked manually to prevent auto-start.
- Installing a new version might cause conflicts between system Redis and other versions (e.g., Dockerized Redis or rbenv/rvm path issues).

---

Let me know if you're using **Docker**, **WSL**, or **custom Redis setup**, and I’ll tailor the fix!