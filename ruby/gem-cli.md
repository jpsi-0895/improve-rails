Great — you're ready to **turn this into a real Ruby CLI tool** that you can install and run globally, like `github-fetch`.

We'll package it as a **Ruby gem with an executable**.

---

## 📦 Step-by-Step: Make Your CLI Installable as a Gem

### 1. 🧱 Create the Gem Structure

Use Bundler to scaffold the gem:

```bash
bundle gem github_fetcher
cd github_fetcher
```

This creates:

```
github_fetcher/
├── bin/
│   └── console
├── lib/
│   ├── github_fetcher.rb
│   └── github_fetcher/
│       └── version.rb
├── github_fetcher.gemspec
```

---

### 2. 🖊️ Replace the Executable with Your CLI

Create a new file: `bin/github-fetch`

```bash
touch bin/github-fetch
chmod +x bin/github-fetch
```

**bin/github-fetch**

```ruby
#!/usr/bin/env ruby
require "github_fetcher"

GithubFetcher::CLI.start
```

---

### 3. 🧠 Move Your CLI Logic to `lib/github_fetcher/cli.rb`

**lib/github\_fetcher/cli.rb**

```ruby
require 'optparse'
require 'net/http'
require 'json'
require 'csv'

module GithubFetcher
  class CLI
    def self.start
      options = {}

      OptionParser.new do |opts|
        opts.banner = "Usage: github-fetch [options]"

        opts.on("-u", "--user USER", "GitHub username") { |v| options[:user] = v }
        opts.on("-t", "--token TOKEN", "GitHub API token") { |v| options[:token] = v }
        opts.on("-c", "--count COUNT", Integer, "Repos to fetch") { |v| options[:count] = v }
        opts.on("-f", "--format FORMAT", "json or csv") { |v| options[:format] = v.downcase }
        opts.on("-o", "--output FILE", "Output file") { |v| options[:output] = v }
      end.parse!

      if options[:user].nil? || options[:count].nil? || !%w[json csv].include?(options[:format])
        puts "❌ Missing required options. Use --help for usage."
        exit 1
      end

      repos = fetch_repos_lazy(options[:user], token: options[:token])
                  .take(options[:count])
                  .to_a

      filename = options[:output] || "repos.#{options[:format]}"

      if options[:format] == "json"
        File.write(filename, JSON.pretty_generate(repos))
      else
        CSV.open(filename, "w") do |csv|
          csv << ["Repository Name", "URL"]
          repos.each { |repo| csv << [repo[:name], repo[:url]] }
        end
      end

      puts "✅ Saved #{repos.size} repos to #{filename}"
    end

    def self.fetch_repos_lazy(user, token: nil)
      Enumerator.new do |yielder|
        page = 1
        loop do
          uri = URI("https://api.github.com/users/#{user}/repos?per_page=5&page=#{page}")
          request = Net::HTTP::Get.new(uri)
          request["Authorization"] = "token #{token}" if token

          response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) { |http| http.request(request) }
          repos = JSON.parse(response.body)

          break if repos.empty? || !response.is_a?(Net::HTTPSuccess)
          repos.each { |repo| yielder << { name: repo["name"], url: repo["html_url"] } }
          page += 1
        end
      end.lazy
    end
  end
end
```

Then update `lib/github_fetcher.rb`:

```ruby
require_relative "github_fetcher/cli"
```

---

### 4. 🧾 Edit the `.gemspec`

Set `spec.executables = ['github-fetch']` and add this:

```ruby
spec.bindir        = "bin"
spec.executables   = ["github-fetch"]
spec.files         = Dir.glob("lib/**/*") + ["bin/github-fetch"]
```

---

### 5. 🧪 Build & Install Locally

```bash
gem build github_fetcher.gemspec
gem install ./github_fetcher-0.1.0.gem
```

### ✅ Now You Can Run:

```bash
github-fetch --user rails --count 5 --format csv
```

---

