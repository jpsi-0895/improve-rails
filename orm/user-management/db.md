Designing a **User, Roles, and Permissions** system in Rails involves setting up a normalized and flexible schema that can scale for large applications. Here's a complete **database design**, including **models, associations**, and **migration structure**, following best practices and adhering to the **Single Responsibility Principle**.

---

## ✅ High-Level Relationships

* **User** has many **Roles**
* **Role** has many **Permissions**
* **User** gets permissions through assigned roles (many-to-many)

---

## 📦 Database Schema

### Tables:

* `users`
* `roles`
* `permissions`
* `user_roles` (join table)
* `role_permissions` (join table)

---

## 🔧 Migrations

### 1. `users` Table

```bash
rails generate model User name:string email:string
```

```ruby
# db/migrate/xxxx_create_users.rb
create_table :users do |t|
  t.string :name
  t.string :email, null: false, index: { unique: true }

  t.timestamps
end
```

---

### 2. `roles` Table

```bash
rails generate model Role name:string
```

```ruby
# db/migrate/xxxx_create_roles.rb
create_table :roles do |t|
  t.string :name, null: false, index: { unique: true }

  t.timestamps
end
```

---

### 3. `permissions` Table

```bash
rails generate model Permission name:string action:string subject:string
```

```ruby
# db/migrate/xxxx_create_permissions.rb
create_table :permissions do |t|
  t.string :name
  t.string :action   # e.g., "read", "create", "update"
  t.string :subject  # e.g., "Product", "User"

  t.timestamps
end
```

---

### 4. `user_roles` Table (Join Table)

```bash
rails generate model UserRole user:references role:references
```

```ruby
# db/migrate/xxxx_create_user_roles.rb
create_table :user_roles do |t|
  t.references :user, null: false, foreign_key: true
  t.references :role, null: false, foreign_key: true

  t.timestamps
end
```

---

### 5. `role_permissions` Table (Join Table)

```bash
rails generate model RolePermission role:references permission:references
```

```ruby
# db/migrate/xxxx_create_role_permissions.rb
create_table :role_permissions do |t|
  t.references :role, null: false, foreign_key: true
  t.references :permission, null: false, foreign_key: true

  t.timestamps
end
```

---

## 📘 Models & Associations

### `User`

```ruby
class User < ApplicationRecord
  has_many :user_roles
  has_many :roles, through: :user_roles
  has_many :permissions, through: :roles
end
```

---

### `Role`

```ruby
class Role < ApplicationRecord
  has_many :user_roles
  has_many :users, through: :user_roles

  has_many :role_permissions
  has_many :permissions, through: :role_permissions
end
```

---

### `Permission`

```ruby
class Permission < ApplicationRecord
  has_many :role_permissions
  has_many :roles, through: :role_permissions
end
```

---

### `UserRole`

```ruby
class UserRole < ApplicationRecord
  belongs_to :user
  belongs_to :role
end
```

---

### `RolePermission`

```ruby
class RolePermission < ApplicationRecord
  belongs_to :role
  belongs_to :permission
end
```

---

## 🔐 Usage Example

```ruby
# Assign role to user
user = User.find(1)
admin_role = Role.find_by(name: 'admin')
user.roles << admin_role

# Assign permission to role
permission = Permission.create!(action: 'read', subject: 'Product')
admin_role.permissions << permission

# Check if user has permission
user.permissions.exists?(action: 'read', subject: 'Product')
```

---

## ✅ Optional: Helper Method in User Model

```ruby
def can?(action, subject)
  permissions.exists?(action: action.to_s, subject: subject.to_s)
end
```

Usage:

```ruby
user.can?(:read, 'Product')  # => true or false
```

---

Would you like me to add **seed data**, **admin interface**, or integrate with a gem like `cancancan` or `pundit` for authorization logic?
