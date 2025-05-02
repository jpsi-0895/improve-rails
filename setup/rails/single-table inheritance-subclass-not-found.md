Ah yes — you’ve just bumped into a **classic Rails gotcha** involving **Single Table Inheritance (STI)** and the use of a column named `type`. 😬

---

### ⚠️ What's Happening?

Rails uses a special column called `type` in any model for **STI** — that means it thinks you're trying to subclass `Product` using a class called `'page'` (based on the value in the `type` column of one of the records).

But you probably didn’t intend that — your `products` table just happens to have a `type` column for something else (like "page", "physical", "digital", etc.), right?

---

### ✅ How to Fix It

#### Option 1: 🔄 Rename the `type` column

This is the **recommended** fix, if you can change the database:

```bash
# Example migration
rename_column :products, :type, :product_type
```

Then update any code using `product.type` to `product.product_type`.

---

#### Option 2: 🛠️ Tell Rails to ignore STI for this model

If renaming isn’t an option (e.g., legacy database), you can **override the STI column** in your model:

```ruby
# app/models/product.rb
class Product < ApplicationRecord
  self.inheritance_column = :_type_disabled
end
```

This tells Rails: “Hey, I’m not using STI here — leave the `type` column alone.”

You can also rename the inheritance column to something else entirely:

```ruby
self.inheritance_column = 'custom_type'
```

---

### 💡 Example

Let’s say your `products` table has:

| id | name       | type   |
|----|------------|--------|
| 1  | PageX      | page   |
| 2  | Widget     | widget |

Without fixing it, Rails looks for a `Page` class — boom 💥 error.

With `self.inheritance_column = :_type_disabled`, Rails treats `type` like any other column.

---

Let me know if you want help writing the migration to rename `type`, or if you're working with other legacy table quirks 👀