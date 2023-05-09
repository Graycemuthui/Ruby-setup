## Ruby-setup

- Create a new application
  `rails new . --database postgresql`

- Update the postgres user and password in config/database.yml
  `  username: postgres
password: Gracewanjiru1573`

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

- Go to the routes and create the namespaced routes
  `namespace :api do
  namespace :v1 do
  resources :table_name
  end
  end

- Create the controllers for the namespaced routes

- Add corss rb gem to the gemfile
  `gem 'rack-cors'`

- Run bundle install

- Create a file in the config/initializers/cors.rb

  `Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
  origins "\*"

      resource "*",
        headers: :any,
        methods: [:get, :post, :put, :patch, :delete, :options, :head]

end
end`

- Add the following gems
  `gem "bcrypt"
gem "rack-cors"
gem "active_model_serializers"
gem "jwt"`

- Install the gems
  `bundle install`

- In the config/initializers/cors.rb file add the following:

  ````Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
  origins '\*',
  resource '\*',
  headers: :any,
  methods: %i[get post put patch delete options head]

  end
  end```
  ````
