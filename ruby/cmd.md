Perfect! Let's turn this script into a proper **command-line tool** with flags using Ruby’s built-in `OptionParser` library.

---

## 🛠️ Features We'll Add:

* `--user` (GitHub username)
* `--token` (GitHub API token, optional)
* `--count` (number of repos to fetch)
* `--format` (json or csv)
* `--output` (optional output filename)

---

## 📄 Final Script: `github_fetcher_cli.rb`

```ruby
require 'net/http'
require 'json'
require 'csv'
require 'optparse'

# --------- GitHub Repo Fetching ---------
def github_repos_lazy(user, token: nil)
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

# --------- Output Helpers ---------
def save_to_json(data, filename)
  File.write(filename, JSON.pretty_generate(data))
  puts "✅ Saved to #{filename}"
end

def save_to_csv(data, filename)
  CSV.open(filename, "w") do |csv|
    csv << ["Repository Name", "URL"]
    data.each { |repo| csv << [repo[:name], repo[:url]] }
  end
  puts "✅ Saved to #{filename}"
end

# --------- CLI Options ---------
options = {}
OptionParser.new do |opts|
  opts.banner = "Usage: ruby github_fetcher_cli.rb [options]"

  opts.on("-u", "--user USER", "GitHub username") { |v| options[:user] = v }
  opts.on("-t", "--token TOKEN", "GitHub API token (optional)") { |v| options[:token] = v }
  opts.on("-c", "--count COUNT", Integer, "Number of repositories to fetch") { |v| options[:count] = v }
  opts.on("-f", "--format FORMAT", "Output format: json or csv") { |v| options[:format] = v.downcase }
  opts.on("-o", "--output FILE", "Output file name (optional)") { |v| options[:output] = v }
end.parse!

# --------- Validate Inputs ---------
if options[:user].nil? || options[:count].nil? || !%w[json csv].include?(options[:format])
  puts "❌ Missing required options. Use --help to see available flags."
  exit
end

# --------- Run & Save ---------
repos = github_repos_lazy(options[:user], token: options[:token])
           .take(options[:count])
           .to_a

filename = options[:output] || "repos.#{options[:format]}"

if options[:format] == "json"
  save_to_json(repos, filename)
else
  save_to_csv(repos, filename)
end
```

---

## ▶️ How to Use:

### ✅ Run from Terminal:

```bash
ruby github_fetcher_cli.rb \
  --user octocat \
  --count 10 \
  --format csv \
  --token YOUR_GITHUB_TOKEN \
  --output my_repos.csv
```

### 🧪 Or try with just public data:

```bash
ruby github_fetcher_cli.rb --user rails --count 5 --format json
```

---

Would you like to package this into a Ruby gem or make it installable as a CLI command (e.g., `github-fetch`)?
