Here’s a **step-by-step guide to set up a new Rails project** with Active Storage, Devise, and a basic user/project structure — perfect for the project management system you described.

---

## 🧱 **1. Prerequisites**

Make sure you have the following installed:

```bash
ruby -v       # Ruby 3.x
rails -v      # Rails 7.x
node -v       # Node.js
yarn -v       # Yarn
postgres -V   # PostgreSQL
```

---

## 🚀 **2. Create the Rails Project**

```bash
rails new project_manager_app -d postgresql
cd project_manager_app
bundle install
rails db:create
```

---

## 👤 **3. Add Devise for User Authentication**

```bash
bundle add devise
rails generate devise:install
rails generate devise User
rails db:migrate
```

✅ Devise will handle:

* Sign in/out
* Registration
* Password resets
* Email validations (optional)

---

## 📂 **4. Add Active Storage for File Uploads**

```bash
rails active_storage:install
rails db:migrate
```

In your `Project` model:

```ruby
has_many_attached :documents
```

---

## 📁 **5. Generate Models for Roles, Projects, Assignments**

```bash
rails g model Role name:string
rails g model RoleAssignment user:references role:references

rails g model Project name:string description:text manager:references{User} token_amount:decimal total_cost:decimal finalized_at:datetime delivery_date:date

rails g model ProjectAssignment user:references project:references
rails db:migrate
```

---

## 🔧 **6. Set Up Associations**

### `user.rb`

```ruby
has_many :managed_projects, class_name: 'Project', foreign_key: 'manager_id'
has_many :project_assignments
has_many :projects, through: :project_assignments
has_many :role_assignments
has_many :roles, through: :role_assignments
```

### `project.rb`

```ruby
belongs_to :manager, class_name: 'User'
has_many :project_assignments
has_many :users, through: :project_assignments
has_many_attached :documents
```

---

## 🌐 **7. Setup Routes & Controller for Projects**

```bash
rails g controller Projects
```

### `config/routes.rb`

```ruby
devise_for :users
resources :projects
root "projects#index"
```

---

## 🧪 **8. Add Seed Data (Optional)**

```ruby
# db/seeds.rb
admin = User.create!(email: 'admin@example.com', password: 'password')
role = Role.create!(name: 'admin')
RoleAssignment.create!(user: admin, role: role)
```

Then:

```bash
rails db:seed
```

---

## 💡 **9. Run the App**

```bash
rails server
```

Visit: [http://localhost:3000](http://localhost:3000)

---

Would you like me to generate the full model/controller code or provide a GitHub-ready starter template?
