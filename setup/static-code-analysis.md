**Static code analysis** is the process of analyzing source code without executing it, to find bugs, code smells, vulnerabilities, or ensure coding standards are met. In a **Rails** (Ruby) project, it’s essential for:

- Improving code quality
- Catching bugs early
- Enforcing style and security best practices
- Ensuring maintainability

---

## ✅ Common Tools for Static Code Analysis in Rails

Here are the most widely used static analysis tools for a **Ruby on Rails** project:

### 🔍 1. **RuboCop** – Linter & Style Guide Enforcer
- Enforces Ruby style guide (e.g., indentation, naming, etc.)
- Can auto-correct issues

#### Installation:
```bash
bundle add rubocop --group="development"
```

#### Usage:
```bash
bundle exec rubocop
```

#### Auto-fix:
```bash
bundle exec rubocop -A
```

You can customize with `.rubocop.yml`.

---

### 🔐 2. **Brakeman** – Security Scanner
- Detects vulnerabilities like SQL injection, XSS, mass assignment
- Rails-specific static security analysis

#### Installation:
```bash
gem install brakeman
```

#### Usage:
```bash
brakeman
```

Generates a security report for your app.

---

### 🧪 3. **Reek** – Code Smell Detector
- Detects "code smells" like long methods, large classes, duplicate logic, etc.

#### Installation:
```bash
gem install reek
```

#### Usage:
```bash
reek
```

Configure with `.reek.yml` to silence or customize certain rules.

---

### 📦 4. **Bundler Audit** – Dependency Vulnerability Checker
- Checks for known vulnerabilities in your `Gemfile.lock` against RubySec's advisory DB

#### Installation:
```bash
gem install bundler-audit
```

#### Usage:
```bash
bundle audit check --update
```

---

### 📊 5. **Rails Best Practices** – Rails-specific Anti-Patterns
- Looks for things like skinny controller violations, improper MVC usage, etc.

#### Installation:
```bash
gem install rails_best_practices
```

#### Usage:
```bash
rails_best_practices .
```

---

## 📦 Optional: CI Integration

You can add these checks to **GitHub Actions**, GitLab CI/CD, or any CI system to **fail builds** on style, security, or smell issues.

---

## 🧰 Full Setup (Example Gemfile Group)

```ruby
group :development, :test do
  gem 'rubocop', require: false
  gem 'brakeman', require: false
  gem 'reek', require: false
  gem 'bundler-audit', require: false
  gem 'rails_best_practices', require: false
end
```

Run them regularly or include in your CI/CD.

---

Would you like a ready-to-use static analysis script or GitHub Actions setup for these tools?