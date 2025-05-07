Rails primarily follows the **Model-View-Controller (MVC)** design pattern—but that’s just the foundation. Rails also integrates or encourages several other design patterns under the hood. Here's a breakdown:

---

## ✅ 1. **Model-View-Controller (MVC)** — **Main Architectural Pattern**

### 🚀 What it means in Rails:

* **Model** (`app/models`): Business logic, validations, database interaction (ActiveRecord).
* **View** (`app/views`): Templating for HTML/JSON output (ERB, HAML).
* **Controller** (`app/controllers`): Orchestrates request flow between models and views.

🧠 **Purpose**: Separation of concerns — keeps business logic (model), user interface (view), and user input control (controller) separate.

---

## 🧩 2. **Active Record Pattern** (in Models)

### Rails Models use the **Active Record** pattern:

* Each model corresponds to a table in the database.
* The model object wraps both **data access** and **business logic**.

```ruby
class User < ApplicationRecord
  # this class represents both the user row and the logic around users
end
```

> ❗ While convenient, this sometimes leads to “fat models” if abused.

---

## 🧬 3. **Convention over Configuration**

While not a "pattern" in the classic GoF sense, Rails enforces this **architectural principle** to reduce decision fatigue.

Example: You don’t configure table names, routes, or controller-method mappings if you follow naming conventions.

---

## 🛠 4. Other Patterns Rails Uses or Encourages

| Pattern                    | Where Used in Rails                                 | Purpose                                         |
| -------------------------- | --------------------------------------------------- | ----------------------------------------------- |
| **Singleton**              | Rails config and initializers (`Rails.application`) | Ensures only one instance of config/environment |
| **Observer** (deprecated)  | Originally in ActiveRecord callbacks                | Not widely used anymore                         |
| **Decorator**              | Through gems like `Draper`                          | Enhances object presentation logic              |
| **Command/Service Object** | In `app/services` (you build them)                  | Encapsulates single business operations         |
| **Builder**                | `form_with`, `ActiveModel::Builder`                 | Builds complex forms/objects step by step       |
| **Template Method**        | `ActiveRecord` lifecycle callbacks                  | Defines skeleton with overridable steps         |

---

## 🏁 Summary

Rails is built around:

* **MVC** for structure
* **Active Record** for ORM
* **Convention over Configuration** for simplicity

And it **supports extensibility** via patterns like Service Objects, Repositories, Decorators, and more.

---

Would you like an illustration of how Rails request flow maps to these patterns?
