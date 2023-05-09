## Ruby-setup

- Create a new application
  `rails new . --database postgresql`

- Update the postgres user and password in config/database.yml:

```
 username: postgres
password: Gracewanjiru1573
```

- Create the database
  `rails db:create`

- Create a scaffold for the database tables
  `rails g scaffold table_name`

- Create the routes for the application

- Create the FKs for the tables in the migration files
  `rails g migration AddColumnToTable table:references`

- Make sure you create the tables first before creating the FKs

- Run the migration files
  `rails db:migrate`

- Create the associations in the models.

- Go to the routes and create the namespaced routes:

```
  namespace :api do
  namespace :v1 do
  resources :table_name
  end
  end
```

- Move the controllers to the api/v1 folder

- Add cors rb gem to the gemfile
  `gem 'rack-cors'`

- Run bundle install

- Create a file in the config/initializers/cors.rb

```
  Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
  origins "\*"

      resource "*",
        headers: :any,
        methods: [:get, :post, :put, :patch, :delete, :options, :head]

end
end
```

- Add the following gems

```
gem "bcrypt"
gem "rack-cors"
gem "active_model_serializers"
gem "jwt"
```

- Install the gems
  `bundle install`

- In the config/initializers/cors.rb file add the following:

```
  Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
  origins '\*',
  resource '\*',
  headers: :any,
  methods: %i[get post put patch delete options head]

  end
  end
```

- Create the user model with the following attributes
  `rails g model User name email password_digest`

- Create the user serializer to serialize our api and choose what we want displayed on our endpoints
  `rails g serializer User`

- Create the user controller
  `rails g controller api/v1/Users`

  - Migrate the database
    `rails db:migrate`

  - We will need to use the has_secure_password macros to make sure that the password is hashed using the bcrypt gem . Add has_secure_password to app/models/user.rb.

  - First , we code a create action in the users controller that will allow for signing up .The create action will be used to create a new user. Add this to the user's controller in api/v1/users_controller.rb

```
def create
 user = User.new(user_params)
 if user.save
 render json: user
  else render json: {error: "Error creating user"}
  end
   end

private
 def user_params
 params.require(:user).permit(:username, :email, :password)
  end
```

- In the user serializer , change the attributes to be displayed . So you can only see the email and the username . Add this to the user serializer in app/serializers/user_serializer.rb

```
class UserSerializer < ActiveModel::Serializer
 attributes  :username, :email
 end
```