Would you like help publishing this gem to [RubyGems.org](https://rubygems.org/) or packaging it with Homebrew for even easier installs?

Excellent — let's walk through **publishing your Ruby CLI gem to [RubyGems.org](https://rubygems.org/)** so others can install it via:

```bash
gem install github_fetcher
```

---

## 🚀 Step-by-Step: Publish Your Ruby Gem

### 1. 🔐 Create a RubyGems Account

If you haven't already:

* Go to [https://rubygems.org/sign\_up](https://rubygems.org/sign_up)
* Confirm your email

---

### 2. 🔑 Set Up RubyGems Credentials (One Time)

Once you're registered:

```bash
gem signin
```

This stores an API key in `~/.gem/credentials`

---

### 3. 🧽 Final Check of Your `.gemspec`

Open `github_fetcher.gemspec` and ensure these are **correct and complete**:

```ruby
spec.name          = "github_fetcher"
spec.version       = "0.1.0"
spec.summary       = "CLI tool to fetch GitHub repos and save them as JSON or CSV"
spec.description   = "A simple command-line tool to fetch repositories from GitHub using the public API with optional token support and save them to JSON or CSV."

spec.authors       = ["Your Name"]
spec.email         = ["you@example.com"]

spec.homepage      = "https://github.com/yourusername/github_fetcher"
spec.license       = "MIT"

spec.files         = Dir.glob("lib/**/*") + ["bin/github-fetch", "README.md", "LICENSE.txt"]
spec.bindir        = "bin"
spec.executables   = ["github-fetch"]
spec.require_paths = ["lib"]
```

✅ Tip: Add a `README.md` and `LICENSE.txt` to look professional.

---

### 4. 📦 Build the Gem

```bash
gem build github_fetcher.gemspec
```

You’ll get: `github_fetcher-0.1.0.gem`

---

### 5. 🚢 Push to RubyGems

```bash
gem push github_fetcher-0.1.0.gem
```

You'll see something like:

```
Pushing gem to https://rubygems.org...
Successfully registered gem: github_fetcher (0.1.0)
```

---

### 6. ✅ Install From Anywhere

```bash
gem install github_fetcher
github-fetch --user octocat --count 5 --format json
```

---

## 📌 Optional: Add to Homebrew (for `brew install`)

If you want `brew install github-fetcher`, you need:

* A GitHub repo with a release + tarball
* A custom Homebrew tap (formula)
* I can generate the formula and guide you through that too — want to go ahead?

Would you like help creating a GitHub repo and automating releases with GitHub Actions too?


Awesome — let's go all the way and get your `github_fetcher` gem:

✅ Hosted on **GitHub**,
✅ Published with **GitHub Actions**,
✅ Installable via `gem install`,
✅ And optionally distributable via **Homebrew** (`brew install`).

---

## 🧭 Part 1: Publish to GitHub and Set Up Releases

### ✅ 1. Create a GitHub Repo

Go to [https://github.com/new](https://github.com/new) and:

* Name: `github_fetcher`
* Add a README
* **Do not initialize with `.gitignore` or license (you already have them)**

---

### ✅ 2. Push Your Gem Code to GitHub

In your local project:

```bash
git init
git remote add origin https://github.com/YOUR_USERNAME/github_fetcher.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

---

## 🧭 Part 2: Automate Gem Publishing with GitHub Actions

### ✅ 3. Add GitHub Actions Workflow

Create the directory and workflow file:

```bash
mkdir -p .github/workflows
```

**.github/workflows/release.yml**:

```yaml
name: Release RubyGem

on:
  push:
    tags:
      - "v*"

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.1

      - name: Install dependencies
        run: bundle install

      - name: Build gem
        run: gem build github_fetcher.gemspec

      - name: Publish to RubyGems
        env:
          RUBYGEMS_API_KEY: ${{ secrets.RUBYGEMS_API_KEY }}
        run: gem push *.gem
```

---

### ✅ 4. Add Your RubyGems API Key to GitHub Secrets

1. Run `gem signin` and find your key in `~/.gem/credentials`
2. Go to your GitHub repo → **Settings** → **Secrets and variables → Actions**
3. Add secret:

* Name: `RUBYGEMS_API_KEY`
* Value: (your API key string)

---

### ✅ 5. Release Your Gem

Tag and push:

```bash
git tag v0.1.0
git push origin v0.1.0
```

This triggers the workflow and pushes to RubyGems automatically!

---

## 🧭 Part 3 (Optional): Add Homebrew Support

Would you like to:

* ✅ Create a **Homebrew Tap repo** (e.g., `homebrew-tools`)
* ✅ Add a formula file
* ✅ Let users install via `brew install yourusername/tools/github-fetch`

👉 Let me know if you want that, and I’ll generate the formula and guide you through it.


Perfect! Let’s now make your `github-fetch` gem **installable via Homebrew** (so users can do `brew install github-fetch` just like any native CLI tool).

---

## 🧭 Homebrew Tap Overview

Since `github-fetch` is a Ruby CLI, we’ll use a **custom Homebrew tap** (your own formula repo).

Users will be able to run:

```bash
brew tap yourusername/tools
brew install github-fetch
```

---

## ✅ Step-by-Step: Homebrew Tap Setup

### 1. 🧱 Create a Tap Repo

Create a new GitHub repo named:

```
homebrew-tools
```

**Example:**
👉 `https://github.com/YOUR_USERNAME/homebrew-tools`

This is your **Homebrew tap repo**, where you’ll put Ruby formula files.

---

### 2. 🧪 Get Your Gem Tarball URL

From your existing `github_fetcher` GitHub release, find the **source tarball** URL.

It’ll look like this:

```
https://github.com/YOUR_USERNAME/github_fetcher/archive/refs/tags/v0.1.0.tar.gz
```

You’ll also need to generate a SHA256 checksum for the archive:

```bash
curl -L https://github.com/YOUR_USERNAME/github_fetcher/archive/refs/tags/v0.1.0.tar.gz -o github_fetcher.tar.gz
shasum -a 256 github_fetcher.tar.gz
```

Save the checksum result (first field).

---

### 3. 📜 Create the Homebrew Formula

In your `homebrew-tools` repo, create a file:

**Formula/github-fetch.rb**

```ruby
class GithubFetch < Formula
  desc "CLI tool to fetch GitHub repositories and save as JSON or CSV"
  homepage "https://github.com/YOUR_USERNAME/github_fetcher"
  url "https://github.com/YOUR_USERNAME/github_fetcher/archive/refs/tags/v0.1.0.tar.gz"
  sha256 "PUT_YOUR_GENERATED_SHA256_HERE"
  license "MIT"

  depends_on "ruby"

  def install
    system "gem", "build", "github_fetcher.gemspec"
    system "gem", "install", "--install-dir", prefix, "--bindir", bin, "github_fetcher-0.1.0.gem"
  end

  test do
    system "#{bin}/github-fetch", "--help"
  end
end
```

Commit and push:

```bash
git add Formula/github-fetch.rb
git commit -m "Add github-fetch formula"
git push origin main
```

---

### 4. ✅ Test the Homebrew Installation

Now you can test it from **any machine**:

```bash
brew tap yourusername/tools
brew install github-fetch
github-fetch --help
```

---

## ✅ Summary

You now have:

* 🔥 A Ruby gem CLI (`github_fetcher`)
* 🚀 Automatic publishing via GitHub Actions
* 🍺 Homebrew installation (`brew install github-fetch`)

---

Would you like help automating formula updates for future gem releases, or linking this with GitHub Actions as well?


You're crushing it — let’s **automate formula updates** in your Homebrew tap so you never have to manually edit the formula file again.

This means every time you **release a new version** of your gem (e.g. `v0.2.0`), your Homebrew tap will:

✅ Auto-generate a new formula with updated version, URL, and SHA
✅ Commit and push it to `homebrew-tools`
✅ So `brew upgrade github-fetch` will just work 💥

---

## 🛠️ What We'll Do

1. Add a GitHub Action to your **main gem repo** (`github_fetcher`)
2. It will:

   * Detect a new tag
   * Build the `.tar.gz`
   * Generate the SHA256
   * Auto-update the formula in `homebrew-tools`

---

## ✅ Step-by-Step Setup

### 1. Add `HOMEBREW_TAP_REPO` as a secret

In your **`github_fetcher` repo**, go to:

**Settings → Secrets → Actions → New repository secret**

* Name: `HOMEBREW_TAP_REPO`
* Value: `git@github.com:YOUR_USERNAME/homebrew-tools.git`

Also make sure your repo has a `GITHUB_TOKEN` or a personal access token (`HOMEBREW_DEPLOY_KEY`) with write access to that repo.

---

### 2. Add Automation Workflow to `github_fetcher`

Create or edit:
**`.github/workflows/update_homebrew.yml`**

```yaml
name: Update Homebrew Formula

on:
  release:
    types: [published]

jobs:
  update-homebrew:
    runs-on: ubuntu-latest

    steps:
      - name: Check out this repo
        uses: actions/checkout@v3

      - name: Get release version
        id: get_version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

      - name: Download source tarball
        run: |
          curl -L -o src.tar.gz https://github.com/${{ github.repository }}/archive/refs/tags/v${{ steps.get_version.outputs.version }}.tar.gz
          echo "sha256=$(shasum -a 256 src.tar.gz | cut -d ' ' -f 1)" >> $GITHUB_OUTPUT
        id: tarball

      - name: Clone Homebrew Tap
        run: |
          git clone https://github.com/YOUR_USERNAME/homebrew-tools.git tap
          cd tap
          FORMULA_PATH=Formula/github-fetch.rb

          cat <<EOF > $FORMULA_PATH
class GithubFetch < Formula
  desc "CLI tool to fetch GitHub repositories and save as JSON or CSV"
  homepage "https://github.com/${{ github.repository }}"
  url "https://github.com/${{ github.repository }}/archive/refs/tags/v${{ steps.get_version.outputs.version }}.tar.gz"
  sha256 "${{ steps.tarball.outputs.sha256 }}"
  license "MIT"

  depends_on "ruby"

  def install
    system "gem", "build", "github_fetcher.gemspec"
    system "gem", "install", "--install-dir", prefix, "--bindir", bin, "github_fetcher-${{ steps.get_version.outputs.version }}.gem"
  end

  test do
    system "#{bin}/github-fetch", "--help"
  end
end
EOF

          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"

          git add Formula/github-fetch.rb
          git commit -m "Update github-fetch to v${{ steps.get_version.outputs.version }}"
          git push
```

---

### ✅ Done!

Now when you tag a new release:

```bash
git tag v0.2.0
git push origin v0.2.0
```

✨ GitHub Actions will:

* Rebuild the tarball
* Compute the SHA
* Update `Formula/github-fetch.rb` in your `homebrew-tools` tap
* Commit and push the new formula

---

### 🧪 Test it:

```bash
brew update
brew upgrade github-fetch
```

Would you like this entire setup saved as a template project or starter repo for future CLI tools?
