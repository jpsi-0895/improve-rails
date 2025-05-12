Here's a cheat sheet of **important Rails commands** that help you quickly create the essential components of a Rails application. These commands use Rails generators to scaffold models, controllers, migrations, and more.


## 🔧 **Core Rails Project Setup**

### Create a New Rails Project:

```bash
rails new app_name
```

## 📦 **Model Generation**

### Create a Model (with optional fields):

```bash
rails generate model ModelName field1:type field2:type
```

#### Example:

```bash
rails generate model Article title:string body:text
```

* This creates:

  * A model file: `app/models/article.rb`
  * A migration: `db/migrate/xxxx_create_articles.rb`
  * Test files (unless skipped)
  * Schema entries once migrated


## 🗃️ **Migration**

### Create a Standalone Migration:

```bash
rails generate migration MigrationName
```

#### Example:

```bash
rails generate migration AddPublishedToArticles published:boolean
```

### Run Migrations:

```bash
rails db:migrate
```


## 🌐 **Controller**

### Create a Controller with Actions:

```bash
rails generate controller ControllerName action1 action2
```

#### Example:

```bash
rails generate controller Articles index show new create
```

* Creates:

  * Controller: `app/controllers/articles_controller.rb`
  * Views: `app/views/articles/index.html.erb`, etc.
  * Routes (you may need to add manually)


## 📄 **Scaffold (Full CRUD)**

### Generate Full MVC Stack for a Resource:

```bash
rails generate scaffold ModelName field1:type field2:type
```

#### Example:

```bash
rails generate scaffold Product name:string price:decimal
```

* Creates:

  * Model
  * Controller
  * Views
  * Migration
  * Routes entry
  * Test files

## 🛣️ **Routing**

### View All Routes:

```bash
rails routes
```

### Manually Add a Resource Route:

In `config/routes.rb`:

```ruby
resources :articles
```


## 📂 **Database**

### Create the Database:

```bash
rails db:create
```

### Drop the Database:

```bash
rails db:drop
```

### Reset the Database:

```bash
rails db:reset
```

## 🔍 **Console & Server**

### Start Rails Console (for interactive debugging/testing):

```bash
rails console
```

### Start Rails Server:

```bash
rails server
```

## 🧪 **Tests (RSpec / Minitest)**

### Run All Tests:

```bash
rails test          # for Minitest
rspec               # for RSpec (if installed)
```

## 🧰 **Other Useful Generators**

### Generate a Mailer:

```bash
rails generate mailer UserMailer
```

### Generate a Job:

```bash
rails generate job CleanupOldData
```

### Generate a Channel (for WebSockets/ActionCable):

```bash
rails generate channel Chat
```
## 📦 **Install Common Gems After Setup**

### Add Devise (Authentication):

```bash
rails generate devise:install
rails generate devise User
```

### Add Pundit (Authorization):

```bash
rails generate pundit:install
```

### Add ActiveStorage (File Uploads):

```bash
rails active_storage:install
rails db:migrate
```

## 🎨 **Assets & Frontend**

### Precompile Assets:

```bash
rails assets:precompile
```

### Tailwind CSS (if installed):

```bash
rails generate tailwindcss:install
```

## 🧠 Summary of Key Generators

| Component       | Command Example                        |
| --------------- | -------------------------------------- |
| Model           | `rails g model`                        |
| Controller      | `rails g controller`                   |
| Scaffold        | `rails g scaffold`                     |
| Migration       | `rails g migration`                    |
| Mailer          | `rails g mailer`                       |
| Job             | `rails g job`                          |
| Resource Routes | `resources :model_name` in `routes.rb` |
| Console         | `rails console`                        |
| Server          | `rails server`                         |


Would you like a downloadable cheat sheet or examples for specific components like Devise, Pundit, or API-only controllers?
