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

- Finally , add a route for our creat action , This will be done in the config/routes.rb . Add this code inside the routes.rb file

```
Rails.application.routes.draw do
 namespace :api do
 namespace :v1 do
 resources :users, only: [:create]
 end
 end
end

```

- Test using postman. Start the server and send a post request to http://localhost:3000/api/v1/users with the following data in the body:

```

{
"user": {
"username": "test",
"email": "test@gmail.com
"password": "test"
}
}

```

- You should get a response with the user's email and username.

```

{
"username": "test",
"email": "test@gmail.com"
}

```

- To add validation to the email so that a user cannot sign up with an invalid email address , add the following to the user model in app/models/user.rb

```

validates :email, presence: true, uniqueness: true

```

- To add validation to the password so that a user cannot sign up with a password that is less than 6 characters , add the following to the user model in app/models/user.rb

```
validates :password, length: {minimum: 6}

```

- Next, create a token for the user when they sign up. Use the jwt gem to create the token. To do this , add an encode_token method to the application controller

```
  def encode_token(payload)
    JWT.encode(payload, 'my_s3cr3t')
  end

```

We now modify our create action to give us a token everytime we sign up a user

```

  def create
  user = User.new(user_params)
  if user.save
    token = encode_token({ user_id: user.id })
    render json: { user: UserSerializer.new(user), jwt: token }
  else
    render json: { error: 'Error creating user' }
  end
end

```

- Try signing up a user again and you should get a token in the response

```

{
"user": {
"username": "test2",
"email":test2@gmail.com",
"password": "password2"
}
}

```

- The output should be

```

{ "user":
{ "username": "test2",
"email": "test2@gmail.com"
},
 "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0fQ.oFQeWUjCQx5X2vxlAH_bzAKijAkSbiS2hW9-EQS883o"
 }

```

- This token will be used to authenticate the user when they try to access protected routes.

- To get the token from the header in the users controller, so as to get access to the current user, add the following method to the application controller.To get the current user, we will need to create a helper method in the application controller. This method will decode the token and get the user id from it. It will then use the user id to find the user in the database and return the user.

```

def decoded_token
    if auth_header
        token = auth_header.split(' ')[1]
        begin
            JWT.decode(token, 'my_s3cr3t', true, algorithm: 'HS256')
        rescue
            nil
        end
    end

end

def current_user
    if decoded_token
        user_id = decoded_token[0]['user_id']
        @user = User.find_by(id: user_id)
    end
end

def logged_in?
    !!current_user
end

def authorized
    if logged_in?
        true
    else
        render json: { message: 'Please log in' }, status: :unauthorized
    end
end

```

- With this , create an action and a route that will provide details on the logged in /signed up current user . This is where the jwt token comes in to play . First , in our users controller create a profile method that will return a user.

```

def profile
    render json: { user: UserSerializer.new(current_user) }, status: :accepted
end

```

- Next , in our routes.rb file , add a route for the profile action

```

Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :users, only: [:create]
      get '/profile', to: 'users#profile'
    end
  end
end

```

- Also add this line to the top of the users controller:

```

 skip_before_action :authorized, only: [:create]

```

- The users controller should now look like this:

```

class Api::V1::UsersController < ApplicationController
  skip_before_action :authorized, only: [:create]
  def create
    user = User.new(user_params)
    if user.save
      token = encode_token({ user_id: user.id })
      render json: { user: UserSerializer.new(user), jwt: token }
    else
      render json: { error: 'Error creating user' }
    end
  end

  def profile
    render json: { user: UserSerializer.new(current_user) }, status: :accepted
  end

  private

  def user_params
    params.require(:user).permit(:username, :email, :password)
  end
end

```

- For the jwt token to get the current user . Lets get this token from the response header after signing up and use it in the header of the GET request to the /api/v1/profile endpoint to get the current user . First, let us sign up a new user , send a POST REQUEST to the endpoint below

```

http://localhost:3000/api/v1/users

```

- With the following data in the body

```

{
"user": {
"username": "test3",
"email": "test3@gmail.com",
"password": "password3"
}
}

```

- You should get a response with the user's email and username and a token

```

{
"user": {
"username": "test3",
"email": "test3@gmail.com"
},
"token" : ewiho2039103j3ijeohoh1oej019u0192jj0is1js01j
`}

```

- Copy the token and use it in the header of the GET request to the /api/v1/profile endpoint to get the current user . In postman , send a GET request to the endpoint below

```

http://localhost:3000/api/v1/profile

```

- On the headers tab , add a key of Authorization and a value of Bearer <token> . The token is the one you copied from the response of the POST request to the /api/v1/users endpoint.

- On the headers for the request add one for Authorization , and add the keyword Bearer , followed by the token you got upon signing up The Authorization Header should look like this
  Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo1fQ.Oo2CYiJr2SrNc8DVm_DNnMHzTm0j4ACjzgs5JqogTQ
- We will get this as the response body

```

{ "user":
 { "username": "test3",
  "email": "test3@gmail.com" }
  }

```

- Create the actions and routes for the login and logout endpoints. Lets create a auth controller and add the following actions to it

```

def create
    @user = User.find_by(username: login_params[:username])
    if @user && @user.authenticate(login_params[:password])
        token = encode_token(user_id: @user.id)
        render json: { user: UserSerializer.new(@user), jwt: token }, status: :accepted
    else
        render json: { error: 'Invalid username or password' }, status: :unauthorized
    end
end

private

def login_params
    params.require(:user).permit(:username, :password)
end

```

- Modify the routes.rb file to add the login route

```

Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :users, only: [:create]
      post '/login', to: 'auth#create'
      get '/profile', to: 'users#profile'
    end
  end
end

```

- Run the rails server and send a POST request to the endpoint below

```

http://localhost:3000/api/v1/login

```

- With the following data in the body

```

{
"user": {
"username": "test3",
"password": "password3"
}
}

```

- You should get a response with the user's email and username and a token

```

{

"user": {
"username": "test3",
"email": "test3@gmail.com"
},
"jwt" : ewiho2039103j3ijeohoh1oej019u0192jj0is1js01j
`}

```

- This profile can now be used to get the current user. To logout a user, we will need to delete the token. To do this, we will need to create a destroy action in the auth controller

```

def destroy
    session.delete(:user_id)
    render json: { message: 'Logged Out' }
end

```

- Modify the routes.rb file to add the logout route

```

Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :users, only: [:create]
      post '/login', to: 'auth#create'
      delete '/logout', to: 'auth#destroy'
      get '/profile', to: 'users#profile'
    end
  end
end

```

- Run the rails server and send a DELETE request to the endpoint below

```

http://localhost:3000/api/v1/logout

```

- You should get a response with the message Logged Out

```

{

"message": "Logged Out"

}

```

- Remember to add the seed data to the seeds.rb file and run the command `rails db:seed ` to seed the database with the data.

- This is the end of the backend part.
