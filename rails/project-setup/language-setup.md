Great! Here's a **ready-to-use Rails configuration template for a multi-locale (internationalized) app**. This setup ensures your application is ready for **multiple languages**, includes **locale switching**, and sets up your views and models to be translation-aware.

---

## 🌍 Step-by-Step Configuration for a Multi-Locale Rails App

---

### ✅ 1. **Add Locale Files**

Rails comes with English by default (`config/locales/en.yml`). Add additional locale files for other languages, like French, Spanish, Hindi, etc.

#### Example: `config/locales/fr.yml`

```yaml
fr:
  hello: "Bonjour"
  welcome: "Bienvenue à notre application"
```

#### Example: `config/locales/es.yml`

```yaml
es:
  hello: "Hola"
  welcome: "Bienvenido a nuestra aplicación"
```

---

### ✅ 2. **Update Global Config: `config/application.rb`**

Edit this file to set the default locale and load all locale files:

```ruby
module MyApp
  class Application < Rails::Application
    config.load_defaults 7.0

    # Set the default locale
    config.i18n.default_locale = :en

    # Auto-load all locale files in nested folders under config/locales
    config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]

    # Available locales in the app
    config.i18n.available_locales = [:en, :fr, :es]
  end
end
```

---

### ✅ 3. **Set Locale Per Request**

In `application_controller.rb`, add a method to switch the locale based on params, cookies, headers, etc.

```ruby
class ApplicationController < ActionController::Base
  before_action :set_locale

  def set_locale
    I18n.locale = params[:locale] || I18n.default_locale
  end

  # Optional: ensure all generated URLs include the locale
  def default_url_options
    { locale: I18n.locale }
  end
end
```

---

### ✅ 4. **Add Locale to Routes**

Wrap your routes in a `scope` block to include locale in the URL:

```ruby
# config/routes.rb
Rails.application.routes.draw do
  scope "(:locale)", locale: /en|fr|es/ do
    root "home#index"
    resources :articles
  end
end
```

Now your URLs can look like:

* `/en/articles`
* `/fr/articles`
* `/es/articles`

---

### ✅ 5. **Add Translations to Views**

Use the `t` helper method in your views instead of hardcoded text:

```erb
<!-- app/views/home/index.html.erb -->
<h1><%= t('welcome') %></h1>
```

Rails will use the current locale to choose the right translation:

* English: "Welcome to our app"
* French: "Bienvenue à notre application"
* Spanish: "Bienvenido a nuestra aplicación"

---

### ✅ 6. **Optional: Add Language Switcher**

In your layout view (`app/views/layouts/application.html.erb`):

```erb
<p>
  <%= link_to "English", url_for(locale: :en) %> |
  <%= link_to "Français", url_for(locale: :fr) %> |
  <%= link_to "Español", url_for(locale: :es) %>
</p>
```

---

### ✅ 7. **I18n in Models & Validations**

Use I18n for validation error messages in models:

```ruby
validates :title, presence: true
```

In your `config/locales/fr.yml`:

```yaml
fr:
  activerecord:
    attributes:
      article:
        title: "Titre"
    errors:
      models:
        article:
          attributes:
            title:
              blank: "ne peut pas être vide"
```

---

### ✅ 8. **Gem for Richer Translation UI (Optional)**

Add [`i18n-tasks`](https://github.com/glebm/i18n-tasks) to manage translations:

```ruby
# Gemfile
gem 'i18n-tasks'
```

Install and use:

```bash
bundle install
i18n-tasks health
i18n-tasks missing
i18n-tasks unused
```

---

### ✅ Final Folder Structure (Locales)

```
config/
  locales/
    en.yml
    fr.yml
    es.yml
    models/
      article.en.yml
      article.fr.yml
    views/
      home.en.yml
      home.fr.yml
```

---

## ✅ Summary of Key Steps

| Step                         | Purpose                                |
| ---------------------------- | -------------------------------------- |
| `config.i18n.default_locale` | Sets app's default language            |
| `available_locales`          | Declares supported languages           |
| `set_locale` method          | Allows dynamic locale switching        |
| Route scope `(:locale)`      | Makes locale part of the URL           |
| `t('key')` in views          | Translates UI text                     |
| `i18n-tasks` gem             | Helps manage missing/untranslated keys |

---

Would you like me to generate sample translation files for a specific app section like login/signup or dashboard?
