To diagnose or monitor your **Rails app** running with **Puma** and served by **Nginx**, you’ll want to check both sets of logs:

---

## 📜 1. **NGINX Logs**

### 📂 Default locations (Ubuntu/Debian):

- **Access log**:
  ```
  /var/log/nginx/access.log
  ```
- **Error log**:
  ```
  /var/log/nginx/error.log
  ```

### 🕵️ View logs live:
```bash
tail -f /var/log/nginx/access.log /var/log/nginx/error.log
```

If you've defined a custom log location in your site config:

```nginx
server {
    access_log /var/log/nginx/rails_access.log;
    error_log /var/log/nginx/rails_error.log;
}
```

Then tail those instead:
```bash
tail -f /var/log/nginx/rails_*.log
```

---

## 🐾 2. **Puma Logs**

Puma logs depend on how you **started it**:

### ✅ If running via `systemd`:
Check logs with:

```bash
journalctl -u rails-app -f
```

(Replace `rails-app` with your actual service name.)

---

### ✅ If running manually or via `foreman`:

By default, logs go to STDOUT. You can redirect or configure log files in `puma.rb`.

#### Example `config/puma.rb`:
```ruby
stdout_redirect '/var/www/myapp/log/puma.stdout.log', '/var/www/myapp/log/puma.stderr.log', true
```

Then check:

```bash
tail -f /var/www/myapp/log/puma.stdout.log
```

---

## 🚀 Bonus: Rails Logs

Rails logs app-specific activity (requests, DB calls, etc.):

```bash
tail -f log/production.log
```

Make sure you're running in the right environment (`RAILS_ENV=production`).

---

## ✅ Summary

| Log Type     | File or Command                             |
|--------------|---------------------------------------------|
| NGINX access | `/var/log/nginx/access.log`                 |
| NGINX error  | `/var/log/nginx/error.log`                  |
| Puma (systemd) | `journalctl -u rails-app -f`             |
| Puma (file)  | `log/puma.stdout.log`, `log/puma.stderr.log` |
| Rails logs   | `log/production.log` or `log/development.log` |

---

Would you like help configuring Puma to always log to files instead of just STDOUT?