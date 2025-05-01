If you're working with **ActiveRecord in Ruby on Rails** and want to connect to an **existing database**, here's a step-by-step guide to integrate it cleanly — without Rails generating new tables or migrations.

---

## ✅ 1. **Connect Rails to the Existing Database**

Update your `config/database.yml` with the correct credentials for your existing database:

```yaml
development:
  adapter: mysql2         # or 'postgresql' if using Postgres
  database: your_db_name
  username: your_user
  password: your_password
  host: localhost
```

Then run:

```bash
rails db:connect
```

or just start the Rails console:

```bash
rails console
```

---

## ✅ 2. **Create Models Without Migrations**

You can define models that match your existing tables, without generating new migrations:

```bash
rails generate model User --skip-migration
```

Then edit the model:

```ruby
# app/models/user.rb
class User < ApplicationRecord
  self.table_name = 'users'     # optional if table name matches
  self.primary_key = 'id'       # specify if your PK is not 'id'

  # Add associations, validations, etc.
end
```

---

## ✅ 3. **Verify Schema (Optional)**

You can introspect the schema via the Rails console:

```ruby
User.columns.map(&:name)
```

Or generate a model based on the existing table with schema comments:

```bash
annotate
```

(You may need the [`annotate`](https://github.com/ctran/annotate_models) gem for that.)

---

## ✅ 4. **Use Legacy Naming Conventions (if needed)**

If your table or column names don't follow Rails conventions, override them:

```ruby
class LegacyOrder < ApplicationRecord
  self.table_name = 'ORDERS_TABLE'
  self.primary_key = 'ORDER_ID'

  def readonly?
    true  # if you only want to read from the legacy table
  end
end
```

---

## ✅ 5. **Don't Run Migrations You Don’t Need**

If you generated any unnecessary migrations, delete them. To avoid creating them at all:

```bash
rails generate model Customer --skip-migration
```

---

## ✅ 6. (Optional) Disable STI (Single Table Inheritance)

If your table has a `type` column but it's **not** used for STI, disable it:

```ruby
self.inheritance_column = nil
```

---

## 🧠 Summary

| Step                     | What to do                              |
|--------------------------|------------------------------------------|
| Connect DB               | Edit `database.yml`                      |
| Generate model           | Use `--skip-migration`                  |
| Override table/key names | `self.table_name`, `self.primary_key`   |
| Disable STI              | `self.inheritance_column = nil`         |
| Read-only table          | `def readonly?; true; end`              |

---

Would you like help mapping a full schema into models or building associations from your existing DB structure?