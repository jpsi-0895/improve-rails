Integrating **CanCanCan** with **JWT (JSON Web Tokens)** for user authentication and role-based authorization in a Rails app involves a few key steps. Here's a comprehensive guide to get you started.

---

### 🏗️ **Step-by-Step Guide to Setup CanCanCan with JWT Authentication**

---

### 🛠️ **1. Set Up the Rails Project**

If you haven't already created the Rails project, follow these steps:

```bash
rails new project_manager_api --api -d postgresql
cd project_manager_api
bundle install
rails db:create
```

---

### 🔑 **2. Add Required Gems**

Add the following gems to your `Gemfile` for **CanCanCan**, **JWT**, and **Devise** (for authentication):

```ruby
gem 'devise'
gem 'cancancan'
gem 'jwt'
```

Run:

```bash
bundle install
```

---

### 👤 **3. Set Up Devise for Authentication**

Install **Devise** for user authentication:

```bash
rails generate devise:install
rails generate devise User
rails db:migrate
```

Configure `config/initializers/devise.rb` (optional) for JWT usage:

```ruby
# Add this if you want to use JWT for authentication
config.jwt do |jwt|
  jwt.secret_key = ENV['DEVISE_JWT_SECRET_KEY']  # Ensure you set this environment variable
end
```

### 📜 **4. JWT Authentication Setup**

Create a service for JWT authentication. This service will generate tokens on login and provide methods to verify them.

Create a new file in `app/services/authentication_service.rb`:

```ruby
class AuthenticationService
  def self.encode(user_id)
    payload = { user_id: user_id }
    JWT.encode(payload, ENV['DEVISE_JWT_SECRET_KEY'], 'HS256')
  end

  def self.decode(token)
    decoded_token = JWT.decode(token, ENV['DEVISE_JWT_SECRET_KEY'], true, { algorithm: 'HS256' })
    decoded_token[0]['user_id']
  rescue JWT::DecodeError
    nil
  end
end
```

---

### 🔐 **5. Add JWT to Devise**

We need to configure Devise to authenticate with JWT.

In `app/controllers/application_controller.rb`, add a helper to authenticate using JWT:

```ruby
class ApplicationController < ActionController::API
  include CanCan::ControllerAdditions

  before_action :authenticate_user!

  def authenticate_user!
    token = request.headers['Authorization']&.split(' ')&.last
    return render json: { error: 'Token missing or invalid' }, status: :unauthorized unless token

    user_id = AuthenticationService.decode(token)
    @current_user = User.find_by(id: user_id)
    render json: { error: 'Invalid token' }, status: :unauthorized unless @current_user
  end

  def current_user
    @current_user
  end
end
```

---

### 🛠️ **6. Add CanCanCan (Authorization)**

Now, let's set up **CanCanCan** to handle roles and permissions.

Create a `Role` model:

```bash
rails g model Role name:string
rails db:migrate
```

Add the association in `User`:

```ruby
class User < ApplicationRecord
  has_many :roles
end
```

You can then add roles and assign them to users in your seeds file or through the console.

Create a `Ability` class to define the permissions.

### `app/models/ability.rb`

```ruby
class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new  # guest user (not logged in)

    if user.roles.any? { |role| role.name == 'admin' }
      can :manage, :all  # Admin can manage everything
    elsif user.roles.any? { |role| role.name == 'manager' }
      can :read, Project
      can :create, Project
      can :update, Project
    else
      can :read, Project
    end
  end
end
```

---

### 📄 **7. Set Up User Controller for JWT Token Generation**

Now, we need to create the controller that handles user login and returns the JWT token.

Create a `SessionsController`:

```bash
rails g controller Sessions
```

In `app/controllers/sessions_controller.rb`:

```ruby
class SessionsController < ApplicationController
  skip_before_action :authenticate_user!, only: [:create]

  def create
    user = User.find_by(email: params[:email])
    if user && user.valid_password?(params[:password])
      token = AuthenticationService.encode(user.id)
      render json: { token: token, user: user }
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end
end
```

---

### 🔄 **8. Routes Setup**

Add the routes for authentication (login) and user management.

In `config/routes.rb`:

```ruby
devise_for :users, controllers: { sessions: 'sessions' }

# Define routes for your API resources
resources :projects
```

---

### 📋 **9. Protect Routes with CanCanCan**

Now, in your controllers, you can use **CanCanCan** to enforce authorization.

For example, in your `ProjectsController`:

```ruby
class ProjectsController < ApplicationController
  load_and_authorize_resource

  def index
    @projects = Project.all
    render json: @projects
  end

  def show
    render json: @project
  end

  # Other actions (create, update, destroy) follow the same pattern
end
```

---

### 🧪 **10. Test the System**

1. **Sign in to generate JWT token**:
   Send a POST request to `POST /sessions` with email and password.

   Example:

   ```bash
   curl -X POST http://localhost:3000/sessions \
   -d "email=user@example.com&password=yourpassword"
   ```

   This will return a token that you will use for subsequent requests.

2. **Use the token to access protected routes**:
   Send a GET request to `GET /projects` with the JWT token in the `Authorization` header.

   Example:

   ```bash
   curl -X GET http://localhost:3000/projects \
   -H "Authorization: Bearer <your_jwt_token>"
   ```

---

### 🎨 **11. Optional: Role Management Interface**

To make managing roles easier, you can add a basic admin interface for managing user roles. This could be done through a simple form to assign or remove roles from users.

---

### 🧑‍💻 **Conclusion**

This setup integrates:

* **JWT authentication** with **Devise** for secure token-based login.
* **CanCanCan** for role-based authorization and permissions.
* **Controller protection** using roles (Admin, Manager, User).

You now have a robust API that can authenticate users via JWT and authorize actions using **CanCanCan**. Would you like to see examples of other controllers or routes in action?
