Here’s how you can extend the functionality for **user authentication** and **authorization** using **CanCanCan** and **JWT** by adding examples for various controllers like `ProjectsController`, `UsersController`, and additional routes.

### 🚀 **Complete Example with Controllers and Routes**

---

## 1. **Projects Controller (Protected by CanCanCan)**

The `ProjectsController` will allow admins and managers to manage projects, and regular users to view them. We'll protect the actions using **CanCanCan** and JWT.

### `app/controllers/projects_controller.rb`

```ruby
class ProjectsController < ApplicationController
  # CanCanCan load and authorize project resource automatically
  load_and_authorize_resource

  # GET /projects
  def index
    @projects = Project.all
    render json: @projects
  end

  # GET /projects/:id
  def show
    render json: @project
  end

  # POST /projects
  def create
    @project = Project.new(project_params)
    @project.manager = current_user

    if @project.save
      render json: @project, status: :created
    else
      render json: @project.errors, status: :unprocessable_entity
    end
  end

  # PATCH/PUT /projects/:id
  def update
    if @project.update(project_params)
      render json: @project
    else
      render json: @project.errors, status: :unprocessable_entity
    end
  end

  # DELETE /projects/:id
  def destroy
    @project.destroy
    head :no_content
  end

  private

  def project_params
    params.require(:project).permit(:name, :description, :token_amount, :total_cost, :delivery_date, :finalized_at, documents: [])
  end
end
```

### 📋 **Explanation:**

* `load_and_authorize_resource`: Automatically loads the resource (`@project`) and checks if the current user is authorized to access it based on the permissions defined in the `Ability` class.
* Actions like `create`, `update`, and `destroy` will only be allowed based on the user's role. For example, only users with an **admin** or **manager** role can create, update, or destroy projects.
* The `documents: []` part of `project_params` allows uploading multiple files using **Active Storage**.

---

## 2. **Users Controller (For User Management)**

You may need an API controller to manage users, for example, listing users or changing roles.

### `app/controllers/users_controller.rb`

```ruby
class UsersController < ApplicationController
  load_and_authorize_resource
  
  # GET /users
  def index
    @users = User.all
    render json: @users
  end

  # GET /users/:id
  def show
    render json: @user
  end

  # PUT/PATCH /users/:id/roles
  def update_role
    if current_user.has_role?(:admin) && params[:role_id].present?
      role = Role.find_by(id: params[:role_id])
      if role
        @user.roles << role unless @user.roles.include?(role)
        render json: @user, status: :ok
      else
        render json: { error: 'Role not found' }, status: :not_found
      end
    else
      render json: { error: 'Unauthorized to update roles' }, status: :unauthorized
    end
  end

  private

  def user_params
    params.require(:user).permit(:name, :email)
  end
end
```

### 📋 **Explanation:**

* The `update_role` method lets an **admin** assign new roles to users. It checks if the current user has the `admin` role before assigning roles.
* We use `load_and_authorize_resource` to ensure that the current user has the required permissions to perform actions on the user model.

---

## 3. **Sessions Controller (For JWT Login)**

The `SessionsController` allows users to log in and receive a JWT token that can be used for authenticated API calls.

### `app/controllers/sessions_controller.rb`

```ruby
class SessionsController < ApplicationController
  skip_before_action :authenticate_user!, only: [:create]

  # POST /sessions
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

### 📋 **Explanation:**

* The `create` action accepts `email` and `password`, and if they are valid, it generates a JWT token using the `AuthenticationService.encode` method we defined earlier.
* The token is returned in the response, and this token must be used in the `Authorization` header for subsequent requests.

---

## 4. **Routes Setup**

Now, set up your routes for user sessions, projects, and user management.

### `config/routes.rb`

```ruby
Rails.application.routes.draw do
  # Devise routes for user registration, password reset, etc.
  devise_for :users, controllers: { sessions: 'sessions' }

  # Project routes (CRUD operations)
  resources :projects

  # User routes for listing and showing users
  resources :users, only: [:index, :show] do
    member do
      patch :update_role  # Allow role assignment only by admin
    end
  end

  # Root route (can be anything, here it shows projects list)
  root 'projects#index'
end
```

### 📋 **Explanation:**

* The `resources :projects` route creates the standard RESTful routes for the project resources.
* The `resources :users` route provides `index` and `show` actions for users, while `update_role` allows admins to modify user roles.

---

## 5. **Ability Class (CanCanCan)**

Make sure your `Ability` class defines who can do what based on roles.

### `app/models/ability.rb`

```ruby
class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new  # guest user (not logged in)

    if user.roles.any? { |role| role.name == 'admin' }
      can :manage, :all  # Admin can manage everything
    elsif user.roles.any? { |role| role.name == 'manager' }
      can :manage, Project
      can :read, User
    else
      can :read, Project
      cannot :manage, Project
    end
  end
end
```

### 📋 **Explanation:**

* Admin users have permission to manage everything (`can :manage, :all`).
* Managers can manage projects but can only view users (`can :manage, Project` and `can :read, User`).
* Regular users can only read projects (`can :read, Project`).

---

## 6. **Test the API**

1. **Create JWT Token:**
   Send a POST request to `/sessions` with the user's email and password:

   ```bash
   curl -X POST http://localhost:3000/sessions \
   -d "email=user@example.com&password=yourpassword"
   ```

   This will return a JWT token that you can use in future requests.

2. **Access Protected Routes (Projects) Using JWT:**
   To access the projects, pass the JWT in the `Authorization` header:

   ```bash
   curl -X GET http://localhost:3000/projects \
   -H "Authorization: Bearer <your_jwt_token>"
   ```

3. **Access User Management Routes (Admin Only):**
   If you're an admin, you can manage user roles:

   ```bash
   curl -X PATCH http://localhost:3000/users/:id/update_role \
   -H "Authorization: Bearer <your_jwt_token>" \
   -d "role_id=<role_id>"
   ```

---

### 💡 **Conclusion**

With this setup:

* **JWT Authentication** handles user login and token-based API authentication.
* **CanCanCan** ensures that users can only access resources they're authorized for based on roles.
* **Admin users** can manage projects, users, and assign roles.
* **Managers** can create and manage projects, while **regular users** can only view projects.

Let me know if you'd like further examples or want to explore something specific!
