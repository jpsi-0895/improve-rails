Great! Let's build a **small Ruby script** that:

✅ Fetches paginated data from a **real API** (GitHub),
✅ Uses **lazy enumerators** for memory efficiency,
✅ Processes results (e.g., repo names),
✅ And prints them cleanly.

---

## 🌐 Example: Lazy Looping Through GitHub Repositories (User)

### 🔧 Prerequisites:

* Ruby ≥ 2.5
* GitHub username
* No auth needed for public requests (but limited to 60/hr)

---

### 📄 Full Ruby Script (Lazy Pagination from GitHub API)

```ruby
require 'net/http'
require 'json'

# Create a lazy enumerator to fetch GitHub repositories page by page
def github_repos_lazy(user)
  Enumerator.new do |yielder|
    page = 1

    loop do
      uri = URI("https://api.github.com/users/#{user}/repos?per_page=5&page=#{page}")
      response = Net::HTTP.get(uri)
      repos = JSON.parse(response)

      break if repos.empty?

      repos.each { |repo| yielder << repo["name"] }
      page += 1
    end
  end.lazy
end

# Use it
puts "Enter GitHub username:"
user = gets.chomp

puts "\nFetching repos for '#{user}'..."

github_repos_lazy(user)
  .take(10) # Limit how many repos we fetch
  .each_with_index do |repo_name, i|
    puts "#{i + 1}. #{repo_name}"
  end
```

---

### 🔍 What It Does:

* Asks for a GitHub username
* Lazily loads 5 repos per page using GitHub API
* Stops after 10 results (you can change this)
* Efficient, readable, and scalable

---

### 🚀 To Run:

Save as `github_lazy.rb` and run:

```bash
ruby github_lazy.rb
```

---

Would you like to add GitHub API token support or save results to a CSV or JSON file?

Awesome! Let’s enhance the script by adding:

1. ✅ **GitHub API token support** (for higher rate limits: 5000/hr vs. 60/hr),
2. ✅ **Saving results to a JSON or CSV file**.

---

## 🧩 Updated Features:

* Token-based GitHub API access
* Saves output to a file (`repos.json` or `repos.csv`)

---

## 💾 Full Script With API Token + File Saving

```ruby
require 'net/http'
require 'json'
require 'csv'

def github_repos_lazy(user, token: nil)
  Enumerator.new do |yielder|
    page = 1

    loop do
      uri = URI("https://api.github.com/users/#{user}/repos?per_page=5&page=#{page}")
      request = Net::HTTP::Get.new(uri)
      request["Authorization"] = "token #{token}" if token

      response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
        http.request(request)
      end

      repos = JSON.parse(response.body)
      break if repos.empty?

      repos.each { |repo| yielder << { name: repo["name"], url: repo["html_url"] } }
      page += 1
    end
  end.lazy
end

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

# 🧑‍💻 User input
puts "Enter GitHub username:"
user = gets.chomp

puts "Enter your GitHub API token (press Enter to skip):"
token = gets.chomp
token = nil if token.empty?

puts "How many repositories do you want to fetch?"
count = gets.chomp.to_i

puts "Choose file format to save: (1) JSON or (2) CSV"
format = gets.chomp

# 🚀 Fetch data
repos = github_repos_lazy(user, token: token).take(count).to_a

# 💾 Save file
if format == "1"
  save_to_json(repos, "repos.json")
elsif format == "2"
  save_to_csv(repos, "repos.csv")
else
  puts "❌ Invalid file format option."
end
```

---

### 📦 To Run:

1. Save as `github_fetcher.rb`
2. Run with: `ruby github_fetcher.rb`
3. Follow the prompts

---

### 🔐 Notes:

* GitHub API token: generate it at [github.com/settings/tokens](https://github.com/settings/tokens) (no scopes needed for public repos).
* File output will be in your working directory.

Would you like to turn this into a command-line tool with flags (`--user`, `--format`, etc.)?
