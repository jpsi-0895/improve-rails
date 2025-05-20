Let's break down this Rails route:

```ruby
get '/category/:slug', to: 'blogs#category_wise_blogs_list', as: :all_blogs
```

---

### 🔍 Full Explanation

#### 1. `get '/category/:slug'`

* This defines a **GET route**.
* It matches URLs like:

  * `/category/technology`
  * `/category/rails`
* The `:slug` is a **dynamic segment**, meaning it's a variable part of the URL (like a category name or slug).
* Whatever value appears in place of `:slug` will be **available as `params[:slug]`** inside the controller.

---

#### 2. `to: 'blogs#category_wise_blogs_list'`

* This tells Rails to route the request to the **`category_wise_blogs_list` action** in the **`BlogsController`**.
* When someone visits `/category/something`, it calls:

  ```ruby
  class BlogsController < ApplicationController
    def category_wise_blogs_list
      slug = params[:slug]
      # You can now use slug to fetch the relevant category and its blogs
    end
  end
  ```

---

#### 3. `as: :all_blogs`

* This sets a **named route helper** called `all_blogs_path` or `all_blogs_url`.
* You can generate the path like this:

  ```ruby
  all_blogs_path('technology') 
  # => "/category/technology"
  ```
* Very useful in views or controllers:

  ```erb
  <%= link_to 'Tech Blogs', all_blogs_path('technology') %>
  ```

---

### ✅ Summary

This route:

| Part                               | Meaning                                     |
| ---------------------------------- | ------------------------------------------- |
| `/category/:slug`                  | Matches URLs like `/category/rails`         |
| `'blogs#category_wise_blogs_list'` | Calls that method in `BlogsController`      |
| `params[:slug]`                    | Gets the category slug from the URL         |
| `as: :all_blogs`                   | Lets you use `all_blogs_path(slug)` in code |

---

Let me know if you want help writing the `category_wise_blogs_list` action itself!
